# Alchemy Skill

Root Agent Skill for integrating [Alchemy](https://www.alchemy.com/) APIs into AI-powered applications. Supports both traditional API key access and the Agentic Gateway for easy agent access to Alchemy's developer platform.

## Repository Structure

This repo uses a single-skill layout:

- `SKILL.md`: unified guide covering both API key and Agentic Gateway access
- `references/`: all reference docs in a flat folder with category-prefixed filenames
  - `references/gateway-*.md`: Agentic Gateway docs (authentication, payments, curl workflow, etc.)
- `spec/`: Agent Skills spec reference
- `template/`: supporting template files

## Getting Started

### Path A: API Key (traditional)
1. Get an API key at [dashboard.alchemy.com](https://dashboard.alchemy.com/).
2. Start with [`SKILL.md`](SKILL.md) → "Path A: API Key Access" for base URLs, auth patterns, and quickstart snippets.

### Path B: Agentic Gateway (x402)
1. Generate a wallet and fund it with USDC.
2. Start with [`SKILL.md`](SKILL.md) → "Path B: Agentic Gateway (x402)" for SIWE authentication and gateway setup.
3. See `references/gateway-overview.md` for the end-to-end flow.

## Endpoint Selector (Top Tasks)

| You need | Use this | Reference | Gateway? |
| --- | --- | --- | --- |
| EVM read/write | JSON-RPC `eth_*` | `references/node-json-rpc.md` | Yes |
| Realtime events | `eth_subscribe` | `references/node-websocket-subscriptions.md` | No |
| Token balances | `alchemy_getTokenBalances` | `references/data-token-api.md` | Yes |
| Token metadata | `alchemy_getTokenMetadata` | `references/data-token-api.md` | Yes |
| Transfers history | `alchemy_getAssetTransfers` | `references/data-transfers-api.md` | Yes |
| NFT ownership | `GET /getNFTsForOwner` | `references/data-nft-api.md` | Yes |
| NFT metadata | `GET /getNFTMetadata` | `references/data-nft-api.md` | Yes |
| Prices (spot) | `GET /tokens/by-symbol` | `references/data-prices-api.md` | Yes |
| Prices (historical) | `POST /tokens/historical` | `references/data-prices-api.md` | Yes |
| Portfolio (multi-chain) | `POST /assets/*/by-address` | `references/data-portfolio-apis.md` | Yes |
| Simulate tx | `alchemy_simulateAssetChanges` | `references/data-simulation-api.md` | Yes |
| Create webhook | `POST /create-webhook` | `references/webhooks-details.md` | No |
| Solana NFT data | `getAssetsByOwner` (DAS) | `references/solana-das-api.md` | No |

## Reference Categories

Category order mirrors `SKILL.md`:

| Category | Overview File | Scope |
| --- | --- | --- |
| **Agentic Gateway** | `references/gateway-overview.md` | Agentic Gateway setup, SIWE auth, x402 payment, wallet setup, curl workflow |
| Node | `references/node-overview.md` | EVM JSON-RPC, websockets, debug/trace, utility methods |
| Data | `references/data-overview.md` | Token, NFT, transfers, prices, portfolio, simulation APIs |
| Webhooks | `references/webhooks-overview.md` | Notify webhook architecture, payloads, and signature verification |
| Solana | `references/solana-overview.md` | Solana RPC, DAS, wallets, and Yellowstone gRPC references |
| Wallets | `references/wallets-overview.md` | Account Kit, smart wallets, bundler, gas manager, wallet APIs |
| Rollups | `references/rollups-overview.md` | High-level Alchemy Rollups guidance |
| Recipes | `references/recipes-overview.md` | End-to-end integration workflows across products |
| Operational | `references/operational-overview.md` | Auth, limits, monitoring, pricing, and production readiness |
| Ecosystem | `references/ecosystem-overview.md` | External library ecosystem across EVM and Solana |

For the full file-by-file map (name + short description for every reference), see `SKILL.md` → `## Skill Map`.

## Specification

This skill follows the [Agent Skills specification](https://agentskills.io/specification). See [spec/agent-skills-spec.md](spec/agent-skills-spec.md) for details.

## Official Links

- [Developer docs](https://www.alchemy.com/docs)
- [Get Started guide](https://www.alchemy.com/docs/get-started)
- [Create a free API key](https://dashboard.alchemy.com/)

## License

MIT
