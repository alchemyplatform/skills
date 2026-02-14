---
id: references/solana-das-api.md
name: 'Solana DAS (Digital Asset Standard) API'
description: 'DAS provides normalized access to Solana NFT and compressed asset data.'
tags:
  - alchemy
  - solana
related:
  - solana-rpc.md
updated: 2026-02-05
---
# Solana DAS (Digital Asset Standard) API

## Summary
DAS provides normalized access to Solana NFT and compressed asset data.

## Primary Use Cases
- Fetch NFT assets for a wallet.
- Query collections or groups.
- Resolve asset metadata and ownership.

## Auth & Setup
- DAS uses the Solana JSON-RPC endpoint with DAS methods.

## Common Methods
- `getAsset`
- `getAssetsByOwner`
- `getAssetsByGroup`
- `searchAssets`
- `getAssetProof`
- `getSignaturesForAsset`

## Example Requests
```bash
curl -s https://solana-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getAssetsByOwner","params":{"ownerAddress":"4Nd1m...","page":1,"limit":100}}'
```

## Gotchas
- Asset grouping fields vary by program.
- Pagination is required for large wallets.

## Related Files
- `solana-rpc.md`

## Official Docs
- [DAS APIs for Solana](https://www.alchemy.com/docs/reference/alchemy-das-apis-for-solana)
