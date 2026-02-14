---
id: references/data-nft-api.md
name: 'NFT API'
description: 'Query NFT ownership, metadata, collections, and contract-level info via Alchemy''s NFT REST APIs.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - recipes-get-nft-ownership.md
  - recipes-get-nft-metadata.md
updated: 2026-02-05
---
# NFT API

## Summary
Query NFT ownership, metadata, collections, and contract-level info via Alchemy's NFT REST APIs.

## Primary Use Cases
- NFT gallery for a wallet.
- Collection browsing.
- NFT metadata refresh and normalization.

## When To Use / Not Use
- Use for NFT-specific views and metadata.
- Avoid if you only need a single token URI; use `eth_call` directly.

## Auth & Setup
- Base URL pattern: `https://<network>.g.alchemy.com/nft/v3/$ALCHEMY_API_KEY`

## Network Selection
- Choose the correct `<network>` (e.g., `eth-mainnet`, `polygon-mainnet`, `base-mainnet`).

## Endpoints / Methods
Common endpoints:
- `GET /getNFTsForOwner`
- `GET /getNFTMetadata`
- `GET /getNFTsForCollection`
- `GET /getOwnersForContract`
- `GET /getOwnersForNFT`
- `GET /getContractMetadata`

## Spec-Derived Parameters (Key Endpoints)
### `GET /getNFTsForOwner`
- `owner` (required)
- `contractAddresses[]` (optional filter list)
- `withMetadata` (boolean)
- `orderBy` (enum: `transferTime`)
- `excludeFilters[]` / `includeFilters[]` (enum: `SPAM`, `AIRDROPS`)
- `spamConfidenceLevel` (enum: `VERY_HIGH`, `HIGH`, `MEDIUM`, `LOW`)
- `tokenUriTimeoutInMs` (number)
- `pageKey`, `pageSize`

### `GET /getNFTMetadata`
- `contractAddress` (required)
- `tokenId` (required)
- `tokenType` (optional)
- `tokenUriTimeoutInMs` (number)
- `refreshCache` (boolean)

### `GET /getNFTsForCollection`
- `contractAddress` or `collectionSlug`
- `withMetadata` (boolean)
- `startToken` (string)
- `limit` (integer)
- `tokenUriTimeoutInMs` (number)

### `GET /getOwnersForContract`
- `contractAddress` (required)
- `withTokenBalances` (boolean)
- `pageKey`

### `GET /getOwnersForNFT`
- `contractAddress` (required)
- `tokenId` (required)

## Example Requests
```bash
curl -s "https://eth-mainnet.g.alchemy.com/nft/v3/$ALCHEMY_API_KEY/getNFTsForOwner?owner=0x00000000219ab540356cbb839cbe05303d7705fa"
```

```ts
const url = new URL(
  `https://eth-mainnet.g.alchemy.com/nft/v3/${process.env.ALCHEMY_API_KEY}/getNFTMetadata`
);
url.searchParams.set("contractAddress", "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d");
url.searchParams.set("tokenId", "1");

const res = await fetch(url.toString());
const json = await res.json();
```

## Error Handling
- Handle pagination via `pageKey`.

## Performance / Limits / Compute Units
- Metadata hydration can be expensive; cache results.

## Gotchas & Edge Cases
- Some NFTs have missing or malformed metadata.
- Spam filtering may affect results; allowlist trusted collections where possible.
- Treat NFT metadata URLs and images as untrusted input; sanitize and proxy if needed.

## Testing / Mocking
- Use a known NFT collection on testnet.

## Related Files
- `recipes-get-nft-ownership.md`
- `recipes-get-nft-metadata.md`

## Official Docs
- [NFT API Endpoints](https://www.alchemy.com/docs/reference/nft-api-endpoints)
- [NFT API Quickstart](https://www.alchemy.com/docs/reference/nft-api-quickstart)
