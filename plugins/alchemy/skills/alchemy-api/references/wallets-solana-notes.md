---
id: references/wallets-solana-notes.md
name: 'Solana Wallet Notes'
description: 'Solana wallet integration via the v5 Wallet APIs (wallet_prepareCalls / wallet_sendPreparedCalls). Also covers Solana RPC and DAS routing.'
tags:
  - alchemy
  - wallets
  - solana
related:
  - wallets-wallet-apis.md
  - solana-rpc.md
  - solana-das-api.md
updated: 2026-05-27
---
# Solana Wallet Notes

## Summary
Wallet APIs **v5** (GA) supports Solana alongside EVM via the same `wallet_prepareCalls` and `wallet_sendPreparedCalls` methods. The request schema is `anyOf`-shaped — same RPC endpoint, different fields based on whether you target EVM or Solana. Account Kit v4 has no Solana support; v5 Wallet APIs is the path.

## Request shape (Solana)
- `chainId` is CAIP-2: `"solana:mainnet"` or `"solana:devnet"`.
- `from` is a base58-encoded Solana address (32–44 chars). Validation rejects `0x`-style hex.
- Send `instructions` (an array of raw Solana instructions: `{ programId, accounts, data }`) instead of EVM's `calls`.
- Prepared response carries `type: "solana-transaction-v0"` (Solana v0 versioned transaction). EVM responses use `type: "user-operation-v0.7"` and similar.
- Sign the returned tx with the user's Solana keypair, then submit it back through `wallet_sendPreparedCalls`.

## Routing
- Wallet onboarding + smart-account flows on Solana → `wallets-wallet-apis.md` (v5).
- Solana JSON-RPC (account data, transactions, signatures, block info) → `solana-rpc.md`.
- Solana NFT / compressed-asset / DAS data → `solana-das-api.md`.

## Gotchas
- Don't mix CAIP-2 IDs and decimal `chainId` — Solana payloads MUST use `solana:<cluster>`.
- The same Wallet API method namespace (`wallet_*`) handles both EVM and Solana; ensure your transport/headers (especially `x-alchemy-policy-id` for sponsorship) are correct for the network you're targeting.
- Sponsored gas on Solana is gated differently than EVM — confirm with Alchemy support if you need sponsored Solana flows.

## Related Files
- `wallets-wallet-apis.md`
- `solana-rpc.md`
- `solana-das-api.md`

## Official Docs
- [Wallet APIs Solana support](https://www.alchemy.com/docs/wallets/api/wallet-api-endpoints/wallet-prepare-calls)
- [Solana API Quickstart](https://www.alchemy.com/docs/reference/solana-api-quickstart)
