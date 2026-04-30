---
name: lifi
description: Query LI.FI's API for cross-chain and same-chain token swap + bridge routing across 60+ blockchains. Aggregates 27 bridges and 31 DEXs (1inch, 0x, Paraswap, DODO, OpenOcean, etc.) under one interface — get quotes with ready-to-execute transaction data, compare multi-step routes, and track cross-chain transfer status. NOT for token prices, token metadata, current wallet balance snapshots, transaction history, or NFT data — for those use `alchemy-cli` (live), `alchemy-mcp`, `alchemy-api` (app code), or `agentic-gateway` (no API key). API key optional but recommended for production rate limits.
license: MIT
compatibility: API key optional via `$LIFI_API_KEY` (`x-lifi-api-key` header) for higher rate limits — register at https://li.fi. Network access required. Endpoints return ready-to-execute `transactionRequest` objects; the user / app signs and submits.
metadata:
  author: lifi
  version: "0.1"
  provider: lifi
  partner: "true"
---

# LI.FI (Cross-Chain Swap + Bridge Routing)

LI.FI aggregates 27 bridges and 31 DEXs across 60+ chains into a single quote/route API. This skill covers cross-chain and same-chain swap routing, bridge transfers, and transaction status tracking. For token prices, token metadata, current wallet balances, transaction history, or live RPC reads, use the corresponding Alchemy skill instead.

| | |
| --- | --- |
| **Base URL** | `https://li.quest/v1` |
| **Auth** | `x-lifi-api-key: $LIFI_API_KEY` header (optional but recommended for production rate limits) |
| **Aggregates** | 27 bridges + 31 DEXs across 60+ chains (EVM + SVM) |
| **Free tier** | Public access without a key, lower rate limits |

## When to use this skill

Use `lifi` when **any** of the following are true:

- The user wants to **swap tokens between two chains** (e.g., USDC on Ethereum → ETH on Arbitrum)
- The user wants to **bridge tokens** to another chain (e.g., move ETH from mainnet to Optimism)
- The user wants the **best swap rate on a single chain** (LI.FI aggregates DEXs)
- The user wants to **track status** of an in-flight cross-chain transfer
- The user wants to **discover** supported chains / tokens / bridges programmatically

## When NOT to use this skill (handoff)

| Need | Use instead |
| --- | --- |
| Token prices (current, historical, by-timestamp) | `alchemy-api` (Prices API) |
| Token metadata, search, list by chain | `alchemy-api` (Token API) |
| Current wallet balances (point-in-time) | `alchemy-api` (Portfolio / Token API) |
| Transaction history (transfers, asset movements) | `alchemy-api` (Transfers API) |
| NFT metadata / floor / ownership | `alchemy-api` (NFT API) |
| Live blockchain reads (block #, gas, `eth_call`) | `alchemy-cli` (live work) or `alchemy-api` (JSON-RPC) |
| Signed transaction submission | the app's wallet / `alchemy-api` |
| Transaction simulation pre-execution | `alchemy-api` (Simulation API) |
| Account abstraction (bundlers, gas managers) | `alchemy-api` |

## Scope contract

**This skill covers (`scope_in`):**

- Single-step quotes with ready-to-execute transaction data (`GET /quote`)
- Multi-route comparison (`POST /advanced/routes`) and step-level transaction population (`POST /advanced/stepTransaction`)
- Cross-chain transfer status tracking (`GET /status`)
- Discovery — supported chains (`GET /chains`), per-chain token lists (`GET /tokens`, `GET /token`), bridges + DEXs (`GET /tools`), connectivity (`GET /connections`)

**This skill does NOT cover (`scope_out`):**

- Token prices → handoff: `alchemy-api` (Prices API)
- Token metadata, search, list (LI.FI returns tokens for routing, not general lookup) → handoff: `alchemy-api` (Token API)
- Current wallet balances (point-in-time) → handoff: `alchemy-api` (Portfolio / Token API)
- Transaction transfer history → handoff: `alchemy-api` (Transfers API)
- NFT metadata / floor prices → handoff: `alchemy-api` (NFT API)
- Live RPC reads, gas estimation, block details → handoff: `alchemy-cli` or `alchemy-api` (JSON-RPC)
- Signed tx submission → user's wallet (LI.FI returns `transactionRequest`; app signs and submits)
- Pre-execution simulation beyond what LI.FI runs internally → handoff: `alchemy-api` (Simulation API)
- Account abstraction → handoff: `alchemy-api` (Wallets / Bundler / Gas Manager)

## Setup

API key is optional. Without one, requests are rate-limited per IP. With one, rate-limited per key (higher tier).

Register for an API key at [li.fi](https://li.fi) and store it as `LIFI_API_KEY`:

```bash
export LIFI_API_KEY="..."
```

Public discovery works without auth:

```bash
curl "https://li.quest/v1/chains"
```

For production rate limits, include the header on every call:

```bash
curl "https://li.quest/v1/chains" \
  -H "x-lifi-api-key: $LIFI_API_KEY"
```

> **Security:** never expose `LIFI_API_KEY` in client-side code. Use it server-side only.

## Endpoint reference → [references/routing.md](./references/routing.md)

### Routing

| Endpoint | Use for |
| --- | --- |
| `GET /quote` | Single-step quote + ready-to-execute `transactionRequest`. Use for "I want X on chain B from Y on chain A". |
| `POST /advanced/routes` | Compare multiple routes; returns steps without TX data. Use to inspect route options, fees, gas estimates. |
| `POST /advanced/stepTransaction` | Populate a chosen step with `transactionRequest`. Use after picking a route from `/advanced/routes`. |
| `GET /status` | Poll cross-chain transfer status. Returns `PENDING` → `DONE` / `FAILED`. |

### Discovery

| Endpoint | Use for |
| --- | --- |
| `GET /chains` | List supported chains (EVM + SVM). |
| `GET /tokens` | Token list per chain (filter by `minPriceUSD`). |
| `GET /token` | Resolve a single token by address or symbol on a specific chain. |
| `GET /tools` | Available bridges + DEXs that LI.FI aggregates. |
| `GET /connections` | Which chain pairs / token pairs are routable. |

## Quick examples

### Cross-chain swap quote (USDC on Arbitrum → DAI on Optimism)

```bash
curl "https://li.quest/v1/quote?\
fromChain=42161&\
toChain=10&\
fromToken=0xaf88d065e77c8cC2239327C5EDb3A432268e5831&\
toToken=0xDA10009cBd5D07dd0CeCc66161FC93D7c9000da1&\
fromAmount=10000000&\
fromAddress=0xYourAddress&\
slippage=0.005" \
  -H "x-lifi-api-key: $LIFI_API_KEY"
```

Response includes `transactionRequest.{to, data, value, gasLimit}` ready for the user's wallet to sign + submit. Pass the resulting tx hash to `/status` to track.

### Same-chain swap (native ETH → USDC on Polygon)

```bash
curl "https://li.quest/v1/quote?\
fromChain=137&\
toChain=137&\
fromToken=0x0000000000000000000000000000000000000000&\
toToken=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174&\
fromAmount=1000000000000000000&\
fromAddress=0xYourAddress&\
slippage=0.005"
```

### Track cross-chain status

```bash
curl "https://li.quest/v1/status?txHash=0xSendingTxHash&fromChain=42161&toChain=10"
```

Status values: `NOT_FOUND`, `INVALID`, `PENDING`, `DONE`, `FAILED`. Poll until `DONE` or `FAILED`.

### List supported chains

```bash
curl "https://li.quest/v1/chains"
curl "https://li.quest/v1/chains?chainTypes=EVM,SVM"
```

## Common gotchas

- **`fromAmount` is in smallest unit** (no decimals applied). 10 USDC = `10000000` (USDC has 6 decimals), not `10`.
- **`fromAddress` is required** even though no signing happens server-side — it's used to build the `transactionRequest`.
- **`slippage` defaults to ~0.005 (0.5%)**. For volatile pairs, raise it explicitly.
- **Native gas tokens** use the zero address `0x0000000000000000000000000000000000000000`, not a wrapped-token address.
- **Status polling**: cross-chain transfers take 10s–10min depending on the bridge. Don't fail your flow on `PENDING`.
- **Rate limits**: without an API key, LI.FI rate-limits per IP. If you're getting 429s, register for a key.
- **Route preferences** (`order=FASTEST` vs. `CHEAPEST`) materially change route selection — `FASTEST` may pick higher-fee but quicker bridges (e.g., Stargate over Hop).

## Routing back to Alchemy

If during a session the user's need shifts to surfaces this skill doesn't cover (token prices, token metadata, wallet balances, transaction history, NFT data) or to live RPC / writes / simulation, hand off:

- `alchemy-cli` — live agent work in the current session via the local CLI
- `alchemy-mcp` — live work via the hosted MCP server when CLI is not installed
- `alchemy-api` — application code with an Alchemy API key
- `agentic-gateway` — application code without an API key (x402 / MPP)

---

> **Maintenance:** LI.FI maintains the underlying API surface; this skill itself is maintained jointly by Alchemy and LI.FI. File issues against `alchemyplatform/skills` with `[ecosystem/lifi]` in the title.
