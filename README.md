# Alchemy Skill

Root Agent Skill for integrating [Alchemy](https://www.alchemy.com/) APIs into AI-powered applications.

## Repository Structure

This repo uses a single-skill layout:

- `SKILL.md`: root guide and entrypoint
- `references/`: all reference docs in a flat folder with category-prefixed filenames
- `spec/`: Agent Skills spec reference
- `template/`: supporting template files

## Getting Started

1. Get an API key at [dashboard.alchemy.com](https://dashboard.alchemy.com/).
2. Start with [`SKILL.md`](SKILL.md) for base URLs, auth patterns, and quickstart snippets.
3. Use the category overview files in `references/` to dive into specific topics.

## Endpoint Selector (Top Tasks)

| You need | Use this | Skill / File |
| --- | --- | --- |
| EVM read/write | JSON-RPC `eth_*` | `node-apis` → `references/node-json-rpc.md` |
| Realtime events | `eth_subscribe` | `node-apis` → `references/node-websocket-subscriptions.md` |
| Token balances | `alchemy_getTokenBalances` | `data-apis` → `references/data-token-api.md` |
| Token metadata | `alchemy_getTokenMetadata` | `data-apis` → `references/data-token-api.md` |
| Transfers history | `alchemy_getAssetTransfers` | `data-apis` → `references/data-transfers-api.md` |
| NFT ownership | `GET /getNFTsForOwner` | `data-apis` → `references/data-nft-api.md` |
| NFT metadata | `GET /getNFTMetadata` | `data-apis` → `references/data-nft-api.md` |
| Prices (spot) | `GET /tokens/by-symbol` | `data-apis` → `references/data-prices-api.md` |
| Prices (historical) | `POST /tokens/historical` | `data-apis` → `references/data-prices-api.md` |
| Portfolio (multi-chain) | `POST /assets/*/by-address` | `data-apis` → `references/data-portfolio-apis.md` |
| Simulate tx | `alchemy_simulateAssetChanges` | `data-apis` → `references/data-simulation-api.md` |
| Create webhook | `POST /create-webhook` | `webhooks` → `references/webhooks-details.md` |
| Solana NFT data | `getAssetsByOwner` (DAS) | `solana` → `references/solana-das-api.md` |

## Reference Categories

Category order mirrors `SKILL.md`:

| Category | Overview File | Scope |
| --- | --- | --- |
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
