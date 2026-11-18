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
updated: 2026-09-23
---
# Portfolio APIs

Consolidated wallet views (tokens, NFTs, transaction history) across multiple networks in single requests. All endpoints are `POST` with JSON bodies.

**Base URL**: `https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY`

**Supported chains** (in `networks` array): `eth-mainnet`, `base-mainnet`, `polygon-mainnet`, `arb-mainnet`, `opt-mainnet`, `sol-mainnet`, and more.

---

## `POST /assets/tokens/by-address`

Returns fungible tokens (native + ERC-20 + SPL) with balances, prices, and metadata for one or more wallets across multiple networks.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `addresses` | array | Yes | — | Wallet/network pairs (max 2 addresses, max 5 networks each) |
| `addresses[].address` | string | Yes | — | Wallet address |
| `addresses[].networks` | string[] | Yes | — | Network slugs to query |
| `withMetadata` | boolean | No | `true` | Include token metadata (name, symbol, decimals, logo) |
| `withPrices` | boolean | No | `true` | Include token prices |
| `includeNativeTokens` | boolean | No | `true` | Include native tokens (ETH, MATIC, etc.) |
| `includeErc20Tokens` | boolean | No | `true` | Include ERC-20 tokens |
| `pageKey` | string | No | — | Pagination cursor |

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/tokens/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "networks": ["eth-mainnet", "base-mainnet", "arb-mainnet"]
    }]
  }'
```

### Response

```json
{
  "data": {
    "tokens": [
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "tokenAddress": null,
        "tokenBalance": "0x000000000000000000000000000000000000000000000001bdd951817729c4f9",
        "tokenMetadata": {
          "symbol": null,
          "decimals": null,
          "name": null,
          "logo": null
        },
        "tokenPrices": [
          {
            "currency": "usd",
            "value": "1970.69",
            "lastUpdatedAt": "2025-06-01T12:00:00Z"
          }
        ]
      },
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "tokenAddress": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
        "tokenBalance": "0x000000000000000000000000000000000000000000000000000000003b9aca00",
        "tokenMetadata": {
          "name": "USD Coin",
          "symbol": "USDC",
          "decimals": 6,
          "logo": "https://..."
        },
        "tokenPrices": [
          {
            "currency": "usd",
            "value": "0.9998",
            "lastUpdatedAt": "2025-06-01T12:00:00Z"
          }
        ]
      }
    ],
    "pageKey": null
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.tokens` | array | Token objects |
| `data.tokens[].network` | string | Network slug |
| `data.tokens[].address` | string | Wallet address |
| `data.tokens[].tokenAddress` | string | Token contract address (`null` for native token) |
| `data.tokens[].tokenBalance` | string | Raw balance (hex, use token decimals to convert) |
| `data.tokens[].tokenMetadata` | object | Token name, symbol, decimals, logo |
| `data.tokens[].tokenPrices` | array | Price entries per currency (each has currency, value, lastUpdatedAt) |
| `data.pageKey` | string | Pagination cursor (null if no more results) |

---

## `POST /assets/tokens/balances/by-address`

Returns raw token balances (without prices/metadata). More efficient when you only need balances.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `addresses` | array | Yes | — | Wallet/network pairs (max 3 pairs, max 20 networks total) |
| `addresses[].address` | string | Yes | — | Wallet address |
| `addresses[].networks` | string[] | Yes | — | Network slugs |
| `includeNativeTokens` | boolean | No | `true` | Include native tokens |
| `includeErc20Tokens` | boolean | No | `true` | Include ERC-20 tokens |
| `pageKey` | string | No | — | Pagination cursor |

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/tokens/balances/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "networks": ["eth-mainnet", "base-mainnet", "arb-mainnet", "opt-mainnet", "polygon-mainnet"]
    }]
  }'
```

### Response

```json
{
  "data": {
    "tokens": [
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "tokenAddress": null,
        "tokenBalance": "0x112210f47de98000"
      },
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "tokenAddress": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
        "tokenBalance": "0x3b9aca00"
      }
    ],
    "pageKey": null
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.tokens[].network` | string | Network slug |
| `data.tokens[].address` | string | Wallet address |
| `data.tokens[].tokenAddress` | string | Token contract address (`null` for native token) |
| `data.tokens[].tokenBalance` | string | Raw balance (hex) |
| `data.pageKey` | string | Pagination cursor |

---

## `POST /assets/nfts/by-address`

Returns NFTs owned by a wallet across multiple networks.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `addresses` | array | Yes | — | Wallet/network pairs (max 2 pairs, max 15 networks each) |
| `addresses[].address` | string | Yes | — | Wallet address |
| `addresses[].networks` | string[] | Yes | — | Network slugs |
| `excludeFilters` | string[] | No | `["SPAM"]` | Exclude `"SPAM"`, `"AIRDROPS"` |
| `includeFilters` | string[] | No | — | Include only `"SPAM"`, `"AIRDROPS"` (mutually exclusive with excludeFilters) |
| `spamConfidenceLevel` | string | No | — | `"VERY_HIGH"`, `"HIGH"`, `"MEDIUM"`, `"LOW"` |
| `withMetadata` | boolean | No | `true` | Include NFT metadata |
| `pageKey` | string | No | — | Pagination cursor |
| `pageSize` | integer | No | `100` | Results per page (max 100) |
| `orderBy` | string | No | — | `"transferTime"` |
| `sortOrder` | string | No | — | `"asc"` or `"desc"` |

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/nfts/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "networks": ["eth-mainnet"]
    }],
    "withMetadata": true,
    "pageSize": 2
  }'
```

### Response

```json
{
  "data": {
    "ownedNfts": [
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "contract": {
          "address": "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d",
          "name": "BoredApeYachtClub",
          "symbol": "BAYC",
          "tokenType": "ERC721",
          "openSeaMetadata": { "collectionName": "Bored Ape Yacht Club", "floorPrice": 10.5 },
          "isSpam": false
        },
        "tokenId": "1234",
        "tokenType": "ERC721",
        "name": "Bored Ape #1234",
        "description": "...",
        "image": { "cachedUrl": "https://nft-cdn.alchemy.com/..." },
        "collection": { "name": "Bored Ape Yacht Club" },
        "acquiredAt": { "blockTimestamp": "2024-03-15T10:30:00.000Z" }
      }
    ],
    "totalCount": 42,
    "pageKey": "..."
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.ownedNfts` | array | NFT objects (same structure as NFT API's `getNFTsForOwner` plus `network` and `address` fields) |
| `data.ownedNfts[].network` | string | Network slug |
| `data.ownedNfts[].address` | string | Wallet address |
| `data.ownedNfts[].contract` | object | Contract info (address, name, symbol, tokenType, openSeaMetadata, isSpam) |
| `data.ownedNfts[].tokenId` | string | Token ID |
| `data.ownedNfts[].name` | string | NFT name |
| `data.ownedNfts[].image` | object | Image URLs |
| `data.ownedNfts[].acquiredAt` | object | Acquisition timestamp |
| `data.totalCount` | integer | Total NFTs across all pages |
| `data.pageKey` | string | Pagination cursor |

---

## `POST /assets/nfts/contracts/by-address`

Returns NFT collections (contracts) owned by a wallet across multiple networks.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `addresses` | array | Yes | — | Wallet/network pairs (max 2 pairs, max 15 networks each) |
| `addresses[].address` | string | Yes | — | Wallet address |
| `addresses[].networks` | string[] | Yes | — | Network slugs |
| `withMetadata` | boolean | No | `true` | Include contract metadata |
| `pageKey` | string | No | — | Pagination cursor |
| `pageSize` | integer | No | `100` | Results per page (max 100) |
| `orderBy` | string | No | — | `"transferTime"` |
| `sortOrder` | string | No | — | `"asc"` or `"desc"` |

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/assets/nfts/contracts/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "networks": ["eth-mainnet"]
    }],
    "withMetadata": true
  }'
```

### Response

```json
{
  "data": {
    "contracts": [
      {
        "network": "eth-mainnet",
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "contract": {
          "address": "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d",
          "name": "BoredApeYachtClub",
          "symbol": "BAYC",
          "totalSupply": "10000",
          "tokenType": "ERC721",
          "contractDeployer": "0xaba7161a7fb69c88e16ed9f455ce62b791ee4d03",
          "deployedBlockNumber": 12287507,
          "openSeaMetadata": {
            "collectionName": "Bored Ape Yacht Club",
            "floorPrice": 10.5
          },
          "isSpam": false,
          "totalBalance": "1",
          "numDistinctTokensOwned": "1",
          "displayNft": {
            "tokenId": "1234",
            "name": "Bored Ape #1234"
          },
          "image": {
            "cachedUrl": "https://nft-cdn.alchemy.com/...",
            "thumbnailUrl": "https://nft-cdn.alchemy.com/...",
            "pngUrl": "https://nft-cdn.alchemy.com/...",
            "contentType": "image/png",
            "originalUrl": "ipfs://..."
          }
        }
      }
    ],
    "totalCount": 5,
    "pageKey": null
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data.contracts` | array | Contract objects |
| `data.contracts[].network` | string | Network slug |
| `data.contracts[].address` | string | Wallet address |
| `data.contracts[].contract` | object | Contract info (same as NFT API `getContractMetadata`) |
| `data.contracts[].contract.totalBalance` | string | Total NFTs owned from this contract |
| `data.contracts[].contract.numDistinctTokensOwned` | string | Number of distinct token IDs owned |
| `data.contracts[].contract.displayNft` | object | Representative NFT (tokenId, name) |
| `data.contracts[].contract.image` | object | Image URLs for the display NFT |
| `data.totalCount` | integer | Total collections |
| `data.pageKey` | string | Pagination cursor |

---

## `POST /transfers/by-address` (Beta)

Returns historical transfers (native, ERC-20, ERC-721, ERC-1155, special NFT, and internal) for a single wallet address across multiple EVM networks, merged and ordered by `blockTimestamp`. This endpoint replaces the earlier `POST /transactions/history/by-address` beta.

**Beta:** Do not rely on this endpoint in production. Access is currently gated — email [data_apis@alchemy.com](mailto:data_apis@alchemy.com) to request access.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `address` | string | Yes | — | Single wallet address (not an array) |
| `category` | string[] | Yes | — | One or more of `external`, `internal`, `erc20`, `erc721`, `erc1155`, `specialnft`. Empty array is rejected. |
| `networks` | string[] | No | `["eth-mainnet", "base-mainnet", "matic-mainnet"]` | Up to 5 networks per request. |
| `direction` | string | No | `both` | `to`, `from`, or `both`. Filters transfers by which side `address` appears on. |
| `fromTimestamp` | string | No | genesis | ISO-8601 timestamp (inclusive lower bound). |
| `toTimestamp` | string | No | now | ISO-8601 timestamp (inclusive upper bound). |
| `order` | string | No | `desc` | `asc` or `desc` by `blockTimestamp`. |
| `contractAddresses` | string[] | No | — | Filter to specific token contracts. Applies to `erc20`, `erc721`, `erc1155`, `specialnft`. |
| `excludeZeroValue` | boolean | No | `true` | Exclude zero-value transfers. |
| `maxCount` | integer | No | `100` | Page size. |
| `pageKey` | string | No | — | Opaque cursor from a prior response. |

**Supported networks (14 EVM):** `eth-mainnet`, `matic-mainnet`, `arb-mainnet`, `opt-mainnet`, `base-mainnet`, `bnb-mainnet`, `celo-mainnet`, `ronin-mainnet`, `monad-mainnet`, `monad-testnet`, `robinhood-mainnet`, `robinhood-testnet`, `arc-mainnet`, `arc-testnet`.

**`internal` category coverage:** narrower than the others — only `eth-mainnet`, `base-mainnet`, `matic-mainnet`, `arc-mainnet`, and `arc-testnet`. Requesting `internal` alongside networks outside that set returns transfers only from the supported subset (the other networks succeed but emit no internal transfers).

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/transfers/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "networks": ["eth-mainnet", "base-mainnet", "matic-mainnet"],
    "category": ["external", "erc20", "erc721", "erc1155"],
    "fromTimestamp": "2026-08-01T00:00:00Z",
    "toTimestamp": "2026-08-06T00:00:00Z",
    "order": "desc",
    "maxCount": 100
  }'
```

### Response

```json
{
  "transfers": [
    {
      "network": "base-mainnet",
      "blockNum": "0x1410a3b",
      "blockTimestamp": "2026-08-05T23:58:41Z",
      "category": "erc20",
      "hash": "0xabc123...",
      "from": "0x1f98abcd...",
      "to": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "contractAddress": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "uniqueId": "0xabc123...:log:12",
      "fungible": { "rawValue": "0xee6b2800", "value": 250.0, "decimals": 6, "asset": "USDC" },
      "nfts": null
    },
    {
      "network": "base-mainnet",
      "blockNum": "0x1410a10",
      "blockTimestamp": "2026-08-05T23:50:03Z",
      "category": "erc721",
      "hash": "0x9a8b7c...",
      "from": "0x0000000000000000000000000000000000000000",
      "to": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "contractAddress": "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D",
      "uniqueId": "0x9a8b7c...:log:0",
      "fungible": null,
      "nfts": { "tokens": [{ "tokenId": "0x1a4", "amount": "1" }] }
    }
  ],
  "pageKey": "eyJ0cyI6IjIw..."
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `transfers[]` | array | Transfers merged across all requested networks, sorted by `blockTimestamp`. Empty array if nothing matches. |
| `transfers[].network` | string | Network the transfer occurred on. |
| `transfers[].blockNum` | string | Block number as hex — not comparable across networks. |
| `transfers[].blockTimestamp` | string | ISO-8601 block timestamp. Cross-chain sort key; always returned. |
| `transfers[].category` | string | `external`, `internal`, `erc20`, `erc721`, `erc1155`, or `specialnft`. |
| `transfers[].hash` | string | Transaction hash. |
| `transfers[].from` | string | Sender. |
| `transfers[].to` | string \| null | Recipient. `null` on contract creation. |
| `transfers[].contractAddress` | string \| null | Token contract address. `null` for native-token transfers. |
| `transfers[].uniqueId` | string | Stable per-transfer id. Formats: `{hash}:log:{logIndex}` (erc20/721/1155/specialnft), `{hash}:external`, `{hash}:internal:{traceAddress}`. |
| `transfers[].fungible` | object \| null | Populated for `external`, `internal`, `erc20`. Fields: `rawValue` (hex), `value` (number, scaled), `decimals`, `asset`. Any inner field can be `null` if metadata is missing. |
| `transfers[].nfts` | object \| null | Populated for `erc721`, `erc1155`, `specialnft`. `tokens[]` with `tokenId` (hex) + `amount`. |
| `pageKey` | string \| null | Cursor for next page. `null` when no more pages. |
| `error.partialErrors[]` | array | Present only when one or more networks failed. Each entry has `network`, `state` (e.g. `timeout`), `message`. |

### Handling partial failures

Multichain fan-out endpoints return HTTP 200 even when some networks fail. When every network succeeds the `error` key is OMITTED (not present-and-null). Failed networks are dropped from pagination — retry them in a fresh request. See the shared partial-failure pattern below.

---

## Notes

- Use the smallest `networks` list you need — each network adds to response time.
- NFT metadata can be missing or malformed. Always handle null fields.
- The Transfers By Wallet endpoint is in beta (email `data_apis@alchemy.com` for access) and supports 14 EVM networks — see the endpoint parameter table for the full list.
- Paginate large wallets using `pageKey` (for asset endpoints and `/transfers/by-address`) or `before`/`after` (for legacy transaction endpoints).

## Other ways to access this API

This file is part of the `alchemy-api` skill — for app code that ships with an Alchemy API key. Other paths to the same data:

- **Live agent work via CLI** (preferred when `@alchemy/cli` is installed locally — see the `alchemy-cli` skill):
  ```bash
  alchemy portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet","base-mainnet"]}]}' --json --no-interactive
  alchemy portfolio token-balances --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  alchemy portfolio nfts --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  alchemy portfolio nft-contracts --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  ```
  Note: the CLI does not wrap the Beta `/transfers/by-address` endpoint yet; call it via HTTP using the `alchemy-api` API-key URL or the gateway URLs below.

- **Live agent work via MCP** (when MCP is wired into your client and the CLI is not installed — see the `alchemy-mcp` skill). Call `select_app` first, then any of:
  `getTokensByAddress`, `getTokenBalancesByAddress`, `getNFTsByAddress`, `getNFTContractsByAddress` (full list in the `alchemy-mcp` skill catalog).

- **App code without an API key** (autonomous agent paying per-request, or explicit x402/MPP — see the `agentic-gateway` skill): same API exposed at `https://x402.alchemy.com/data/v1/...` (x402) or `https://mpp.alchemy.com/data/v1/...` (MPP).

## Official Docs
- [Portfolio APIs Overview](https://www.alchemy.com/docs/reference/portfolio-apis)
- [Tokens by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-tokens-by-address)
- [Token Balances by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-token-balances-by-address)
- [NFTs by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nfts-by-address)
- [NFT Contracts by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nft-contracts-by-address)
- [Transfers by Wallet (Beta)](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-transfers-by-address)
