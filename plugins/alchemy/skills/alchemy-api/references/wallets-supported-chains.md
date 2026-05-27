---
id: references/wallets-supported-chains.md
name: 'Wallet Supported Chains'
description: 'Wallet tooling supports a different chain set than raw RPC. Confirm chain support against the live dashboard before launch.'
tags:
  - alchemy
  - wallets
related:
  - wallets-gas-manager.md
  - operational-supported-networks.md
updated: 2026-05-27
---
# Wallet Supported Chains

## Summary
Wallet tooling (Wallet APIs v5, bundler, paymaster, Gas Manager) supports a subset / superset of chains compared to raw RPC. Always confirm against live sources before launch.

## Sources of truth
- [Wallet APIs supported chains matrix](https://www.alchemy.com/docs/wallets/supported-chains) — chain-by-chain capability axes (bundler, paymaster, ERC-20 gas payments, BSOs).
- **Tokens & Networks** panel of the [Gas Manager Dashboard](https://dashboard.alchemy.com/gas-manager) — authoritative live list of which networks have ERC-20 gas payments enabled and which tokens are preset.
- For tokens that aren't preset, the Admin API can enable any token supported by Alchemy's [Token Prices By Address API](https://www.alchemy.com/docs/data/prices-api/prices-api-endpoints/prices-api-endpoints/get-token-prices-by-address).

## Recently added wallet chains (capability axis)
- **Arbitrum Nova** — re-added to the ERC-20 paymaster supported list.
- **BNB Smart Chain** — ERC-20 gas payments (USDC, DAI).
- **Celo** — ERC-20 gas payments (USDC, USDT, CELO).
- **Stable mainnet / testnet** — ERC-20 gas payments (USDT0).

These additions don't replace the docs/dashboard matrix — check there before assuming a chain × capability cell is live.

## Guidance
- Test on testnet before enabling production.
- A chain having bundler ≠ that chain having gas sponsorship. Cross-reference both columns.

## Related Files
- `wallets-gas-manager.md`
- `operational-supported-networks.md`

## Official Docs
- [Wallet APIs supported chains](https://www.alchemy.com/docs/wallets/supported-chains)
- [Supported Networks (RPC)](https://www.alchemy.com/docs/reference/node-supported-chains)
