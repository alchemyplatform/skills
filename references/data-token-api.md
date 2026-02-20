---
id: references/data-token-api.md
name: 'Token API'
description: 'Fetch token balances, metadata, and allowances without manual contract calls. Token API methods are exposed as Alchemy JSON-RPC methods.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - node-enhanced-apis.md
  - recipes-get-token-balances.md
updated: 2026-02-05
---
# Token API

## Summary
Fetch token balances, metadata, and allowances without manual contract calls. Token API methods are exposed as Alchemy JSON-RPC methods.

## Primary Use Cases
- ERC-20 balances for a wallet.
- Token metadata (name, symbol, decimals, logo).
- Allowance checks for spend approvals.

## When To Use / Not Use
- Use for wallet asset views and portfolio dashboards.
- Avoid if you need a raw contract call or custom ABI data; use `eth_call`.

## Auth & Setup
- JSON-RPC endpoint: `https://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Endpoints / Methods
- `alchemy_getTokenBalances`
- `alchemy_getTokenMetadata`
- `alchemy_getTokenAllowance`

## Spec-Derived Parameters
### `alchemy_getTokenBalances`
- `address` (string, hex address)
- `tokenSpec` (enum: `erc20`, `DEFAULT_TOKENS`, `NATIVE_TOKEN` OR array of token contract addresses)
- `options.pageKey` (string)
- `options.maxCount` (integer, default 100, capped at 100)

### `alchemy_getTokenAllowance`
- `tokenAllowanceRequest.contract` (token contract address)
- `tokenAllowanceRequest.owner` (owner address)
- `tokenAllowanceRequest.spender` (spender address)

### `alchemy_getTokenMetadata`
- `contractAddress` (token contract address)

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_getTokenMetadata","params":["0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"]}'
```

```ts
const url = `https://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`;
const body = {
  jsonrpc: "2.0",
  id: 1,
  method: "alchemy_getTokenBalances",
  params: ["0x00000000219ab540356cbb839cbe05303d7705fa"],
};

const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});
const json = await res.json();
```

## Error Handling
- Handle JSON-RPC errors and rate limits.

## Performance / Limits / Compute Units
- Filter by contract addresses when you know the set.

## Gotchas & Edge Cases
- Token balances can be zero for historical tokens; filter if you want non-zero.

## Testing / Mocking
- Use a known wallet with a small token set for predictable tests.

## Agentic Gateway (x402)
This API is available through Alchemy's Agentic Gateway without an API key.
- Base URL: `https://x402.alchemy.com/{chainNetwork}/v2` (Token API methods are JSON-RPC)
- Auth: `Authorization: SIWE <token>` header (see [gateway-authentication](gateway-authentication.md))
- Payment: x402 USDC micropayment (see [gateway-payment](gateway-payment.md))

## Related Files
- `node-enhanced-apis.md`
- `recipes-get-token-balances.md`
- `gateway-overview.md`
- `gateway-reference.md`

## Official Docs
- [Token API Quickstart](https://www.alchemy.com/docs/docs/reference/token-api-quickstart)
