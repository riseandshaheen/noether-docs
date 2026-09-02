# History

A timeline of Cartesi Proof of Stake and Noether development. This covers major releases and architectural shifts across all five repos.

## 2020 — Genesis

| Date | Event |
| --- | --- |
| Sep 2020 | **pos-dlib** created. Early beta releases (`v0.1.0-beta`) establish the core staking and selection contracts. |
| Dec 2020 | **pos-dlib v1.0.0** — first production release of PoS contracts. |
| Dec 2020 | **noether** repo created. Implements the off-chain worker that polls PoS contracts and produces blocks. |
| Dec 2020 | **noether v1.0.0** — first node release. Docker image `cartesi/noether` published. |

The initial system launched with PoS v1 contracts (`PoS`, `BlockSelector`, `RewardManager`, `StakingImpl`) and a simple node that produced empty blocks paying CTSI rewards.

## 2021 — Production hardening and pools

| Date | Event |
| --- | --- |
| Jan 2021 | **noether v1.0.x** — ARM Docker images (Raspberry Pi support), gas price spike handling, reduced RPC usage. |
| Feb 2021 | **pos-dlib v1.1.0** — target interval changed from elapsed time to block count; 23% gas reduction on block production. |
| Mar 2021 | **noether v1.1.0** — ETH Gas Station API integration, ARM v7 support, graceful shutdown. |
| May 2021 | **noether v1.2.0** — stops block production on deprecated protocol versions. |
| Jun 2021 | **pos-dlib v1.1.x** — Solidity 0.8 interfaces, Ropsten deployments. |
| Sep 2021 | **noether v2.0.0** — **staking pool support**. Prometheus monitoring. Major node rewrite for pool integration. |
| Sep–Nov 2021 | **noether v2.0.x** — pool retire fixes, monitoring metric fixes. |
| 2021 | **staking-pool v1.0.0** — staking pool contracts released. Five-facet design (User, Staking, Worker, Management, Producer). FlatRate and GasTax commission models. |
| 2021 | **explorer** — staking portal launched at explorer.cartesi.io. |
| 2021 | **subgraph** — initial indexing for pools, nodes, blocks, and user histories. |

This period transformed PoS from a solo-staker system into a delegation platform with public pools.

## 2022 — PoS v2 contracts

| Date | Event |
| --- | --- |
| Nov 2022 | **pos-dlib v2.0.0** — **PoSV2 smart contracts**. Modular redesign: separate Eligibility, Difficulty, and RewardManagerV2 libraries. PoSV2Factory for deployment. 100% backward compatibility with v1. |
| 2022 | **staking-pool v2.0.0** — updated for PoS v2 contracts. |
| 2022 | **explorer v3.x** — ongoing UI improvements, Storybook component library, multi-network support. |
| 2022 | **subgraph v3.x** — indexing updates for PoS v2 events and block selector context versions (1.0, 1.1, 2.0). |

## 2023–2024 — Network evolution and maintenance

| Date | Event |
| --- | --- |
| 2023 | Legacy testnet cleanup begins (Ropsten, Goerli deployments deprecated). |
| Feb 2024 | **pos-dlib v2.1.0** — Sepolia network support added. Legacy testnet deployments removed (Avax, BSC, Optimism Goerli, Polygon Mumbai, Rinkeby, Ropsten). |
| Apr 2024 | **noether** — Sepolia support merged. Last significant public commit. |
| 2024 | Explorer continues active development (v3.12.x releases through 2026). |

## 2025–2026 — Ongoing maintenance

| Date | Event |
| --- | --- |
| Aug 2025 | **staking-pool** — ethers package upgrade. |
| Sep 2025 | **subgraph** — Chainstack mainnet preview deployment script removed. |
| Jun 2026 | **explorer v3.12.x** — latest release with changelog updates. |

The PoS stack is in **production / maintenance** mode. The Cartesi team's primary engineering focus shifted to Rollups v2 (machine, contracts, node, sequencer, CLI). PoS repos receive dependency updates, network migration support, and explorer UI improvements, but no major architectural changes.

## Architectural evolution summary

```
2020          2021              2022              2024+
│             │                 │                 │
PoS v1        Pools +           PoS v2            Maintenance
+ Noether     Noether v2        contracts         + Sepolia
              + Explorer        + modular         + explorer UI
              + Subgraph        libraries         updates
```

## Key design decisions (chronological)

1. **Ethereum-native PoS** — staking and block claims on Ethereum mainnet, not a separate chain with its own consensus.
2. **Weighted random selection** — exponential distribution based on stake, not round-robin or fixed slots.
3. **Worker separation** — staker and block producer (worker) are distinct roles, enabling pool delegation.
4. **Maturation delays** — 6h stake / 48h unstake maturation to prevent manipulation around selection.
5. **Facet-based pools** — staking pool split into five facets for auditability and upgradeability.
6. **Permissionless rebalance** — anyone can trigger pool maintenance, protecting delegators from abandoned operators.
7. **PoS v2 modularization** — eligibility, difficulty, and rewards as separate libraries behind a factory.
8. **No slashing** — node failure costs missed rewards, not principal.
9. **Fixed block reward** — 2,900 CTSI per block from Mine Reserve, regardless of stake size.

## Authors across repos

| Person | Repos |
| --- | --- |
| Danilo Tuler | pos-dlib, noether, staking-pool, subgraph |
| Felipe Argento | pos-dlib |
| Gabriel Barros | staking-pool |
| Gabriel Coutinho | noether |
| Stephen Chen | pos-dlib |
| Alexander Bai | subgraph |
