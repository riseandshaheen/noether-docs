# Relationship to Rollups

Cartesi's engineering organization maintains two distinct stacks. Understanding the boundary between them prevents confusion when navigating the codebase.

## Two stacks, one token

```
┌─────────────────────────────────────────────────────────────┐
│                    Cartesi ecosystem                         │
│                                                             │
│  ┌─────────────────────┐    ┌─────────────────────────┐  │
│  │   Rollups stack      │    │   PoS / Noether stack   │  │
│  │   (applications)     │    │   (token economics)     │  │
│  │                      │    │                         │  │
│  │  machine-emulator    │    │  pos-dlib               │  │
│  │  rollups-contracts   │    │  staking-pool           │  │
│  │  rollups-node        │    │  noether                │  │
│  │  sequencer           │    │  explorer               │  │
│  │  cli / rollups-ts    │    │  subgraph               │  │
│  │  rollups-explorer    │    │                         │  │
│  └─────────────────────┘    └─────────────────────────┘  │
│           │                          │                     │
│           └──────── CTSI token ──────┘                     │
└─────────────────────────────────────────────────────────────┘
```

Both stacks involve CTSI, but in different ways:

| | Rollups stack | PoS / Noether stack |
| --- | --- | --- |
| **Purpose** | Run blockchain applications off-chain | Distribute CTSI rewards to stakers |
| **What runs** | Application logic in Cartesi Machine | Empty block production |
| **User action** | Deploy / interact with Rollups apps | Stake CTSI, run or delegate to a node |
| **On-chain anchor** | L1 Rollups contracts (InputBox, Application) | PoS contracts (StakingImpl, PoSV2) |
| **Node software** | rollups-node | noether |
| **Engineering focus (2026)** | Active development (v2 alphas) | Maintenance |

## What "Noether" means in each context

The word "Noether" appears in both stacks but refers to different things:

- **PoS / Noether** — the sidechain where PoS stakers produce blocks and earn CTSI. The `noether` repo is the node software for this.
- **Rollups docs** — occasionally reference Noether in the context of Cartesi's broader L2 narrative, but Rollups applications do not run on the Noether sidechain.

When reading Cartesi documentation, check whether "Noether" refers to the staking sidechain or is used as a general term for Cartesi's L2 positioning.

## Explorer vs Rollups Explorer

Two different web applications:

| App | Repo | URL | Purpose |
| --- | --- | --- | --- |
| **Staking Explorer** | `explorer` | explorer.cartesi.io | CTSI staking, pools, node management |
| **Rollups Explorer** | `rollups-explorer` | — | Interact with Rollups applications (inputs, deposits, outputs) |

They share no code and serve different users.

## Subgraph vs Rollups Explorer API

Similarly, indexing is separate:

| Indexer | Repo | Indexes |
| --- | --- | --- |
| PoS subgraph | `subgraph` | Staking, pools, blocks, nodes |
| Rollups Explorer API | `rollups-explorer-api` | Rollups application events, inputs, outputs |

## Shared infrastructure

A few things cross the boundary:

| Shared item | Used by |
| --- | --- |
| **CTSI ERC-20 token** | Staked in PoS; may be used in Rollups apps via portals |
| **Ethereum mainnet** | Both stacks anchor to Ethereum L1 |
| **Cartesi org on GitHub** | All repos under `github.com/cartesi` |
| **Official docs** | `docs.cartesi.io` covers both Rollups and Earn CTSI sections |

## Engineering dependency

There is **no code dependency** between the stacks. The Rollups node does not import noether. PoS contracts do not reference Rollups contracts. They are independently deployable and versioned.

The parent workspace README reflects this: PoS repos are listed under "CTSI staking and chain explorer" as distinct from the Rollups engineering stack diagram.

## Why both exist

Cartesi's token economics (PoS) predate and operate independently from the application platform (Rollups):

- **PoS** incentivizes CTSI holders to participate in the network by staking and producing blocks, distributing Mine Reserve rewards.
- **Rollups** provides the execution environment where developers build and deploy decentralized applications.

A CTSI holder can stake without ever touching Rollups. A Rollups developer can build applications without running a PoS node. The stacks converge only at the token level.

## For documentation readers

When this `noether-docs` folder refers to "the stack," it means the PoS / Noether stack only. For Rollups architecture, see the Rollups documentation in `../docs/` and the engineering stack diagram in the parent workspace README.
