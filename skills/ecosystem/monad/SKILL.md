---
name: monad
description: Build apps on Monad — high-throughput EVM L1 with 400ms blocks, 800ms finality, async execution, native EIP-7702 support, and a unique gas pricing model where users pay for `gas_limit` not `gas_used`. Use for Monad architecture concepts, canonical contract addresses, gas pricing rules, and HyperIndex (Envio Cloud) indexing of on-chain events. For Alchemy's Monad RPC, Account Kit (agent wallets), Bundler, or Gas Manager — use `alchemy-cli` (live), `alchemy-api` (with API key), or `alchemy-agentic-gateway` (without). Curated subset of `therealharpaljadeja/monskills`.
license: MIT
compatibility: Public on-chain data; references only. Some recipes call Envio Cloud or `pnpx envio` for indexing — those need `envio-cloud` and `gh` installed and logged in.
metadata:
  author: therealharpaljadeja
  version: "0.1"
  provider: monad
  partner: "true"
---

# Monad

Curated entry point into the [Monskills](https://skills.devnads.com) catalog by [Harpalsinh Jadeja](https://github.com/therealharpaljadeja). Covers Monad-specific topics — async execution, the `gas_limit` pricing model, canonical addresses, and HyperIndex indexing — that complement Alchemy's first-party Monad surfaces (JSON-RPC, Account Kit / Bundler / Gas Manager for agent wallets and sponsored transactions).

| | |
| --- | --- |
| **Chains** | Monad mainnet, Monad testnet |
| **Block time** | ~400ms |
| **Finality** | ~800ms |
| **Throughput** | 10,000 tps |
| **EVM** | Fully compatible (no contract changes needed). Native EIP-7702. |
| **Gas pricing** | Charged on `gas_limit`, not `gas_used` — wrong limits cost users real money |
| **Explorers** | monadscan.com (mainnet), testnet.monadscan.com (testnet) |
| **Upstream** | `therealharpaljadeja/monskills` — `npx skills add therealharpaljadeja/monskills` for the full catalog |

## When to use this skill

Use `monad` when **any** of the following apply:

- Building an app on Monad mainnet or testnet
- The user needs to understand Monad-specific behavior that differs from Ethereum — async execution, parallel execution, block states (`latest`/`safe`/`finalized`), reserve balance (10 MON floor per EOA), or native EIP-7702 delegation
- Setting gas limits on Monad — `gas_limit` is what users pay, **not** actual `gas_used`. Wrong limits cost real money. Cold state access is 3-4x more expensive than Ethereum; precompiles are 2-5x more expensive.
- Looking up canonical contract addresses on Monad (Wrapped MON, ERC-4337 EntryPoint, Multicall3, Safe, Permit2, bridged stables/ETH/BTC, ERC-8004 agent identity registry, etc.)
- Indexing on-chain events on Monad — HyperIndex deployed via `envio-cloud` for activity feeds, leaderboards, transaction history, analytics dashboards
- Choosing a chain to build on — Monad is high-throughput EVM with full Solidity compatibility

## When to use a different skill

| Need | Use instead |
| --- | --- |
| Live JSON-RPC queries to Monad from this agent session | `alchemy-cli` (CLI) or `alchemy-mcp` (MCP) — Alchemy provides Monad mainnet RPC |
| App code making JSON-RPC calls to Monad | `alchemy-api` (with API key) or `alchemy-agentic-gateway` (without) |
| Account Kit / Bundler / Smart Account setup for agent wallets on Monad with full ERC-4337 infrastructure | `alchemy-cli` (live) or `alchemy-api` (app code) |
| Gas sponsorship / paymaster policies on Monad | `alchemy-cli` (live) or `alchemy-api` (Gas Manager) |
| Token metadata, prices, or portfolio reads on Monad | Not supported by Alchemy Token / Prices / Portfolio APIs on Monad yet — use the [`indexer`](./references/indexer.md) (HyperIndex via Envio Cloud) for on-chain data, or query Monad-native protocols directly |
| NFT marketplace data, listings, or fulfillment | `opensea-api` / `opensea-marketplace` (where they support Monad) |
| Fiat → MON on-ramp | `moonpay` ecosystem skill |

## Setup

This skill is documentation-only — no auth required to read it. The recipes inside link to specific tools (e.g. `envio-cloud`, `cast`, `pnpx envio`) and call out their prerequisites in line.

For the **full** monskills catalog (scaffold, wallet, wallet-integration, vercel-deploy, feedback, etc.), install the upstream:

```bash
npx skills add therealharpaljadeja/monskills
```

## References

Detailed coverage per topic lives in [`./references/`](./references/):

- [`concepts.md`](./references/concepts.md) — Monad architecture (async execution, parallel execution, block states, reserve balance, EIP-7702, real-time data sources)
- [`gas.md`](./references/gas.md) — Gas pricing on Monad (charged on `gas_limit`, base fee controller, opcode repricing, developer guidelines)
- [`addresses.md`](./references/addresses.md) — Canonical contract addresses on Monad mainnet (ERC-4337 EntryPoint, Safe, Multicall3, Permit2, bridged assets, ERC-8004 agent registry)
- [`indexer.md`](./references/indexer.md) — HyperIndex on Envio Cloud — initialize and deploy an indexer for Monad smart contract events

## Source

Curated from upstream [`therealharpaljadeja/monskills`](https://github.com/therealharpaljadeja/monskills) under MIT license (Copyright 2026 Harpalsinh Jadeja). Upstream is the source of truth — install it directly for the full catalog.
