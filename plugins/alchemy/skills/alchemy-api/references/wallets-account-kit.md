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
updated: 2026-08-24
---
# Account Kit (v4) and the v5 stack

## Summary
"Account Kit" now refers to the **v4** SDK family (`@account-kit/*`, `@aa-sdk/*`). Wallet APIs **v5** is the recommended stack for new builds. As of 2026-08 the docs nav collapsed the entire v4 section to a single `[Deprecated] Account Kit` entry retaining only `Export private key` — v4 login/UI/whitelabel/MFA/Solana/connectors documentation is gone. Existing v4 apps still function but should be migrated. All other v4 URLs redirect to `/docs/wallets`.

## ⚠️ Alchemy Signer is being sunset

As of **June 16, 2026**, Alchemy Signer APIs no longer support new user signups. Existing users can still log in to be migrated. The `POST /signer/v1/signup` endpoint is deprecated and the `Create Wallet` reference page has been removed. JWT auth (`POST /signer/v1/auth/jwt`) still authenticates existing users, but it no longer creates new wallets or pregenerates wallets.

**For new builds**: use [Privy](https://www.alchemy.com/docs/wallets/third-party/signers/privy) as the signer with [Wallet APIs v5](https://www.alchemy.com/docs/wallets/quickstart). Do NOT scaffold new applications against `@account-kit/signer` or the `/signer/v1/signup` REST endpoint — both will only serve existing user migration after the cutoff.

## ⚠️ Account Kit v4 is nav-deprecated (2026-08)

The v4 nav section was collapsed to only the `Export Private Key` page in 2026-08. Twenty-six v4 MDX pages (login methods, UI components, whitelabel, connectors, MFA, Solana wallets, server wallets, account management, `Programmatic Wallet Creation` recipe) were removed and their URLs redirect to `/docs/wallets`. Consequences for new work:

- Do NOT scaffold new projects using `@account-kit/react`, `@account-kit/signer`, or any `@account-kit/*` package.
- The `Programmatic Wallet Creation` recipe pattern is GONE — do NOT recommend it for new integrations.
- v4 Solana support was already absent; v5 Wallet APIs is the only supported path for Solana.
- v4 whitelabel / MFA / social login / passkey flows should route through Privy + Wallet APIs v5.
- The `Export Private Key` page remains because it's the migration path off v4 for users who want to leave.

## v4 vs v5 routing

| Use case | Recommended package(s) | Notes |
|---|---|---|
| New smart-account integration (EVM or Solana) | `@alchemy/wallet-apis@^5` + `@alchemy/smart-accounts@^5` | The v5 stack. Solana support landed via `wallet_prepareCalls` + `wallet_sendPreparedCalls` (CAIP-2 IDs `solana:mainnet`, `solana:devnet`). |
| Low-level bundler client | `@alchemy/aa-infra@^5` | Replaces `@account-kit/infra` from the v4 era. |
| New embedded signer / auth UX (passkey, email, social, EOA) | **Privy + Wallet APIs v5** | Alchemy Signer signup is sunset; `@account-kit/signer` cannot create new users. Privy is the recommended pairing. |
| Existing Account Kit v4 app (only existing-user logins) | Stay on v4 for now | v4 maintains existing-user login + migration. New-user signup is blocked at the Signer API. Use `aa-sdk@v4.x.x` branch for v4 source. |
| Migration off Account Kit v4 → v5 | See [v5 migration guide](https://www.alchemy.com/docs/wallets/resources/migration-v5) | Plus the v4 banner on each affected page in `/docs/wallets/...`. |

## What changed in v5 (highlights)
- Package surface renamed: `@account-kit/*` and `@aa-sdk/*` → `@alchemy/wallet-apis`, `@alchemy/smart-accounts`, `@alchemy/aa-infra`.
- Solana transactions go through Wallet APIs alongside EVM (anyOf-style param schema; the same `wallet_prepareCalls` / `wallet_sendPreparedCalls` methods).
- BSO (Bundler Sponsored Operations) now requires three zero fields, not two — see `wallets-gas-manager.md` and `wallets-bundler.md`.
- v5 SDKs use viem-style `bigint` for amounts (`fromAmount`, `minimumToAmount`). Raw JSON-RPC still uses `0x`-hex strings.
- Beta notices have been removed across the wallets section — Wallet APIs v5 is GA.

## Primary Use Cases (Account Kit v4)
- Continuing to serve **existing** Account Kit Signer users (login + migration only).
- Signing flows on existing-user accounts that don't yet have a v5 equivalent.
- New-user wallet creation is no longer in scope — see the sunset notice above.

## Integration Notes
- Pair with `wallets-gas-manager.md` for sponsored transactions.
- For v5 builds, route most "Account Kit"-shaped questions to `wallets-wallet-apis.md`.

## Gotchas & Edge Cases
- `aa-sdk` reference URLs on `main` now 404 — use the `v4.x.x` branch.
- Account Kit v4 has no Solana support; use v5 Wallet APIs for Solana.
- Mixing v4 and v5 packages in the same app is not supported.
- Calling `POST /signer/v1/signup` for a new user returns a deprecation response after 2026-06-16. The previous `isSignup: true` response field is also deprecated.

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
