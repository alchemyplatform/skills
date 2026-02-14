---
id: references/data-portfolio-apis.md
name: 'Portfolio APIs'
description: 'Portfolio APIs provide consolidated wallet views (tokens, NFTs, and transaction history) across multiple networks in single requests.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - data-token-api.md
  - data-nft-api.md
  - data-transfers-api.md
  - recipes-get-portfolio.md
updated: 2026-02-05
---
# Portfolio APIs

## Summary
Portfolio APIs provide consolidated wallet views (tokens, NFTs, and transaction history) across multiple networks in single requests.

## Primary Use Cases
- Build a full wallet portfolio page.
- Multi-chain wallet views without parallel RPC calls.
- Portfolio feeds and wallet analytics.

## When To Use / Not Use
- Use when you want multi-chain asset views in one request.
- Avoid if you only need a single token balance or metadata field; Token API may be cheaper.

## Auth & Setup
- Base URL: `https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY`
- All endpoints below are `POST` with JSON bodies.

## Network Selection (EVM + Solana)
- Each request includes `addresses` entries shaped like `{ address, networks }`.
- `networks` is an array of network enums like `eth-mainnet`, `polygon-mainnet`, `base-mainnet`, `arb-mainnet`, `opt-mainnet`, `sol-mainnet`.

## Endpoints / Methods
### Token Balances By Wallet
- `POST /assets/tokens/balances/by-address`
- Request fields:
- `addresses` (required, limit 3 address/network pairs, max 20 networks total)
- `includeNativeTokens` (optional, default `true`)
- `includeErc20Tokens` (optional, default `true`)
- `pageKey` (optional)
- Response fields:
- `data.tokens[]`: `network`, `address`, `tokenAddress`, `tokenBalance`
- `data.pageKey`

### NFTs By Wallet
- `POST /assets/nfts/by-address`
- Request fields:
- `addresses` (required, limit 2 address/network pairs, max 15 networks each)
- `withMetadata` (optional, default `true`)
- `pageKey` (optional)
- `pageSize` (optional, default `100`)
- `orderBy` (optional, `transferTime`)
- `sortOrder` (optional, `asc` or `desc`)
- Response fields:
- `data.ownedNfts[]`: `network`, `address`, `contract` (includes opensea metadata + spam flags), `tokenId`, `tokenType`, `name`, `description`, `image`, `raw`, `collection`, `tokenUri`, `timeLastUpdated`, `acquiredAt`
- `data.totalCount`, `data.pageKey`

### NFT Collections By Wallet
- `POST /assets/nfts/contracts/by-address`
- Request fields:
- `addresses` (required, limit 2 address/network pairs, max 15 networks each)
- `withMetadata` (optional, default `true`)
- `pageKey` (optional)
- `pageSize` (optional, default `100`)
- `orderBy` (optional, `transferTime`)
- `sortOrder` (optional, `asc` or `desc`)
- Response fields:
- `data.contracts[]`: `network`, `address`, `contract` (includes name, symbol, tokenType, totalSupply, deploy info, opensea metadata, spam flags)
- `data.totalCount`, `data.pageKey`

### Transactions By Wallet (Beta)
- `POST /transactions/history/by-address`
- Request fields:
- `addresses` (required, limit 1 address/network pair, max 2 networks; beta supports ETH and BASE mainnets only)
- `before` (optional cursor)
- `after` (optional cursor)
- `limit` (optional, default `25`, max `50`)
- `pageKey` (optional)
- Response fields:
- `transactions[]`: `network`, `hash`, `timeStamp`, `blockNumber`, `blockHash`, `nonce`, `transactionIndex`, `fromAddress`, `toAddress`, `contractAddress`, `value`, `cumulativeGasUsed`, `effectiveGasPrice`, `gasUsed`, `logs`, `internalTxns`
- `before`, `after`, `totalCount`

## Example Requests
```bash
curl -sX POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/tokens/balances/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0x00000000219ab540356cbb839cbe05303d7705fa",
      "networks": ["eth-mainnet","base-mainnet","arb-mainnet","opt-mainnet","polygon-mainnet"]
    }]
  }'
```

```bash
curl -sX POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/nfts/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0x00000000219ab540356cbb839cbe05303d7705fa",
      "networks": ["eth-mainnet"]
    }],
    "withMetadata": true,
    "pageSize": 100
  }'
```

## Error Handling
- Handle HTTP `429` with backoff.
- Use `pageKey` for pagination.

## Performance / Limits / Compute Units
- Use the smallest `networks` list you need.
- Paginate large wallets.

## Gotchas & Edge Cases
- NFT metadata can be missing or malformed; handle null fields.
- Some networks may not be supported for specific endpoints.

## Testing / Mocking
- Use a small wallet address and a limited network list for deterministic tests.

## Related Files
- `data-token-api.md`
- `data-nft-api.md`
- `data-transfers-api.md`
- `recipes-get-portfolio.md`

## Official Docs
- [Portfolio APIs](https://www.alchemy.com/docs/docs/reference/portfolio-apis)
- [Token Balances By Wallet](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-token-balances-by-address)
- [NFTs By Wallet](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nfts-by-address)
- [NFT Collections By Wallet](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nft-contracts-by-address)
- [Transactions By Wallet (Beta)](https://www.alchemy.com/docs/data/beta-apis/beta-api-endpoints/beta-api-endpoints/get-transaction-history-by-address)
