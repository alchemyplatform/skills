---
id: references/wallets-supported-chains.md
name: 'Wallet Supported Chains'
description: 'Wallet APIs are exposed on a subset of chains, with separate availability per capability (Bundler, Gas Sponsorship, ERC-20 Gas Payments, BSOs). Always confirm chain × capability before launch.'
tags:
  - alchemy
  - wallets
related:
  - operational-supported-networks.md
  - wallets-wallet-apis.md
  - wallets-bundler.md
  - wallets-gas-manager.md
updated: 2026-08-24
---
# Wallet Supported Chains

## Summary
Wallet APIs (formerly "Smart Wallets") run on a subset of chains compared to raw RPC, and support varies *per capability* on each chain. Always confirm the specific capability you need is enabled on the chain you're targeting before launch.

## Capability Axes
The supported-chains table lists each chain × four capability columns:

- **Bundler** — submit ERC-4337 user operations on this chain.
- **Gas Sponsorship** — Gas Manager (paymaster) policies cover user op gas in the chain's native token.
- **ERC-20 Gas Payments** — users pay gas in a stable / project token instead of the native asset.
- **BSOs (Bundler Sponsored Operations)** — sponsored ops without a Gas Manager policy. Currently NOT available on MegaETH or Solana.

Login and signer authentication are chain-agnostic — they work everywhere the SDK runs. The four columns above gate transaction sending and sponsorship.

## Routing Logic
- "Does Wallet APIs work on chain X?" → check the [supported chains table](https://www.alchemy.com/docs/wallets/resources/supported-chains). Don't guess — the table changes weekly.
- "Can I sponsor gas on chain X?" → confirm the Gas Sponsorship column.
- "Can users pay gas in USDC on chain X?" → confirm the ERC-20 Gas Payments column.
- "Why doesn't BSO work on MegaETH / Solana?" → BSOs aren't enabled there yet (Gas Manager + Bundler are).
- Solana is supported for Gas Sponsorship only — no Bundler, no ERC-20 gas, no BSOs.
- Need a chain that isn't listed? Email `wallets@alchemy.com` or use the chain support [request form](https://alchemy.chilipiper.com/router/wallet-services-chain-support-requests).

## Integration Notes
- Use the same Alchemy app across multiple chains; see the [multi-chain setup](https://www.alchemy.com/docs/wallets/recipes/multi-chain-setup) recipe.
- The chain identifier (e.g. `base-mainnet`) is what the SDK uses to construct RPC URLs of the form `https://<chain>.g.alchemy.com/v2/<API_KEY>`.
- Always test on testnet (Sepolia variants, Solana Devnet, etc.) before enabling production.

## Related Files
- `operational-supported-networks.md` — node/RPC-level chain support (broader set).
- `wallets-wallet-apis.md` — the Wallet APIs surface that runs on these chains.
- `wallets-bundler.md`, `wallets-gas-manager.md` — capability-specific details.

## Official Docs
- [Wallet APIs supported chains](https://www.alchemy.com/docs/wallets/resources/supported-chains)
- [Node Supported Chains](https://www.alchemy.com/docs/reference/node-supported-chains)
