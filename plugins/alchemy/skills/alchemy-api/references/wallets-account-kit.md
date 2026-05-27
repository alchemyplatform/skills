---
id: references/wallets-account-kit.md
name: 'Account Kit (v4) and the v5 stack'
description: 'Account Kit v4 is the legacy SDK for signer + smart-account UX (still maintained for signer-only flows). New builds should target the v5 stack: @alchemy/wallet-apis + Privy as signer.'
tags:
  - alchemy
  - wallets
related:
  - wallets-wallet-apis.md
  - wallets-smart-wallets.md
  - wallets-gas-manager.md
  - operational-auth-and-keys.md
updated: 2026-05-27
---
# Account Kit (v4) and the v5 stack

## Summary
"Account Kit" now refers to the **v4** SDK family (`@account-kit/*`, `@aa-sdk/*`). Wallet APIs **v5** is the recommended stack for new builds. The docs site labels the legacy nav section "Account Kit (v4)" to make this explicit.

## v4 vs v5 routing

| Use case | Recommended package(s) | Notes |
|---|---|---|
| New smart-account integration (EVM or Solana) | `@alchemy/wallet-apis@^5` + `@alchemy/smart-accounts@^5` | The v5 stack. Solana support landed via `wallet_prepareCalls` + `wallet_sendPreparedCalls` (CAIP-2 IDs `solana:mainnet`, `solana:devnet`). |
| Low-level bundler client | `@alchemy/aa-infra@^5` | Replaces `@account-kit/infra` from the v4 era. |
| Embedded signer / auth UX (passkey, email, social, EOA) | Account Kit v4 (`@account-kit/react`, `@account-kit/signer`) | v5 has no signer SDK yet. For new builds, Alchemy's recommended pairing is **Wallet APIs v5 + Privy as signer**. |
| Existing Account Kit v4 app | Stay on v4 for now | v4 is still maintained. The `aa-sdk` repo dropped v4 reference docs from `main`; use `aa-sdk@v4.x.x` branch for source. |
| Migration off Account Kit v4 → v5 | See [v5 migration guide](https://www.alchemy.com/docs/wallets/resources/migration-v5) | Plus the v4 banner on each affected page in `/docs/wallets/...`. |

## What changed in v5 (highlights)
- Package surface renamed: `@account-kit/*` and `@aa-sdk/*` → `@alchemy/wallet-apis`, `@alchemy/smart-accounts`, `@alchemy/aa-infra`.
- Solana transactions go through Wallet APIs alongside EVM (anyOf-style param schema; the same `wallet_prepareCalls` / `wallet_sendPreparedCalls` methods).
- BSO (Bundler Sponsored Operations) now requires three zero fields, not two — see `wallets-gas-manager.md` and `wallets-bundler.md`.
- v5 SDKs use viem-style `bigint` for amounts (`fromAmount`, `minimumToAmount`). Raw JSON-RPC still uses `0x`-hex strings.
- Beta notices have been removed across the wallets section — Wallet APIs v5 is GA.

## Primary Use Cases (Account Kit v4)
- Embedded wallet UX in web apps (signer-driven authentication).
- Email/social/passkey wallet creation.
- Signing flows that don't yet have a v5 equivalent.

## Integration Notes
- Pair with `wallets-gas-manager.md` for sponsored transactions.
- For v5 builds, route most "Account Kit"-shaped questions to `wallets-wallet-apis.md`.

## Gotchas & Edge Cases
- `aa-sdk` reference URLs on `main` now 404 — use the `v4.x.x` branch.
- Account Kit v4 has no Solana support; use v5 Wallet APIs for Solana.
- Mixing v4 and v5 packages in the same app is not supported.

## Related Files
- `wallets-wallet-apis.md` (v5 surface)
- `wallets-smart-wallets.md`
- `wallets-gas-manager.md`
- `operational-auth-and-keys.md`

## Official Docs
- [Account Kit Core Reference](https://www.alchemy.com/docs/wallets/reference/account-kit/core)
- [Intro to Account Kit](https://www.alchemy.com/docs/wallets/concepts/intro-to-account-kit)
- [v5 migration guide](https://www.alchemy.com/docs/wallets/resources/migration-v5)
- [aa-sdk v4.x.x branch](https://github.com/alchemyplatform/aa-sdk/tree/v4.x.x) — source of truth for v4 reference symbols.
