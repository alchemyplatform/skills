---
id: references/wallets-bundler.md
name: 'Bundler'
description: 'A bundler aggregates and submits account abstraction user operations. Use this when integrating smart accounts (Wallet APIs). Supports EntryPoint v0.6, v0.7, and v0.8.'
tags:
  - alchemy
  - wallets
related:
  - wallets-smart-wallets.md
  - wallets-gas-manager.md
  - wallets-wallet-apis.md
updated: 2026-05-27
---
# Bundler

## Summary
A bundler aggregates and submits account abstraction user operations. Use this when integrating smart accounts via Alchemy Wallet APIs. Powered by Rundler, Alchemy's production-grade ERC-4337 bundler.

## Supported EntryPoint Versions
- **v0.6** — original ERC-4337 EntryPoint
- **v0.7** — updated gas model and validation logic
- **v0.8** — latest version with additional optimizations

When configuring the bundler, ensure you target the correct EntryPoint version for your smart account implementation.

## Primary Use Cases
- AA transaction submission via standard ERC-4337 JSON-RPC endpoints.
- UserOperation lifecycle handling (submit, track, drop-and-replace).
- BSO (Bundler Sponsored Operations) for gas-free smart-account flows — see `wallets-gas-manager.md`.

## v5 client SDK
Bundler APIs are available in `@alchemy/aa-infra` (v5; replaces `@account-kit/infra` from the v4 era). For higher-level abstractions, use `@alchemy/wallet-apis` (Wallet APIs v5).

```ts
import { createAlchemySmartAccountClient } from "@alchemy/aa-infra";
// pair with @alchemy/smart-accounts for the account factory
```

## BSO request shape
Bundler Sponsored Operations require **all three** gas fields zeroed (not just two):

```ts
await bundlerClient.sendUserOperation({
  calls: [{ to, data, value }],
  maxFeePerGas: 0n,
  maxPriorityFeePerGas: 0n,
  preVerificationGas: 0n,
});
```

The `x-alchemy-policy-id` header goes on the bundler transport via `fetchOptions.headers` — no `createPaymasterClient` needed, BSO does not use a paymaster contract. If you forget `preVerificationGas`, the bundler returns `precheck failed: preVerificationGas too low`.

## Integration Notes
- Ensure correct chain configuration and EntryPoint version.
- Monitor bundler latency and failures.
- Use `eth_getUserOperationByHash` to poll UserOp status; if still null after timeout, drop and replace with higher fees.
- For `Replacement Underpriced` errors, increase both `maxFeePerGas` and `maxPriorityFeePerGas` by at least 10%.

## Related Files
- `wallets-smart-wallets.md`
- `wallets-gas-manager.md`
- `wallets-wallet-apis.md`

## Official Docs
- [Bundler Overview](https://www.alchemy.com/docs/wallets/low-level-infra/bundler/overview)
- [Bundler Sponsored Operations](https://www.alchemy.com/docs/wallets/bundler-api/bundler-sponsored-operations)
- [Bundler FAQs](https://www.alchemy.com/docs/wallets/reference/bundler-faqs)
