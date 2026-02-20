---
id: references/node-enhanced-apis.md
name: 'Enhanced APIs (Alchemy RPC Extensions)'
description: 'Alchemy provides enhanced JSON-RPC methods (prefixed with `alchemy_`) that offer indexed, higher-level data without manual log scanning.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
related:
  - data-apis
  - recipes
updated: 2026-02-05
---
# Enhanced APIs (Alchemy RPC Extensions)

## Summary
Alchemy provides enhanced JSON-RPC methods (prefixed with `alchemy_`) that offer indexed, higher-level data without manual log scanning.

## Primary Use Cases
- Wallet token balances and metadata.
- Transfer history for wallets/contracts.
- NFT ownership and metadata queries.

## When To Use / Not Use
- Use when you need asset analytics or history quickly.
- Avoid when strict canonical data is required and you can afford raw RPC queries.

## Auth & Setup
- Same JSON-RPC endpoint as standard node API: `https://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Endpoints / Methods
Common methods (subset; verify per-chain support):
- `alchemy_getTokenBalances`
- `alchemy_getTokenMetadata`
- `alchemy_getAssetTransfers`
- `alchemy_getTokenAllowance`
- `alchemy_getNFTs`
- `alchemy_getNFTMetadata`
- `alchemy_getNFTsForCollection`
- `alchemy_getOwnersForCollection`

## Parameters
- Most methods accept an address plus optional filters (contract addresses, categories, block range).
- Pagination is commonly supported via `pageKey`.

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_getTokenBalances","params":["0x00000000219ab540356cbb839cbe05303d7705fa"]}'
```

```ts
const url = `https://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`;
const body = {
  jsonrpc: "2.0",
  id: 1,
  method: "alchemy_getAssetTransfers",
  params: [{
    fromBlock: "0x0",
    toBlock: "latest",
    category: ["erc20"],
    toAddress: "0x00000000219ab540356cbb839cbe05303d7705fa",
    withMetadata: true,
    maxCount: "0x3e8",
  }],
};

const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});
const json = await res.json();
```

## Error Handling
- Handle JSON-RPC errors, and pagination limits via `pageKey`.

## Performance / Limits / Compute Units
- Enhanced APIs are efficient but still compute-unit metered.
- Prefer filters over large ranges where possible.

## Gotchas & Edge Cases
- Availability varies by network.
- NFT APIs may exclude spam NFTs unless configured.

## Testing / Mocking
- Snapshot responses and use fixed block ranges.

## Agentic Gateway (x402)
These enhanced APIs (Token API, Transfers API) are available through Alchemy's Agentic Gateway without an API key.
- Base URL: `https://x402.alchemy.com/{chainNetwork}/v2` (same JSON-RPC endpoint)
- Auth: `Authorization: SIWE <token>` header (see [gateway-authentication](gateway-authentication.md))
- Payment: x402 USDC micropayment (see [gateway-payment](gateway-payment.md))

## Related Files
- `data-overview.md`
- `recipes-overview.md`
- `gateway-overview.md`
- `gateway-reference.md`

## Official Docs
- [Enhanced APIs Overview](https://www.alchemy.com/docs/reference/enhanced-apis-overview)
