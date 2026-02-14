---
id: references/data-transfers-api.md
name: 'Transfers API'
description: 'Query historical transfers across ERC-20/721/1155 and native transfers without manual log scanning.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - node-enhanced-apis.md
  - recipes-get-transfers-history.md
updated: 2026-02-05
---
# Transfers API

## Summary
Query historical transfers across ERC-20/721/1155 and native transfers without manual log scanning.

## Primary Use Cases
- Wallet transfer history.
- Contract-level token flow analytics.
- Activity feeds.

## When To Use / Not Use
- Use for historical transfers and activity feeds.
- Avoid if you only need a single transaction receipt.

## Auth & Setup
- JSON-RPC endpoint: `https://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Endpoints / Methods
- `alchemy_getAssetTransfers`

## Spec-Derived Parameters
`alchemy_getAssetTransfers` accepts a single `assetTransferParams` object with:
- `fromBlock` / `toBlock` (hex block numbers or tags)
- `fromAddress` / `toAddress`
- `excludeZeroValue` (boolean)
- `category` (array enum): `external`, `internal`, `erc20`, `erc721`, `erc1155`, `specialnft` (Note: `internal` is not supported on Base; it is only available on Ethereum Mainnet and Polygon Mainnet.)
- `contractAddresses` (array of token contract addresses)
- `order` (enum: `asc`, `desc`)
- `withMetadata` (boolean)
- `maxCount` (hex string, default `0x3e8` = 1000)
- `pageKey` (string)

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_getAssetTransfers","params":[{"fromBlock":"0x0","toBlock":"latest","toAddress":"0x00000000219ab540356cbb839cbe05303d7705fa","category":["erc20"],"withMetadata":true,"maxCount":"0x3e8"}]}'
```

## Error Handling
- Pagination via `pageKey` for large histories.

## Performance / Limits / Compute Units
- Use narrow block ranges for repeat polling.
- Prefer filters to reduce CU usage.

## Gotchas & Edge Cases
- The `internal` category reflects internal transfers and may be incomplete on some chains.

## Testing / Mocking
- Use fixed block ranges in tests.

## Related Files
- `node-enhanced-apis.md`
- `recipes-get-transfers-history.md`

## Official Docs
- [alchemy_getAssetTransfers](https://www.alchemy.com/docs/reference/alchemy-getassettransfers)
