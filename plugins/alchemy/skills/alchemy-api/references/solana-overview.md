---
id: references/solana-overview.md
name: solana
description: Solana-specific APIs including standard JSON-RPC, Digital Asset Standard (DAS) for NFTs and Bubblegum compressed NFTs, the Photon indexer for ZK Compression accounts/tokens, WebSocket PubSub subscriptions, and wallet integration. Use when building Solana applications that need RPC access, NFT/asset queries, ZK-compressed state, real-time event streams, or Solana wallet tooling. For high-throughput streaming, see the yellowstone-grpc skill.
tags: []
related: []
updated: 2026-04-30
metadata:
  author: alchemyplatform
  version: "1.0"
---
# Solana APIs

## Summary
Solana-specific APIs and streaming endpoints, including DAS (Digital Asset Standard) and Yellowstone gRPC.

## References (Recommended Order)
1. [solana-rpc.md](solana-rpc.md) - Standard Solana JSON-RPC usage patterns.
2. [solana-das-api.md](solana-das-api.md) - DAS endpoints for NFTs and Bubblegum compressed NFTs.
3. [solana-wallets.md](solana-wallets.md) - Solana wallet integration notes.
4. [node-websocket-subscriptions.md](node-websocket-subscriptions.md) - Solana PubSub subscriptions (`accountSubscribe`, `programSubscribe`, `logsSubscribe`, `signatureSubscribe`, `slotSubscribe`, `rootSubscribe`).

## Photon API (ZK Compression)
Alchemy serves the [Photon](https://github.com/helius-labs/photon) indexer on the standard Solana endpoints. Photon exposes [ZK Compression](https://www.zkcompression.com/) state — compressed accounts, compressed token balances, and compression-related signatures — via JSON-RPC. Methods include `getCompressedAccount`, `getCompressedAccountsByOwner`, `getCompressedTokenAccountsByOwner`, `getCompressedTokenBalancesByOwnerV2`, `getValidityProof` / `getValidityProofV2`, and the `getCompressionSignaturesFor*` family. Use the same Solana endpoint URL (`https://solana-mainnet.g.alchemy.com/v2/<API_KEY>` or `solana-devnet`).

Note: ZK Compression (Photon) and Bubblegum compressed NFTs (DAS) are **different** primitives. Photon is for arbitrary ZK-compressed accounts/tokens; DAS `getAsset` / `getAssetProof` is for Bubblegum cNFTs.

## Cross-References
- `yellowstone-grpc` skill for high-throughput streaming (accounts/transactions/blocks).
- `data-apis` skill for EVM assets.
- `wallets` skill → `wallets-solana-notes.md` for high-level wallet guidance.
- `alchemy-cli` skill for live agent work via the local CLI (preferred local fallback). Includes Solana RPC and DAS commands.
- `alchemy-mcp` skill for live agent work via the hosted MCP server (when CLI is not installed). Exposes 50+ Solana RPC tools and DAS tools.
- `agentic-gateway` skill for app code without an API key (x402 or MPP). Solana wallets pay USDC via SVM x402.

## Official Docs
- [Solana API Quickstart](https://www.alchemy.com/docs/reference/solana-api-quickstart)
- [DAS APIs for Solana](https://www.alchemy.com/docs/reference/alchemy-das-apis-for-solana)
- [Photon APIs for Solana ZK Compression](https://www.alchemy.com/docs/reference/alchemy-photon-apis-for-solana)
- [Solana Subscription API Endpoints](https://www.alchemy.com/docs/reference/solana-subscription-api-endpoints)
