# Noether / CTSI PoS Documentation

High-level knowledge base for Cartesi's Proof of Stake (PoS) system and the Noether node software. This folder covers the *what* and *why* at the ecosystem level. Per-repo deep dives will be added later.

**Slides:** [riseandshaheen.github.io/noether-docs](https://riseandshaheen.github.io/noether-docs/) (also [slides.html](./slides.html) in-repo)

## What this is

Cartesi's PoS stack is separate from the Rollups application stack. It governs **CTSI token staking**, **block production on the Noether sidechain**, and **reward distribution** from Cartesi's Mine Reserve. Users earn CTSI by staking and participating in block production — either directly (private node) or through staking pools (delegation).

## Documents

| Document | Description |
| --- | --- |
| [HTML slides](./slides.html) | Presentation deck covering all topics below (open in browser) |
| [Introduction to CTSI Staking](./intro-ctsi-staking.md) | What staking is, how rewards work, and the three ways to participate |
| [Components and Repositories](./components-and-repos.md) | Each piece of the stack and its GitHub repo |
| [Architecture Overview](./architecture-overview.md) | How on-chain contracts, the node, indexer, and UI fit together |
| [Key Concepts](./key-concepts.md) | Sidechain, workers, maturation periods, pools, commissions, and more |
| [History](./history.md) | Timeline of PoS / Noether development |
| [Relationship to Rollups](./relationship-to-rollups.md) | How PoS differs from the Rollups engineering stack |
| [pos-dlib (core contracts)](./pos-dlib.md) | High-level map of PoS contracts, how they interact, and audits |
| [staking-pool](./staking-pool.md) | Public pools: facets, rebalance, commissions, and audits |

## External resources

- [Cartesi Staking Portal](https://explorer.cartesi.io) — live staking UI
- [Official docs: Earn CTSI](https://docs.cartesi.io/earn-ctsi/staking) — user-facing guides
- [Staking FAQ](https://docs.cartesi.io/earn-ctsi/staking-faq) — costs, security, operations

## Planned deep dives (not yet written)

- `noether/` — node software, Docker deployment, monitoring
- `pos-dlib/` — selection math, v2 historical tree, deployment/scripts (high-level: [pos-dlib.md](./pos-dlib.md))
- `staking-pool/` — share math, rebalance edge cases, factory/oracle ops (high-level: [staking-pool.md](./staking-pool.md))
- `explorer/` — Next.js app, wallet integration, local dev setup
- `subgraph/` — GraphQL schema, indexing, deployment
