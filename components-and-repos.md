# Components and Repositories

The CTSI PoS stack spans five main repositories. Each owns a distinct layer: contracts, pool logic, node software, indexing, and the user interface.

## Stack diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     explorer (Next.js UI)                     │
│              https://explorer.cartesi.io                    │
└──────────────────────────┬──────────────────────────────────┘
                           │ GraphQL + wallet (MetaMask)
┌──────────────────────────▼──────────────────────────────────┐
│                    subgraph (The Graph)                     │
│         Indexes on-chain events → GraphQL API               │
└──────────────────────────┬──────────────────────────────────┘
                           │ reads events
┌──────────────────────────▼──────────────────────────────────┐
│              Ethereum mainnet (on-chain layer)                │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│  │  pos-dlib   │  │ staking-pool │  │   CTSI ERC-20 token │  │
│  │  (PoS core) │  │  (pools)     │  │                     │  │
│  └──────┬──────┘  └──────┬───────┘  └─────────────────────┘  │
└─────────┼────────────────┼──────────────────────────────────┘
          │                │
          │  produceBlock  │  rebalance / hire
┌─────────▼────────────────▼──────────────────────────────────┐
│                    noether (node software)                    │
│              Docker image: cartesi/noether                    │
└─────────────────────────────────────────────────────────────┘
```

---

## pos-dlib — Proof of Stake core contracts

**Repo:** [cartesi/pos-dlib](https://github.com/cartesi/pos-dlib) · **Local:** `../pos-dlib`

The foundational smart contract library for Cartesi PoS. Published on npm as `@cartesi/pos`.

### Responsibilities

- **Staking** (`StakingImpl`) — deposit, stake, unstake, withdraw CTSI with maturation periods
- **Selection** (`Eligibility`, `EligibilityCalImpl`) — weighted random block producer selection
- **Difficulty** (`Difficulty`, `DifficultyManagerImpl`) — adaptive difficulty to regulate block interval
- **Rewards** (`RewardManagerV2`) — fixed CTSI payout per block
- **PoS orchestration** (`PoSV2`, `PoSV2FactoryImpl`) — ties staking, selection, and rewards together; exposes `produceBlock()` and eligibility queries
- **Worker auth** (`WorkerAuthManager`) — links staker addresses to authorized worker (node) addresses

### Key detail

PoS v2 (current) is backward-compatible with v1. The factory deploys contracts in either V1-compatible or V2 mode. The selection algorithm uses a future Ethereum block hash as a randomness source and an exponential distribution weighted by stake.

### Tech stack

Solidity, Hardhat, Foundry, TypeScript deployment scripts. Deployments tracked for mainnet, Sepolia, and localhost.

---

## staking-pool — Staking pool contracts

**Repo:** [cartesi/staking-pool](https://github.com/cartesi/staking-pool) · **Local:** `../staking-pool`

Smart contracts that let operators create **public staking pools** — aggregating many delegators behind a single PoS stake and Noether node.

### Responsibilities

The pool contract is split into five facets:

| Facet | Role |
| --- | --- |
| **User** | `deposit`, `stake`, `unstake`, `withdraw` — delegator lifecycle |
| **Staking** | `rebalance()` — syncs pool liquidity with PoS staking contract |
| **Worker** | `hire`, `cancelHire`, `retire` — node hiring and lifecycle |
| **Management** | `pause`/`unpause`, ENS name, commission configuration |
| **Block Producer** | `produceBlock` — claims rewards, takes commission, compounds rest |

### Commission models

- **FlatRateCommission** — fixed percentage of each block reward
- **GasTaxCommission** — reimbursement based on gas cost, converted via Chainlink oracles (gas → ETH → CTSI)

### Key detail

`rebalance()` is permissionless — any user can call it if the pool node is down, ensuring delegators can exit even if the operator abandons the pool.

### Tech stack

Solidity, Hardhat, TypeScript. Depends on `@cartesi/pos`. Also provides a local dev script that deploys the full contract set for integration testing.

---

## noether — Node software

**Repo:** [cartesi/noether](https://github.com/cartesi/noether) · **Local:** `../noether`

The reference implementation of a Cartesi PoS **worker node**. Runs as a Docker container (`cartesi/noether`) and implements the off-chain half of block production.

### Responsibilities

- Connect to an Ethereum JSON-RPC provider
- Manage an encrypted node wallet (created on first run)
- Poll PoS contracts for eligibility (`whenCanProduceBlock`, `canProduceBlock`)
- Submit `produceBlock()` transactions when eligible
- For pool nodes: call `rebalance()` on the pool contract (~every 30 seconds)
- Optional Prometheus metrics (`noether_balance_eth`, `noether_block_total`, etc.)

### Operating modes

| Mode | Description |
| --- | --- |
| Private node | Worker for a single staker's direct stake |
| Pool node | Worker for a staking pool; also handles rebalance |
| Local dev | Connects to Hardhat with contracts from pos-dlib |

### Tech stack

TypeScript, Node.js, ethers.js, Docker. CLI commands: `start`, `import`, `export`.

---

## explorer — Staking portal UI

**Repo:** [cartesi/explorer](https://github.com/cartesi/explorer) · **Local:** `../explorer`

The public-facing web application at [explorer.cartesi.io](https://explorer.cartesi.io). A Next.js app for all staking interactions.

### Responsibilities

- **Stake** — browse pools, deposit, stake, unstake, withdraw
- **Node Runners** — create private nodes, create public pools, hire nodes
- **Blocks** — view produced blocks and chain history
- **Wallet integration** — MetaMask, WalletConnect, Coinbase Wallet, Ledger
- **Networks** — mainnet, Sepolia, local devnet (31337)

### Data sources

- **The Graph subgraph** — pool stats, user histories, block data, commissions
- **Direct RPC** — wallet transactions, live contract reads

### Tech stack

Next.js, TypeScript, Storybook for component development. Deployed via Vercel with semver release tags.

---

## subgraph — Blockchain indexer

**Repo:** [cartesi/subgraph](https://github.com/cartesi/subgraph) · **Local:** `../subgraph`

A monorepo of [The Graph](https://thegraph.com) subgraph definitions that index PoS on-chain events into a GraphQL API consumed by the explorer.

### Packages

| Package | Indexes |
| --- | --- |
| `pos-subgraph` | Staking events, pools, nodes, blocks, fees, user histories |
| `block-subgraph` | Block production data |
| `toolbox` | Shared utilities |

### Responsibilities

- Map contract events to GraphQL entities (pools, users, blocks, commissions)
- Support multiple networks via template YAML files (mainnet, Sepolia, localhost)
- Unit tests with Matchstick framework

### Tech stack

AssemblyScript mappings, Graph CLI, Hardhat for local testing.

---

## Dependency relationships

```
pos-dlib  ←── staking-pool (depends on @cartesi/pos)
    ↑              ↑
    │              │
 noether      subgraph (indexes both)
    ↑              ↑
    └──── explorer (reads subgraph + RPC)
```

## Related but outside this stack

| Repo | Relationship |
| --- | --- |
| [cartesi/docs](https://github.com/cartesi/docs) | User-facing staking guides (`earn-ctsi/`) |
| CTSI ERC-20 token | Separate token contract; staked via `StakingImpl` |
| Cartesi Rollups repos | Different stack — application execution, not token staking |
