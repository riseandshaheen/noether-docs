# Key Concepts

Terms and ideas you need to understand the PoS / Noether stack before diving into individual repos.

## Noether sidechain

Noether is Cartesi's **sidechain** — a separate chain where blocks are produced by PoS-selected stakers. Today, Noether blocks are **empty** (no application state or transactions). The sidechain exists primarily as a **reward distribution mechanism**: stakers earn CTSI from the Mine Reserve for each block they produce.

Noether is **not** Cartesi Rollups. Rollups run application logic inside the Cartesi Machine. Noether handles token staking economics only.

Block claims happen on Ethereum mainnet — the "sidechain" is tracked on-chain via the PoS contract's block counter and producer history, not as a separate network you connect to directly.

## Staker vs worker

The PoS system separates two roles:

| Role | What it is | Example |
| --- | --- | --- |
| **Staker (owner)** | The Ethereum address that holds staked CTSI | Your MetaMask wallet |
| **Worker (node)** | The authorized agent that submits `produceBlock()` transactions | Noether node's wallet |

A staker **hires** a worker by linking the node's address to their stake via `WorkerAuthManager`. The worker spends ETH gas; the staker receives CTSI rewards. One worker per staker (or per pool).

For private nodes, you are both staker and worker operator. For pools, the pool contract is the staker and the operator's node is the worker.

## Hiring

"Hiring" a node means sending ETH to the node's wallet and registering it as your worker on-chain. This:

1. Funds the node wallet for gas
2. Authorizes the node to call `produceBlock()` on your behalf
3. "Wakes up" the node to start polling for eligibility

Without hiring, a node wallet exists but cannot produce blocks.

## Maturation periods

Staking and unstaking are not instant. Security delays prevent last-minute stake manipulation around block selection.

| Action | Maturation | Effect |
| --- | --- | --- |
| **Stake** | ~6 hours | New CTSI does not count toward selection until matured |
| **Unstake** | Immediate removal from selection | Tokens enter a releasing queue |
| **Withdraw** | ~48 hours after unstake | Tokens transfer back to wallet |

**Important:** Adding new stake while existing stake is still maturing **resets the maturation timer** for all maturing tokens. Same for unstaking while tokens are in the releasing queue.

## Staking pool shares

Pools use an ERC-20-style share system:

- Initial ratio: 1 CTSI = 1 share (at different decimal scales)
- When the pool earns block rewards, total CTSI increases but share count stays fixed → each share becomes worth more
- New depositors receive fewer shares per CTSI than early depositors
- Rewards auto-compound — no separate claim step for delegators

## Rebalance

Pools cannot stake/unstake on the PoS contract immediately when a user deposits or unstakes. The `rebalance()` function is the bridge:

- Checks pool's free CTSI balance vs. liquidity needed for pending withdrawals
- Stakes excess balance on PoS, or unstakes/withdraws to cover shortfalls
- Called automatically by the Noether node every ~30 seconds
- **Permissionless** — anyone can call it if the node is down

## Commission

Pool operators earn commission on block rewards. Two models:

| Model | How it works |
| --- | --- |
| **Flat rate** | Fixed % of each block reward (e.g. 5%) |
| **Gas tax** | Reimbursement for gas spent, converted via Chainlink oracles |

Commission can be **decreased** at any time. Increases are rate-limited (max step size + cooldown period) to protect delegators.

## Difficulty and target interval

The PoS system self-regulates block production speed:

- **Target interval** — desired time between blocks (configured at deployment)
- **Difficulty** — scalar that scales with total staked balance
- After each block, difficulty adjusts based on how long the last interval actually took

If total stake increases 2×, difficulty increases 2× to keep the same block rate. This is the inverse of Bitcoin's difficulty model (which adjusts for hashrate, not stake).

## Block reward

Each successfully produced block pays a **fixed reward** of approximately **2,900 CTSI** from the Mine Reserve. The reward does not depend on stake size — only on winning the selection race.

## Gas price strategy

Noether nodes must predict gas prices for their transactions. Two strategies:

- **eth-provider** (default) — uses `eth_gasPrice` from the RPC provider
- **gas-station** — uses ETH Gas Station API with configurable speed tier

Poor gas price prediction means slower block claims or wasted ETH.

## Prometheus monitoring

Noether exposes optional metrics for operators:

| Metric | Meaning |
| --- | --- |
| `noether_balance_eth` | Node wallet ETH balance |
| `noether_stake_ctsi` | Owner's staked CTSI |
| `noether_block_total` | Blocks produced |
| `noether_eligibility_total` | Times the node was eligible |
| `noether_rebalance_total` | Pool rebalance calls |
| `noether_errors_total` | Error counter |

## PoS v1 vs v2

| | PoS v1 | PoS v2 (current) |
| --- | --- | --- |
| Contracts | `PoS`, `BlockSelector` | `PoSV2`, modular libraries |
| Selection | BlockSelector v1/v2 | Eligibility + Difficulty libraries |
| Rewards | `RewardManager` | `RewardManagerV2` |
| Compatibility | — | 100% backward compatible with v1 |
| Factory | — | `PoSV2FactoryImpl` deploys either mode |

Production mainnet runs PoS v2 in V1-compatible mode. The v2 mode introduces additional features (historical data, modular eligibility/difficulty).

## No slashing

Unlike many PoS systems, Cartesi PoS currently has **no slashing**. If your node goes offline or runs out of ETH:

- You lose potential rewards (missed blocks)
- Your staked CTSI remains safe
- You can unstake and withdraw at any time via the explorer

## ENS pool names

Pool operators can register an ENS name for their pool via `setName()` in the management facet. This gives pools a human-readable identity on the explorer.
