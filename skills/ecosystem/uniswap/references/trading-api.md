# Uniswap Trading API Reference

**Base URL:** `https://trade-api.gateway.uniswap.org/v1`

**Required headers (every request):**

```text
Content-Type: application/json
x-api-key: $UNISWAP_API_KEY
x-universal-router-version: 2.0
```

These endpoints are non-overlapping with Alchemy first-party. For prices, metadata, balances, transfers, or NFTs, route to `alchemy-api`. For cross-chain bridges, route to `lifi`.

---

## `POST /check_approval`

Check whether `tokenIn` has Universal Router approval for the swap amount.

### Body

```json
{
  "walletAddress": "0xYourAddress",
  "token":         "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "amount":        "1000000",
  "chainId":       1
}
```

### Response

```json
{
  "approval": {
    "to":      "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "from":    "0xYourAddress",
    "data":    "0x095ea7b3...",
    "value":   "0",
    "chainId": 1
  }
}
```

If `approval` is `null`, the token is already approved — proceed to `/quote`.

---

## `POST /quote`

Single-call quote across CLASSIC AMM and UniswapX intent routing.

### Body

```json
{
  "swapper":           "0xYourAddress",
  "tokenIn":           "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "tokenOut":          "0x6B175474E89094C44Da98b954EedeAC495271d0F",
  "tokenInChainId":    "1",
  "tokenOutChainId":   "1",
  "amount":            "1000000",
  "type":              "EXACT_INPUT",
  "slippageTolerance": 0.5,
  "routingPreference": "BEST_PRICE",
  "protocols":         ["V2", "V3", "V4"],
  "urgency":           "normal"
}
```

### Key parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `swapper` | string | Yes | User wallet address |
| `tokenIn` / `tokenOut` | string | Yes | Token addresses (`0x000...000` for native ETH) |
| `tokenInChainId` / `tokenOutChainId` | **string** | Yes | Chain IDs as **strings**, not numbers |
| `amount` | string | Yes | Amount in smallest unit (no decimals) |
| `type` | string | Yes | `EXACT_INPUT` or `EXACT_OUTPUT` |
| `slippageTolerance` | number | Conditional | 0–100 percentage. Required if `autoSlippage` is `false` or absent. `0.5` = 0.5%, **not 50%** |
| `autoSlippage` | boolean | No | `true` lets the API pick — overrides `slippageTolerance` |
| `routingPreference` | string | No | `BEST_PRICE` (default) · `FASTEST` · `CLASSIC` (forces AMM only) |
| `protocols` | string[] | No | `["V2", "V3", "V4"]` for CLASSIC routing |
| `urgency` | string | No | `normal` or `fast` — affects UniswapX auction timing |

### CLASSIC response (AMM swap)

```json
{
  "routing": "CLASSIC",
  "quote": {
    "input":          { "token": "0x...", "amount": "1000000" },
    "output":         { "token": "0x...", "amount": "999000" },
    "slippage":       0.5,
    "route":          [ /* per-pool hops */ ],
    "gasFee":         "5000000000000000",
    "gasFeeUSD":      "0.01",
    "gasUseEstimate": "150000"
  },
  "permitData": null
}
```

### UniswapX response (DUTCH_V2 / DUTCH_V3 / PRIORITY)

```json
{
  "routing": "DUTCH_V2",
  "quote": {
    "orderInfo": {
      "reactor":  "0x...",
      "swapper":  "0x...",
      "nonce":    "...",
      "deadline": 1772031054,
      "cosigner": "0x...",
      "input":   { "token": "0x...", "startAmount": "1000000000000000000",
                   "endAmount":   "1000000000000000000" },
      "outputs": [{ "token": "0x...", "startAmount": "999000",
                    "endAmount": "994000", "recipient": "0x..." }],
      "chainId": 1
    },
    "encodedOrder": "0x...",
    "orderHash":    "0x..."
  },
  "permitData": { "domain": {}, "types": {}, "values": {} }
}
```

> **Critical:** UniswapX has **no `quote.output.amount`**. Use `quote.orderInfo.outputs[0].startAmount` (best fill) and `endAmount` (auction floor) for display. Accessing `quote.output.amount` on a UniswapX response throws at runtime.

### Routing types

| `routing` value | Notes | Chains |
| --- | --- | --- |
| `CLASSIC` | Standard AMM swap through V2/V3/V4 pools | All Uniswap-supported chains |
| `DUTCH_V2` | UniswapX Dutch auction V2; gasless for swapper | Ethereum, Arbitrum, Base, Unichain |
| `DUTCH_V3` | UniswapX Dutch auction V3 | Ethereum |
| `PRIORITY` | UniswapX MEV-protected priority order | Base, Unichain |
| `WRAP` / `UNWRAP` | ETH ↔ WETH | All |
| `BRIDGE` | Cross-chain — **prefer `lifi` ecosystem skill** | Limited |

---

## `POST /swap`

Build the executable transaction (CLASSIC) or signed UniswapX order from a quote.

### Body — spread the quote response, strip null fields

```typescript
const quoteResponse = await fetchQuote(params);

// Always strip permitData / permitTransaction; handle them per routing type
const { permitData, permitTransaction, ...cleanQuote } = quoteResponse;
const swapRequest: Record<string, unknown> = { ...cleanQuote };

const isUniswapX =
  quoteResponse.routing === "DUTCH_V2" ||
  quoteResponse.routing === "DUTCH_V3" ||
  quoteResponse.routing === "PRIORITY";

if (isUniswapX) {
  // UniswapX: signature only — permitData must NOT go to /swap
  if (permit2Signature) swapRequest.signature = permit2Signature;
} else {
  // CLASSIC: both signature and permitData, or neither (never partial)
  if (permit2Signature && permitData && typeof permitData === "object") {
    swapRequest.signature  = permit2Signature;
    swapRequest.permitData = permitData;
  }
}
```

> **Critical:** do not wrap the quote in `{ quote: quoteResponse }`. Spread it.

### Permit2 rules (CLASSIC)

- `signature` and `permitData` must **both** be present or **both** be absent.
- Never set `permitData: null` — omit the field entirely.
- `permitData: null` is common in quote responses; strip it before sending.

### UniswapX rules

`permitData` is used **locally** to sign the order, but must be **excluded** from the `/swap` body. Sign Permit2 client-side, send only `signature` to `/swap`.

### Response

```json
{
  "swap": {
    "to":       "0xUniversalRouter",
    "from":     "0xSwapper",
    "data":     "0x...",
    "value":    "0",
    "chainId":  1,
    "gasLimit": "250000"
  }
}
```

CLASSIC returns a ready-to-sign tx the wallet submits. UniswapX returns a signed order that fillers compete for off-chain (no on-chain tx until a filler executes).

---

## Deep-link generator (no API call)

For interactive flows where the user signs in the Uniswap interface:

```text
https://app.uniswap.org/swap?
  inputCurrency=<address-or-symbol>&
  outputCurrency=<address>&
  exactAmount=<human-readable-decimal>&
  exactField=input&
  chain=<chain-name>
```

Examples:

```text
https://app.uniswap.org/swap?inputCurrency=ETH&outputCurrency=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48&exactAmount=0.1&exactField=input&chain=mainnet

https://app.uniswap.org/swap?inputCurrency=ETH&outputCurrency=0x...&exactAmount=10&exactField=input&chain=base
```

Use this when:

- The user explicitly wants to swap on the Uniswap interface (not via your app)
- You don't want to handle Permit2 / Universal Router calldata yourself
- The user has their wallet connected to app.uniswap.org already

---

## Common gotchas

- **String chain IDs**: `tokenInChainId: "1"` not `tokenInChainId: 1`. Numbers will be rejected.
- **Slippage units**: `slippageTolerance: 0.5` is 0.5%, not 50%. Range 0–100.
- **UniswapX response shape**: no `quote.output` field; use `quote.orderInfo.outputs[]`.
- **Gasless for UniswapX**: don't display "estimated gas" on UniswapX swaps — it's $0 for the swapper. Display the auction window.
- **Permit2 atomicity**: signature + permitData are atomic on CLASSIC. Strip null permitData before posting.
- **Universal Router header**: `x-universal-router-version: 2.0` is required; without it requests fail.
- **Native ETH** uses `0x0000000000000000000000000000000000000000` as the token address, not WETH.
- **Cross-chain (`BRIDGE`)** routing exists but is limited; prefer the `lifi` ecosystem skill for serious cross-chain workflows.

---

## Out of scope here

- viem / ethers / wagmi setup, chain configuration, wallet connection — see `alchemy-api` ecosystem references (`ecosystem-viem.md`, `ecosystem-ethers.md`, `ecosystem-wagmi.md`).
- v4 hook authoring, security, SDK integration — out of scope for this curated skill.
- LP / liquidity provision, fee-tier analytics, pool TVL — out of scope; route via subgraphs and `alchemy-api` JSON-RPC.
