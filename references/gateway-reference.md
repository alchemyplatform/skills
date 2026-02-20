# Reference

## Endpoints

The gateway supports the following APIs. All require SIWE auth and x402 payment.

| Route | Method | Description |
|-------|--------|-------------|
| `/:chainNetwork/v2` | POST | Node JSON-RPC — standard RPC methods plus Token API, Transfers API, and Simulation API (via JSON-RPC) |
| `/:chainNetwork/nft/v3/*` | GET/POST | [NFT API v3](https://www.alchemy.com/docs/reference/nft-api-quickstart) — REST endpoints for NFT data |
| `/data/v1/assets/*` | POST | [Portfolio API](https://www.alchemy.com/docs/reference/portfolio-apis) — multi-chain portfolio data (not chain-specific) |
| `/prices/v1/tokens/*` | GET/POST | [Prices API](https://www.alchemy.com/docs/reference/prices-api-quickstart) — token prices, historical prices (GET for current by symbol, POST for by-address and historical) |

**Base URL**: `https://x402.alchemy.com`

Chain-specific routes use the chain slug in the URL (e.g. `https://x402.alchemy.com/eth-mainnet/v2`). Non-chain-specific routes omit it (e.g. `https://x402.alchemy.com/data/v1/assets/tokens/by-address`).

---

## Node JSON-RPC (`/:chainNetwork/v2`)

Standard Ethereum JSON-RPC methods are proxied to Alchemy. Common methods:

- `eth_blockNumber` — latest block number
- `eth_getBalance` — native token balance
- `eth_call` — read-only contract call
- `eth_getTransactionByHash` — transaction details
- `eth_getTransactionReceipt` — transaction receipt
- `eth_estimateGas` — gas estimation
- `eth_getLogs` — event logs

Additionally, Alchemy's enhanced JSON-RPC methods are available:

**Token API** ([docs](https://www.alchemy.com/docs/reference/token-api-quickstart)):
- `alchemy_getTokenBalances` — ERC-20 token balances for an address
- `alchemy_getTokenMetadata` — token contract metadata (name, symbol, decimals, logo)
- `alchemy_getTokenAllowance` — token allowance by spender for an owner

**Transfers API** ([docs](https://www.alchemy.com/docs/reference/transfers-api-quickstart)):
- `alchemy_getAssetTransfers` — historical transfers (ETH, ERC-20, ERC-721, ERC-1155)

These are all called as JSON-RPC POST requests to `/:chainNetwork/v2`.

### `eth_blockNumber` — request & response

```
POST https://x402.alchemy.com/eth-mainnet/v2
Content-Type: application/json
```

```json
{ "id": 1, "jsonrpc": "2.0", "method": "eth_blockNumber" }
```

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x175db18" }
```

### `eth_getBalance` — request & response

```
POST https://x402.alchemy.com/eth-mainnet/v2
Content-Type: application/json
```

```json
{ "id": 1, "jsonrpc": "2.0", "method": "eth_getBalance", "params": ["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045", "latest"] }
```

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x1bdcb3c9a4fbbdeff" }
```

### `alchemy_getTokenMetadata` — request & response

```
POST https://x402.alchemy.com/eth-mainnet/v2
Content-Type: application/json
```

```json
{ "id": 1, "jsonrpc": "2.0", "method": "alchemy_getTokenMetadata", "params": ["0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"] }
```

```json
{
  "jsonrpc": "2.0", "id": 1,
  "result": {
    "decimals": 6,
    "logo": "https://static.alchemyapi.io/images/assets/3408.png",
    "name": "USD Coin",
    "symbol": "USDC"
  }
}
```

### `alchemy_getTokenBalances` — request & response

```
POST https://x402.alchemy.com/eth-mainnet/v2
Content-Type: application/json
```

```json
{ "id": 1, "jsonrpc": "2.0", "method": "alchemy_getTokenBalances", "params": ["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045", "erc20"] }
```

```json
{
  "jsonrpc": "2.0", "id": 1,
  "result": {
    "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
    "tokenBalances": [
      {
        "contractAddress": "0x000000000000e63d2c9c29d3edf6efb99071f92c",
        "tokenBalance": "0x00000000000000000000000000000000000000000000001b1ae4d6e2ef500000"
      }
    ]
  }
}
```

### `alchemy_getAssetTransfers` — request & response

```
POST https://x402.alchemy.com/eth-mainnet/v2
Content-Type: application/json
```

```json
{
  "id": 1, "jsonrpc": "2.0", "method": "alchemy_getAssetTransfers",
  "params": [{
    "fromBlock": "0x0", "toBlock": "latest",
    "fromAddress": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "category": ["external"], "maxCount": "0x2"
  }]
}
```

```json
{
  "jsonrpc": "2.0", "id": 1,
  "result": {
    "transfers": [
      {
        "blockNum": "0x4dc40",
        "hash": "0x6ff0860e202c61189cb2a3a38286bffd694acbc50577df6cb5a7ff40e21ea074",
        "from": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "to": "0x7e2d0fe0ffdd78c264f8d40d19acb7d04390c6e8",
        "value": 0.00012,
        "asset": "ETH",
        "category": "external"
      }
    ]
  }
}
```

---

## NFT API v3 (`/:chainNetwork/nft/v3/*`)

REST endpoints for querying NFT data. Full docs: [NFT API reference](https://www.alchemy.com/docs/reference/nft-api-endpoints).

**Ownership:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/:chainNetwork/nft/v3/getNFTsForOwner` | All NFTs owned by an address |
| GET | `/:chainNetwork/nft/v3/getOwnersForNFT` | Owners of a specific NFT |
| GET | `/:chainNetwork/nft/v3/getOwnersForContract` | All owners for a contract |
| GET | `/:chainNetwork/nft/v3/isHolderOfContract` | Check if address holds an NFT from a contract |
| GET | `/:chainNetwork/nft/v3/getContractsForOwner` | NFT contracts owned by an address |
| GET | `/:chainNetwork/nft/v3/getCollectionsForOwner` | NFT collections owned by an address |

**Metadata:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/:chainNetwork/nft/v3/getNFTsForContract` | All NFTs in a contract |
| GET | `/:chainNetwork/nft/v3/getNFTMetadata` | Metadata for a single NFT |
| POST | `/:chainNetwork/nft/v3/getNFTMetadataBatch` | Metadata for multiple NFTs |
| GET | `/:chainNetwork/nft/v3/getContractMetadata` | Contract-level metadata |
| POST | `/:chainNetwork/nft/v3/getContractMetadataBatch` | Metadata for multiple contracts |
| GET | `/:chainNetwork/nft/v3/searchContractMetadata` | Search NFT contracts by name |
| GET | `/:chainNetwork/nft/v3/computeRarity` | Rarity scores for an NFT |
| GET | `/:chainNetwork/nft/v3/summarizeNFTAttributes` | Attribute summary for a collection |

**Spam:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/:chainNetwork/nft/v3/getSpamContracts` | Known spam NFT contracts |
| GET | `/:chainNetwork/nft/v3/isSpamContract` | Check if a contract is spam |
| GET | `/:chainNetwork/nft/v3/isAirdropNFT` | Check if an NFT is an airdrop |

**Sales:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/:chainNetwork/nft/v3/getFloorPrice` | Floor price for a collection |
| GET | `/:chainNetwork/nft/v3/getNFTSales` | Recent sales for an NFT or collection |

### `getNFTMetadata` — request & response

```
GET https://x402.alchemy.com/eth-mainnet/nft/v3/getNFTMetadata?contractAddress=0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D&tokenId=1
```

```json
{
  "contract": {
    "address": "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D",
    "name": "BoredApeYachtClub",
    "symbol": "BAYC",
    "tokenType": "ERC721"
  },
  "tokenId": "1",
  "tokenType": "ERC721",
  "name": null,
  "description": null,
  "image": {
    "cachedUrl": "https://ipfs.io/ipfs/QmPbxeGcXhYQQNgsC6a36dDyYUcHgMLnGKnF8pVFmGsvqi",
    "originalUrl": "https://ipfs.io/ipfs/QmPbxeGcXhYQQNgsC6a36dDyYUcHgMLnGKnF8pVFmGsvqi"
  }
}
```

### `getContractMetadata` — request & response

```
GET https://x402.alchemy.com/eth-mainnet/nft/v3/getContractMetadata?contractAddress=0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D
```

```json
{
  "address": "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D",
  "name": "BoredApeYachtClub",
  "symbol": "BAYC",
  "tokenType": "ERC721",
  "totalSupply": "10000"
}
```

### `getFloorPrice` — request & response

```
GET https://x402.alchemy.com/eth-mainnet/nft/v3/getFloorPrice?contractAddress=0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D
```

```json
{
  "openSea": {
    "floorPrice": 6.05,
    "priceCurrency": "ETH",
    "collectionUrl": "https://opensea.io/collection/boredapeyachtclub",
    "retrievedAt": "2026-02-20T22:21:30.244Z"
  },
  "looksRare": {
    "floorPrice": 48.2,
    "priceCurrency": "ETH",
    "collectionUrl": "https://looksrare.org/collections/0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d",
    "retrievedAt": "2023-05-17T18:54:41.736Z"
  }
}
```

---

## Portfolio API (`/data/v1/assets/*`)

Multi-chain REST endpoints — query across multiple networks in a single request. Full docs: [Portfolio API reference](https://www.alchemy.com/docs/reference/portfolio-apis).

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/data/v1/assets/tokens/by-address` | Fungible tokens (native + ERC-20 + SPL) with metadata and prices |
| POST | `/data/v1/assets/tokens/balances/by-address` | Token balances only (lighter response) |
| POST | `/data/v1/assets/nfts/by-address` | NFTs across multiple chains |
| POST | `/data/v1/assets/nfts/contracts/by-address` | NFT collections across multiple chains |

### `tokens/by-address` — request & response

```
POST https://x402.alchemy.com/data/v1/assets/tokens/by-address
Content-Type: application/json
```

```json
{
  "addresses": [
    { "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045", "networks": ["eth-mainnet"] }
  ]
}
```

```json
{
  "data": {
    "tokens": [
      {
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "network": "eth-mainnet",
        "tokenAddress": null,
        "tokenBalance": "0x000000000000000000000000000000000000000000000001bdcb3c9a4fbbdeff",
        "tokenMetadata": { "symbol": null, "decimals": null, "name": null, "logo": null },
        "tokenPrices": [
          { "currency": "usd", "value": "1967.60", "lastUpdatedAt": "2026-02-20T22:21:36Z" }
        ]
      },
      {
        "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
        "network": "eth-mainnet",
        "tokenAddress": "0x000000000000e63d2c9c29d3edf6efb99071f92c",
        "tokenBalance": "0x00000000000000000000000000000000000000000000001b1ae4d6e2ef500000",
        "tokenMetadata": { "decimals": 18, "logo": null, "name": "Holy Trinity", "symbol": "HOLY" },
        "tokenPrices": [
          { "currency": "usd", "value": "20.99", "lastUpdatedAt": "2025-08-11T18:18:55Z" }
        ]
      }
    ]
  }
}
```

Note: native token (ETH) has `tokenAddress: null`. ERC-20 tokens have the contract address.

---

## Prices API (`/prices/v1/tokens/*`)

Token price data. Full docs: [Prices API reference](https://www.alchemy.com/docs/reference/prices-api-quickstart).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/prices/v1/tokens/by-symbol?symbols=ETH,BTC` | Current prices by token symbol (up to 25 symbols) |
| POST | `/prices/v1/tokens/by-address` | Current prices by contract address — **must be POST with JSON body** |
| POST | `/prices/v1/tokens/historical` | Historical prices with intervals (`5m`, `1h`, `1d`) — **must be POST with JSON body** |

### `tokens/by-symbol` — request & response

```
GET https://x402.alchemy.com/prices/v1/tokens/by-symbol?symbols=ETH,BTC
```

```json
{
  "data": [
    {
      "symbol": "ETH",
      "prices": [
        { "currency": "usd", "value": "1970.69", "lastUpdatedAt": "2025-06-01T12:00:00Z" }
      ]
    },
    {
      "symbol": "BTC",
      "prices": [
        { "currency": "usd", "value": "67727.36", "lastUpdatedAt": "2025-06-01T12:00:00Z" }
      ]
    }
  ]
}
```

### `tokens/by-address` — request & response

```
POST https://x402.alchemy.com/prices/v1/tokens/by-address
Content-Type: application/json
```

```json
{
  "addresses": [
    { "network": "eth-mainnet", "address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48" }
  ]
}
```

```json
{
  "data": [
    {
      "network": "eth-mainnet",
      "address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      "prices": [
        { "currency": "usd", "value": "0.9998838804", "lastUpdatedAt": "2026-02-20T22:22:06Z" }
      ]
    }
  ]
}
```

### `tokens/historical` — request & response

```
POST https://x402.alchemy.com/prices/v1/tokens/historical
Content-Type: application/json
```

```json
{
  "symbol": "ETH",
  "startTime": "2025-01-01T00:00:00Z",
  "endTime": "2025-01-07T00:00:00Z",
  "interval": "1h"
}
```

```json
{
  "symbol": "ETH",
  "currency": "usd",
  "data": [
    { "value": "3350.12", "timestamp": "2025-01-01T00:01:13.233Z" },
    { "value": "3348.50", "timestamp": "2025-01-01T01:03:18.732Z" }
  ]
}
```

Note: `data` is a flat array of `{ value, timestamp }` objects sorted chronologically. The response does **not** include OHLCV — only single price points per interval.

---

## Chain Network Slugs

Use these as the `:chainNetwork` path parameter for chain-specific routes (`/v2` and `/nft/v3`):

### EVM Chains

| Chain | Mainnet | Testnet |
|-------|---------|---------|
| Ethereum | `eth-mainnet` | `eth-sepolia` |
| Base | `base-mainnet` | `base-sepolia` |
| Polygon | `polygon-mainnet` | `polygon-amoy` |
| BNB | `bnb-mainnet` | `bnb-testnet` |
| Arbitrum | `arb-mainnet` | `arb-sepolia` |
| Optimism | `opt-mainnet` | `opt-sepolia` |
| World Chain | `worldchain-mainnet` | `worldchain-sepolia` |
| Tempo | `tempo-mainnet` | `tempo-testnet` |
| Hyperliquid | `hyperliquid-mainnet` | `hyperliquid-testnet` |
| MegaETH | `megaeth-mainnet` | `megaeth-testnet` |
| Monad | `monad-mainnet` | `monad-testnet` |

### Non-EVM Chains

| Chain | Mainnet | Testnet |
|-------|---------|---------|
| Solana | `solana-mainnet` | `solana-devnet` |

## Payment Networks

Payments are made on these networks (independent of which chain you're querying):

| Network | CAIP-2 ID | USDC Address | EIP-712 Domain Name |
|---------|-----------|--------------|---------------------|
| Base Sepolia (testnet) | `eip155:84532` | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` | `USDC` |
| Base Mainnet | `eip155:8453` | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | `USD Coin` |
| Ethereum Mainnet | `eip155:1` | `0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` | `USD Coin` |

**Gateway receiving wallet**: `0x658dc531A7FE637F7BA31C3dDd4C9bf8A27c81e5`

## Request Headers (Client → Gateway)

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `SIWE <base64(siwe_message)>.<signature>` |
| `Content-Type` | Yes | `application/json` |
| `Accept` | Recommended | `application/json` |
| `Payment-Signature` | On payment | Base64-encoded x402 payment payload |

## Response Headers (Gateway → Client)

| Header | When | Description |
|--------|------|-------------|
| `X-Protocol-Version` | Always on success | `x402/2.0` |
| `PAYMENT-REQUIRED` | 402 responses | Encoded payment requirements |
| `PAYMENT-RESPONSE` | After successful payment | Encoded settlement result (transaction hash, network, payer) |

## HTTP Status Codes

| Status | Meaning |
|--------|---------|
| 200 | Request proxied successfully |
| 401 | SIWE authentication failed (see [gateway-authentication](gateway-authentication.md) for error codes) |
| 402 | Payment required — respond with a `Payment-Signature` header |
| 404 | Invalid chain network slug or route |
| 500 | Internal gateway error |

## Testnet Funding

For development on Base Sepolia:

1. Get testnet ETH from a Base Sepolia faucet for gas
2. Get testnet USDC from the [Circle USDC faucet](https://faucet.circle.com/) — select "Base Sepolia"
3. USDC address on Base Sepolia: `0x036CbD53842c5426634e7929541eC2318f3dCF7e`
