# LI.FI Routing & Discovery Reference

**Base URL:** `https://li.quest/v1` · **Auth:** `x-lifi-api-key: $LIFI_API_KEY` (optional)

These endpoints are non-overlapping with Alchemy first-party. For token prices, token metadata, balances, transfers, or live RPC reads, route to `alchemy-api`.

---

## `GET /quote`

Single-step quote with `transactionRequest` populated. Easiest path for "swap X on chain A to Y on chain B".

### Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `fromChain` | number | Yes | Source chain ID |
| `toChain` | number | Yes | Destination chain ID |
| `fromToken` | string | Yes | Source token address (use `0x0000…` for native gas token) |
| `toToken` | string | Yes | Destination token address |
| `fromAmount` | string | Yes | Amount in smallest unit (no decimals applied) |
| `fromAddress` | string | Yes | Sender wallet address |
| `toAddress` | string | No | Recipient address (defaults to `fromAddress`) |
| `slippage` | number | No | Slippage tolerance (`0.005` = 0.5%; default ~0.5%) |
| `integrator` | string | No | Your integrator ID (used for fee accounting) |
| `order` | string | No | `FASTEST` or `CHEAPEST` (default: `RECOMMENDED`) |
| `allowBridges` / `denyBridges` / `preferBridges` | string[] | No | Filter or steer bridge selection |
| `allowExchanges` / `denyExchanges` / `preferExchanges` | string[] | No | Filter or steer DEX selection |
| `allowDestinationCall` | boolean | No | Allow contract calls on destination (default `true`) |
| `fee` | number | No | Integrator fee (`0.02` = 2%, max < 100%) |
| `maxPriceImpact` | number | No | Max price impact threshold (`0.1` = 10%; default 10%) |

### Example

```bash
curl "https://li.quest/v1/quote?\
fromChain=42161&\
toChain=10&\
fromToken=0xaf88d065e77c8cC2239327C5EDb3A432268e5831&\
toToken=0xDA10009cBd5D07dd0CeCc66161FC93D7c9000da1&\
fromAmount=10000000&\
fromAddress=0xYourAddress&\
slippage=0.005&\
order=CHEAPEST" \
  -H "x-lifi-api-key: $LIFI_API_KEY"
```

### Key response fields

- `transactionRequest.{to, data, value, gasLimit, gasPrice, chainId}` — ready for `eth_sendTransaction`
- `estimate.{fromAmount, toAmount, toAmountMin, executionDuration, feeCosts, gasCosts}` — UX preview
- `action.{fromToken, toToken, fromAddress, toAddress, slippage}` — echoes inputs
- `tool` — selected aggregator/bridge name
- `id` — quote ID (handy for support tickets)

---

## `POST /advanced/routes`

Multi-route comparison. Returns routes without `transactionRequest` data — use this when you want to **inspect** options before committing.

### Body

```json
{
  "fromChainId": 42161,
  "toChainId": 10,
  "fromTokenAddress": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "toTokenAddress": "0xDA10009cBd5D07dd0CeCc66161FC93D7c9000da1",
  "fromAmount": "10000000",
  "fromAddress": "0xYourAddress",
  "options": {
    "integrator": "YourAppName",
    "slippage": 0.005,
    "order": "CHEAPEST",
    "bridges": { "allow": ["stargate", "hop", "across"] },
    "exchanges": { "allow": ["1inch", "uniswap"] },
    "allowSwitchChain": true,
    "maxPriceImpact": 0.1
  }
}
```

### Example

```bash
curl -X POST "https://li.quest/v1/advanced/routes" \
  -H "Content-Type: application/json" \
  -H "x-lifi-api-key: $LIFI_API_KEY" \
  -d '{
    "fromChainId": 42161,
    "toChainId": 10,
    "fromTokenAddress": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "toTokenAddress": "0xDA10009cBd5D07dd0CeCc66161FC93D7c9000da1",
    "fromAmount": "10000000",
    "options": { "slippage": 0.005, "integrator": "YourApp" }
  }'
```

### Response

Array of `Route` objects sorted by `order` preference. Each route has `steps[]` describing the bridge / DEX hops, plus aggregate `fromAmount`, `toAmount`, `gasCostUSD`, `executionDuration`. **No `transactionRequest`** — call `/advanced/stepTransaction` to populate the chosen step.

---

## `POST /advanced/stepTransaction`

Populate a specific step from `/advanced/routes` with the `transactionRequest`.

### Body

Send the full `Step` object (from a chosen route's `steps[]`).

### Query parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `skipSimulation` | boolean | No | Skip TX simulation for faster response |

### Response

The same `Step` object with `transactionRequest` now populated.

---

## `GET /status`

Track cross-chain (or same-chain) transfer status. Polling endpoint — invoke after the user submits the `transactionRequest` from `/quote` or `/advanced/stepTransaction`.

### Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `txHash` | string | Yes | Sending tx hash, receiving tx hash, or `transactionId` |
| `fromChain` | number | No | Source chain ID (speeds up resolution) |
| `toChain` | number | No | Destination chain ID |
| `bridge` | string | No | Bridge tool name (from `/quote` response) |

For same-chain swaps, set `fromChain` and `toChain` to the same value.

### Example

```bash
curl "https://li.quest/v1/status?txHash=0xSendingTxHash&fromChain=42161&toChain=10"
```

### Response

```json
{
  "transactionId": "...",
  "sending":    { "txHash": "0x...", "amount": "...", "chainId": 42161, "token": {...} },
  "receiving":  { "txHash": "0x...", "amount": "...", "chainId": 10,    "token": {...} },
  "status":     "DONE",
  "substatus":  "COMPLETED",
  "lifiExplorerLink": "https://scan.li.fi/tx/..."
}
```

### Status values

`NOT_FOUND` · `INVALID` · `PENDING` · `DONE` · `FAILED`

Poll until `DONE` or `FAILED`. Substatuses include `WAIT_SOURCE_CONFIRMATIONS`, `WAIT_DESTINATION_TRANSACTION`, `BRIDGE_NOT_AVAILABLE`, `CHAIN_NOT_AVAILABLE`, `REFUNDED`, `COMPLETED` — see [li.fi docs](https://docs.li.fi/) for the full list.

---

## `GET /chains`

List supported chains.

```bash
curl "https://li.quest/v1/chains"
curl "https://li.quest/v1/chains?chainTypes=EVM,SVM"
```

Returns array of `Chain` objects with `id`, `name`, `key`, `chainType`, `nativeToken`, `metamask` (RPC / explorer URLs), `multicallAddress`, etc.

## `GET /tokens`

List tokens for one or more chains.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `chains` | string | No | Comma-separated chain IDs or keys (e.g., `1,137` or `ETH,POL`) |
| `chainTypes` | string | No | Filter by chain types: `EVM`, `SVM` |
| `minPriceUSD` | number | No | Min token price filter (default `0.0001`) |

```bash
curl "https://li.quest/v1/tokens?chains=1,137,42161"
```

> **Note:** for general token metadata lookup outside the swap-routing context, prefer `alchemy-api` (Token API). LI.FI's token list is curated for routing purposes — it omits unroutable tokens.

## `GET /token`

Resolve a single token.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `chain` | string | Yes | Chain ID or key |
| `token` | string | Yes | Token address or symbol |

```bash
curl "https://li.quest/v1/token?chain=POL&token=DAI"
```

## `GET /tools`

Available bridges + DEXs.

```bash
curl "https://li.quest/v1/tools"
```

Returns `bridges[]` and `exchanges[]` with names, supported chains, fees, etc. Useful when the user asks "what bridges does LI.FI use?" or to constrain `allowBridges` / `allowExchanges` in `/quote`.

## `GET /connections`

Which chain pairs / token pairs are routable.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `fromChain` | number | No | Source chain |
| `toChain` | number | No | Destination chain |
| `fromToken` | string | No | Source token |
| `toToken` | string | No | Destination token |

```bash
curl "https://li.quest/v1/connections?fromChain=1&toChain=137"
```

Useful for "is X → Y supported?" checks before constructing a quote.

---

## Common gotchas

- `/history` and similar reads (transfer history) — **not exposed by LI.FI**. Route to `alchemy-api` (Transfers API).
- Token prices are sometimes returned alongside `/quote` responses for UX — these are **routing-context prices**, not authoritative. For pricing, route to `alchemy-api` (Prices API).
- `transactionRequest.value` is in hex wei. The wallet handles the conversion when signing.
- For SVM (Solana), responses include Solana-shaped transaction encodings — handle separately from EVM.
- Bridge status can flip from `PENDING` → `DONE` and back to `PENDING` if the destination chain reorgs (rare). Treat `DONE` as eventually-final, not strictly-final.
