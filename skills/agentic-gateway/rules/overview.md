# Gateway Overview

The Alchemy Agentic Gateway lets agents easily access Alchemy's developer platform, authenticating with SIWE and paying per-request with USDC via the x402 protocol.

## Base URL

```
https://x402.alchemy.com
```

The gateway exposes four API routes:

| Route | Example | Description |
|-------|---------|-------------|
| `/:chainNetwork/v2` | `/eth-mainnet/v2` | Node JSON-RPC + Token API + Transfers API |
| `/:chainNetwork/nft/v3/*` | `/eth-mainnet/nft/v3/getNFTsForOwner?owner=0x...` | NFT API v3 (REST) |
| `/data/v1/*` | `/data/v1/assets/tokens/by-address` | Portfolio API (not chain-specific) |
| `/prices/v1/*` | `/prices/v1/tokens/by-symbol?symbols=ETH` | Prices API (current + historical) |

See [reference](reference.md) for all endpoints, supported chains, and available methods.

## End-to-End Flow

1. **Set up a wallet** — Use an existing Ethereum wallet or generate a new one with `npx @alchemy/x402 wallet generate`.
2. **Fund the wallet** — Load USDC on a supported payment network (Base Mainnet or Base Sepolia for testnet).
3. **Create a SIWE token** — Run `npx @alchemy/x402 sign-siwe --private-key ./wallet-key.txt` or use `signSiwe()` in code.
4. **Send a request** — Call any gateway route with the `Authorization: SIWE <token>` header. For quick queries without an npm project, see the [curl-workflow](curl-workflow.md) for a lightweight curl-based alternative.
5. **Handle 402 Payment Required** — If the gateway returns 402, create an x402 payment with `npx @alchemy/x402 pay` or `createPayment()` and retry with a `Payment-Signature` header.
6. **Receive the result** — After payment, the gateway proxies the request to Alchemy and returns the result. Subsequent requests with the same SIWE token do not require payment again.

## Packages

### `@alchemy/x402` — CLI + Library (recommended)

```bash
npm install @alchemy/x402
```

Provides both CLI commands and library utilities for wallet management, SIWE authentication, and x402 payments:

| CLI command | Library function | Purpose |
|-------------|-----------------|---------|
| `npx @alchemy/x402 wallet generate` | `generateWallet()` | Create a new wallet |
| `npx @alchemy/x402 wallet import` | `getWalletAddress()` | Import / verify a wallet |
| `npx @alchemy/x402 sign-siwe` | `signSiwe()` | Generate a SIWE auth token |
| `npx @alchemy/x402 pay` | `createPayment()` | Create an x402 payment from a 402 response |
| — | `buildX402Client()` | Create an x402 client for use with `@x402/fetch` or `@x402/axios` |

### Additional packages for app development

For building applications with automatic payment handling, also install a fetch/axios wrapper:

```bash
npm install @alchemy/x402 @x402/fetch   # or @x402/axios
```

| Package | Purpose |
|---------|---------|
| `@x402/fetch` | `wrapFetchWithPayment` — auto-handles 402 → sign → retry with `fetch` |
| `@x402/axios` | `wrapAxiosWithPayment` — auto-handles 402 → sign → retry with `axios` |
