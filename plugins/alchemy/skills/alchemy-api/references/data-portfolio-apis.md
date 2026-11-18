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
updated: 2026-09-02
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
| `includeBlockMetadata` | boolean | No | `false` | Include per-network block metadata (block number, hash, timestamp) alongside the token data. See "Block metadata" below. |
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
| `includeBlockMetadata` | boolean | No | `false` | Include per-network block metadata (block number, hash, timestamp) at the response root. See "Block metadata" below. |
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

## `POST /transactions/history/by-address` (Beta)

Returns transaction history for a wallet across networks.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `addresses` | array | Yes | — | Wallet/network pairs (max 1 pair, max 2 networks) |
| `addresses[].address` | string | Yes | — | Wallet address |
| `addresses[].networks` | string[] | Yes | — | Network slugs (beta: `eth-mainnet`, `base-mainnet` only) |
| `before` | string | No | — | Cursor for previous page |
| `after` | string | No | — | Cursor for next page |
| `limit` | integer | No | `25` | Results per page (max 50) |

### Request

```bash
curl -s -X POST "https://api.g.alchemy.com/data/v1/$ALCHEMY_API_KEY/transactions/history/by-address" \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [{
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "networks": ["eth-mainnet"]
    }],
    "limit": 5
  }'
```

### Response

```json
{
  "transactions": [
    {
      "network": "eth-mainnet",
      "hash": "0x3847245c01829b043431067fb2bfa95f7b5bdc7e...",
      "timeStamp": "2025-06-01T10:30:00.000Z",
      "blockNumber": "20000000",
      "blockHash": "0x...",
      "nonce": "42",
      "transactionIndex": "5",
      "fromAddress": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
      "toAddress": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
      "contractAddress": null,
      "value": "0",
      "gasUsed": "46000",
      "effectiveGasPrice": "15000000000",
      "cumulativeGasUsed": "1234567",
      "logs": [],
      "internalTxns": []
    }
  ],
  "before": "cursor_prev...",
  "after": "cursor_next...",
  "totalCount": 500
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `transactions` | array | Transaction objects |
| `transactions[].network` | string | Network slug |
| `transactions[].hash` | string | Transaction hash |
| `transactions[].timeStamp` | string | ISO 8601 timestamp |
| `transactions[].blockNumber` | string | Block number |
| `transactions[].fromAddress` | string | Sender address |
| `transactions[].toAddress` | string | Recipient address |
| `transactions[].contractAddress` | string | Deployed contract address (null if not a deployment) |
| `transactions[].value` | string | ETH value transferred |
| `transactions[].gasUsed` | string | Gas used |
| `transactions[].effectiveGasPrice` | string | Effective gas price |
| `transactions[].logs` | array | Event logs |
| `transactions[].internalTxns` | array | Internal transactions |
| `before` | string | Cursor for previous page |
| `after` | string | Cursor for next page |
| `totalCount` | integer | Total transactions |

---

## Notes

- Use the smallest `networks` list you need — each network adds to response time.
- NFT metadata can be missing or malformed. Always handle null fields.
- The Transactions endpoint is in beta and only supports Ethereum and Base mainnets.
- Paginate large wallets using `pageKey` (for asset endpoints) or `before`/`after` (for transactions).

## Block metadata (`includeBlockMetadata`)

Both multichain token endpoints — `POST /assets/tokens/by-address` and `POST /assets/tokens/balances/by-address` — accept an optional `includeBlockMetadata: true` on the request body. When set, the response carries a top-level `blockMetadata` object keyed by network slug, with the block that each per-network balance snapshot was read at:

```json
{
  "data": { "tokens": [ ... ] },
  "blockMetadata": {
    "eth-mainnet": {
      "blockNumber": "0x18a6825",
      "blockHash": "0xb9587b181217d4c32c45e614d85c92c857dac76b57e15d36fb7d38b27dcecb92",
      "blockTimestamp": "2026-08-27T16:45:59.000Z"
    },
    "matic-mainnet": {
      "blockNumber": "0x5877288",
      "blockHash": "0x4ddbc1d12e14909468c49efd2771d1a09dec2850a823e0d9942db54d54633e43",
      "blockTimestamp": "2026-08-27T16:46:07.000Z"
    }
  }
}
```

* Default is `false` (opt-in). Requests that omit the flag get the pre-existing response shape.
* `blockNumber` is `0x`-hex; `blockTimestamp` is ISO-8601 UTC.
* Only **successful** networks appear in `blockMetadata`. Networks that failed the multichain fan-out show up in `error.partialErrors[]` (see "Partial failures" below) and are absent from `blockMetadata`.
* `blockMetadata` is the anchor to use when reconciling balances across chains — the snapshot is not guaranteed to be at head; each network's block is captured independently by the fan-out.

## Partial failures

The four multichain fan-out endpoints (`assets/tokens/by-address`, `assets/tokens/balances/by-address`, `assets/nfts/by-address`, `assets/nfts/contracts/by-address`) return **HTTP 200** with an optional `error.partialErrors[]` array when one or more networks in the request fail (upstream error or per-network timeout: ~5s paginated / ~10s fetch-all).

```json
{
  "data": { "tokens": [ /* successful networks only */ ] },
  "error": {
    "partialErrors": [
      { "network": "polygon-mainnet", "reason": "upstream_timeout" }
    ]
  }
}
```

Key semantics:

- `error` is **omitted** (not present-and-null) when every network in the request succeeds. Check for the key's presence, not for null.
- Networks may **repeat** in `partialErrors` if the same network had multiple failure modes.
- **Failed networks are dropped from pagination** — subsequent `pageKey` calls do not retry them.
- Native tokens have `tokenAddress: null`.

Client retry pattern: on receiving `error.partialErrors`, retry only the failed networks in a fresh request (do NOT retry the whole payload — that double-counts costs and rewinds pagination on successful networks). Surface partial data to the user, and cap retries per network.

## Other ways to access this API

This file is part of the `alchemy-api` skill — for app code that ships with an Alchemy API key. Other paths to the same data:

- **Live agent work via CLI** (preferred when `@alchemy/cli` is installed locally — see the `alchemy-cli` skill):
  ```bash
  alchemy portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet","base-mainnet"]}]}' --json --no-interactive
  alchemy portfolio token-balances --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  alchemy portfolio nfts --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  alchemy portfolio nft-contracts --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'
  ```
  Note: the CLI does not (yet) wrap the Beta `transactions/history/by-address` endpoint; call it via HTTP using the `alchemy-api` API-key URL or the gateway URLs below.

- **Live agent work via MCP** (when MCP is wired into your client and the CLI is not installed — see the `alchemy-mcp` skill). Call `select_app` first, then any of:
  `getTokensByAddress`, `getTokenBalancesByAddress`, `getNFTsByAddress`, `getNFTContractsByAddress` (full list in the `alchemy-mcp` skill catalog).

- **App code without an API key** (autonomous agent paying per-request, or explicit x402/MPP — see the `agentic-gateway` skill): same API exposed at `https://x402.alchemy.com/data/v1/...` (x402) or `https://mpp.alchemy.com/data/v1/...` (MPP).

## Official Docs
- [Portfolio APIs Overview](https://www.alchemy.com/docs/reference/portfolio-apis)
- [Tokens by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-tokens-by-address)
- [Token Balances by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-token-balances-by-address)
- [NFTs by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nfts-by-address)
- [NFT Contracts by Address](https://www.alchemy.com/docs/data/portfolio-apis/portfolio-api-endpoints/portfolio-api-endpoints/get-nft-contracts-by-address)
- [Transaction History by Address (Beta)](https://www.alchemy.com/docs/data/beta-apis/beta-api-endpoints/beta-api-endpoints/get-transaction-history-by-address)
