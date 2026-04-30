---
name: uniswap
description: Plan and execute Uniswap swaps via the Uniswap Trading API — get quotes (CLASSIC AMM or UniswapX intent-based routing), check token approvals, build ready-to-sign swap transactions, and generate Uniswap interface deep-links for user-driven flows. Supports Uniswap V2/V3/V4 across all Uniswap-supported chains. NOT for token prices, token metadata, current wallet balances, transaction history, NFT data, live RPC reads, or pool TVL/volume reads — for those use `alchemy-cli` (live), `alchemy-mcp`, `alchemy-api` (app code), or `agentic-gateway` (no API key). For cross-chain bridges, use the `lifi` ecosystem skill. Requires `UNISWAP_API_KEY`.
license: MIT
compatibility: API key required via `$UNISWAP_API_KEY` (`x-api-key` header) — register at https://developers.uniswap.org/. Network access required. Endpoints return ready-to-sign transactions or UniswapX orders; the user / app signs and submits.
metadata:
  author: uniswap
  version: "0.1"
  provider: uniswap
  partner: "true"
---

# Uniswap (Swap Routing + Planning)

Uniswap's Trading API exposes the same routing engine that powers app.uniswap.org — CLASSIC AMM swaps, UniswapX intent-based orders (Dutch V2 / V3 / Priority), WRAP/UNWRAP, and BRIDGE. This skill covers programmatic swap planning, quoting, and transaction construction. For token prices, metadata, balances, transfers, NFTs, or live RPC reads, use the corresponding Alchemy skill instead. For cross-chain bridges, use the `lifi` ecosystem skill.

| | |
| --- | --- |
| **Trading API base** | `https://trade-api.gateway.uniswap.org/v1` |
| **Required auth** | `x-api-key: $UNISWAP_API_KEY` + `x-universal-router-version: 2.0` headers |
| **Protocols** | Uniswap V2, V3, V4 (CLASSIC) + UniswapX (DUTCH_V2, DUTCH_V3, PRIORITY) |
| **Universal Router** | `0x66a9893cC07D91D95644AEDD05D03f95e1dBA8Af` (and per-chain deployments) |

## When to use this skill

Use `uniswap` when **any** of the following are true:

- The user wants to **swap tokens on Uniswap** (V2 / V3 / V4 or UniswapX intent)
- The user wants a **best-price quote** with optional auto-routing across protocols
- The user wants to **plan a swap** and hand off a deep-link to the Uniswap interface
- The user wants to **check ERC-20 approval** before a swap
- The user wants **WRAP / UNWRAP** ETH ↔ WETH

## When NOT to use this skill (handoff)

| Need | Use instead |
| --- | --- |
| Cross-chain bridges / multi-chain swap routing | `lifi` (ecosystem skill) |
| Token prices (current, historical, by-timestamp) | `alchemy-api` (Prices API) |
| Token metadata, search, list by chain | `alchemy-api` (Token API) |
| Current wallet balances | `alchemy-api` (Portfolio / Token API) |
| Transaction history (transfers in / out) | `alchemy-api` (Transfers API) |
| NFT metadata / floor / ownership | `alchemy-api` (NFT API) |
| Live RPC reads, pool state, gas estimation | `alchemy-cli` (live work) or `alchemy-api` (JSON-RPC) |
| Pool TVL / volume / fee-tier analytics | `alchemy-api` (JSON-RPC) + a subgraph indexer |
| viem / ethers / wagmi setup, chain configuration | `alchemy-api` ecosystem references (`ecosystem-viem.md`, etc.) |
| Pre-execution simulation | `alchemy-api` (Simulation API) |
| Account abstraction (bundlers, gas managers) | `alchemy-api` |
| Signed transaction submission | the app's wallet / `alchemy-api` |

## Scope contract

**This skill covers (`scope_in`):**

- `POST /check_approval` — ERC-20 approval check (returns approval tx if needed)
- `POST /quote` — single-call quote across CLASSIC and UniswapX routing types, with `routingPreference` and `protocols` filters
- `POST /swap` — populate the quote into a ready-to-sign transaction (CLASSIC) or signed UniswapX order
- WRAP / UNWRAP routing types for ETH ↔ WETH
- Swap planning + Uniswap-interface deep-link generation (`https://app.uniswap.org/swap?...`)

**This skill does NOT cover (`scope_out`):**

- Cross-chain bridges → handoff: `lifi` (ecosystem skill)
- Token prices → handoff: `alchemy-api` (Prices API)
- Token metadata, search, list → handoff: `alchemy-api` (Token API)
- Wallet balances → handoff: `alchemy-api` (Portfolio / Token API)
- Transaction transfer history → handoff: `alchemy-api` (Transfers API)
- NFT data → handoff: `alchemy-api` (NFT API)
- Live RPC reads, pool state, gas estimation → handoff: `alchemy-cli` or `alchemy-api` (JSON-RPC)
- viem / ethers / wagmi setup and chain configuration → handoff: `alchemy-api` ecosystem references (we don't reproduce viem coverage here)
- Pre-execution simulation → handoff: `alchemy-api` (Simulation API)
- Account abstraction → handoff: `alchemy-api` (Wallets / Bundler / Gas Manager)
- Signed tx submission → user wallet (Trading API returns ready-to-sign payloads; app signs)

## Setup

API key is **required** for the Trading API. Register at the [Uniswap Developer Portal](https://developers.uniswap.org/) and store the key:

```bash
export UNISWAP_API_KEY="..."
```

Required headers on **every** Trading API request:

```text
Content-Type: application/json
x-api-key: $UNISWAP_API_KEY
x-universal-router-version: 2.0
```

> **Security:** never expose `UNISWAP_API_KEY` in client-side code. Server-side only.

## Endpoint reference → [references/trading-api.md](./references/trading-api.md)

### Programmatic swap (Trading API)

| Endpoint | Use for |
| --- | --- |
| `POST /check_approval` | Check if `tokenIn` is approved for the Universal Router. Returns an approval tx if needed. |
| `POST /quote` | Single quote spanning CLASSIC AMM + UniswapX intent routing. Honors `protocols`, `routingPreference`, `slippageTolerance`, `urgency`. |
| `POST /swap` | Convert a quote into a signed UniswapX order or a ready-to-sign CLASSIC tx. |

### Routing types (selectable via `routingPreference` + `protocols`)

| Type | Notes | Chains |
| --- | --- | --- |
| `CLASSIC` | Standard AMM swap through Uniswap V2/V3/V4 pools | All Uniswap-supported chains |
| `DUTCH_V2` | UniswapX Dutch auction V2 (intent-based, gasless for swapper) | Ethereum, Arbitrum, Base, Unichain |
| `DUTCH_V3` | UniswapX Dutch auction V3 | Ethereum |
| `PRIORITY` | UniswapX MEV-protected priority order | Base, Unichain |
| `WRAP` / `UNWRAP` | ETH ↔ WETH conversion | All |
| `BRIDGE` | Cross-chain — **scope_out, prefer `lifi`** | Limited |

### Swap planning + deep-links

For interactive flows where the user signs in the Uniswap interface, generate a deep-link instead of a Trading API tx:

```
https://app.uniswap.org/swap?
  inputCurrency=<address-or-symbol>&
  outputCurrency=<address>&
  exactAmount=<human-readable>&
  exactField=input&
  chain=<chain-name>
```

## Quick examples

### Quote (USDC → DAI on Ethereum, CLASSIC routing)

```bash
curl -X POST "https://trade-api.gateway.uniswap.org/v1/quote" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "x-universal-router-version: 2.0" \
  -d '{
    "swapper": "0xYourAddress",
    "tokenIn":  "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "tokenOut": "0x6B175474E89094C44Da98b954EedeAC495271d0F",
    "tokenInChainId":  "1",
    "tokenOutChainId": "1",
    "amount": "1000000",
    "type": "EXACT_INPUT",
    "slippageTolerance": 0.5,
    "routingPreference": "CLASSIC",
    "protocols": ["V3", "V4"]
  }'
```

### Quote (best price, UniswapX may win on Ethereum)

```bash
# routingPreference=BEST_PRICE often returns UniswapX (DUTCH_V2/V3/PRIORITY)
# rather than CLASSIC; the response shape differs — see references/trading-api.md
curl -X POST "https://trade-api.gateway.uniswap.org/v1/quote" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "x-universal-router-version: 2.0" \
  -H "Content-Type: application/json" \
  -d '{ "swapper": "0x...", "tokenIn": "0x...", "tokenOut": "0x...",
        "tokenInChainId": "1", "tokenOutChainId": "1",
        "amount": "1000000000000000000", "type": "EXACT_INPUT",
        "slippageTolerance": 0.5, "routingPreference": "BEST_PRICE" }'
```

### Approval check + swap construction

```bash
# 1. Check approval
curl -X POST "https://trade-api.gateway.uniswap.org/v1/check_approval" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "walletAddress": "0x...", "token": "0xA0b86991...", "amount": "1000000", "chainId": 1 }'

# 2. Get quote (above)

# 3. Build swap tx — spread the quote response into the body, strip permitData null fields
# See references/trading-api.md for the exact CLASSIC vs UniswapX wiring
```

### Deep-link to Uniswap interface

```text
https://app.uniswap.org/swap?inputCurrency=ETH&outputCurrency=0x...&exactAmount=0.1&exactField=input&chain=base
```

## Common gotchas

- `tokenInChainId` and `tokenOutChainId` are **strings**, not numbers (e.g., `"1"` not `1`).
- `slippageTolerance` is a percentage 0–100, not a decimal (`0.5` = 0.5%, **not** 50%).
- Both `signature` and `permitData` must be **both present or both absent** on CLASSIC `/swap` calls. Strip `permitData: null` before sending.
- UniswapX (`DUTCH_V2`/`DUTCH_V3`/`PRIORITY`) responses have **no `quote.output.amount`** — use `quote.orderInfo.outputs[0].startAmount` (best fill) and `endAmount` (auction floor).
- UniswapX is **gasless for the swapper** — there is no `tx.gas` to display; show the auction window (`startTime` → `deadline`) instead.
- Don't manually convert `gasFee` (wei) using a hardcoded ETH price — use the API-returned `gasFeeUSD` string for display. Hardcoded conversion has been observed to mis-estimate by ~9000x.
- Universal Router version header is required: `x-universal-router-version: 2.0`. Older versions of the API used a different router and won't work.

## Routing back to Alchemy

If during a session the user's need shifts to surfaces this skill doesn't cover (token prices, token metadata, balances, transaction history, NFT data) or to live RPC / pool state / writes / simulation, hand off:

- `alchemy-cli` — live agent work in the current session via the local CLI
- `alchemy-mcp` — live work via the hosted MCP server when CLI is not installed
- `alchemy-api` — application code with an Alchemy API key
- `agentic-gateway` — application code without an API key (x402 / MPP)

For cross-chain bridges or multi-chain swap routing, hand off to the `lifi` ecosystem skill (covers 27 bridges + 31 DEXs across 60+ chains).

---

> **Maintenance:** Uniswap maintains the underlying Trading API; this skill itself is maintained jointly by Alchemy and Uniswap. File issues against `alchemyplatform/skills` with `[ecosystem/uniswap]` in the title.
