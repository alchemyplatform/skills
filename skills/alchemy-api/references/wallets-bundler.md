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
updated: 2026-08-24
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
- BSO (Bundler Sponsored Operations) — the recommended default for new gas sponsorship integrations; see `wallets-gas-manager.md`.

## v5 client SDK
Bundler APIs are available in `@alchemy/aa-infra` (v5; replaces `@account-kit/infra` from the v4 era). For higher-level abstractions, use `@alchemy/wallet-apis` (Wallet APIs v5).

```ts
import { createAlchemySmartAccountClient } from "@alchemy/aa-infra";
// pair with @alchemy/smart-accounts for the account factory
```

## BSO — recommended default (out of beta)
BSO went GA in 2026-07 and moved into the Gas Manager "Sponsor gas" dropdown. It is now the recommended default for new gas sponsorship integrations. Two paths:

- **Wallet APIs (recommended)**: `createSmartWalletClient` + `paymaster: { policyId }`. High-level, handles the triple-zero convention automatically.
- **Low-level (direct bundler)**: viem `createBundlerClient` + zeroed gas fields (see below) + `x-alchemy-policy-id` header.

Bundler Sponsored Operations via the low-level path require **all three** gas fields zeroed (not just two):

```ts
await bundlerClient.sendUserOperation({
  calls: [{ to, data, value }],
  maxFeePerGas: 0n,
  maxPriorityFeePerGas: 0n,
  preVerificationGas: 0n,
});
```

The `x-alchemy-policy-id` header goes on the bundler transport via `fetchOptions.headers` — no `createPaymasterClient` needed, BSO does not use a paymaster contract. If you forget `preVerificationGas`, the bundler returns `precheck failed: preVerificationGas too low`.

## Bundler RPC error data fields (Rundler wire format)
Structured error data on bundler responses uses **camelCase** field names in Rundler's wire format. When keying an integration off these fields (e.g., automated retries or user messaging), use these exact names:

| Error code | Meaning | Notable `data` fields |
|---|---|---|
| `-32500` | Sender signature check failed / OOG | `innerReason`, `revertData` |
| `-32501` | Paymaster signature check failed | — (no `data` payload) |
| `-32503` | UserOp expired or not-yet-valid | `validUntil`, `validAfter` |
| `-32504` | Entity threw during validation | `Entity` shape: single-key `{ <entityType>: <address> }` (NOT `{ entity, address }`) |
| `-32505` | Unsupported signature aggregator | — (no `data` payload) |
| `-32508` | Paymaster balance too low | `PaymasterBalanceTooLow` — reported as of 2026-08 |
| `-32521` | UserOp execution reverted | `revertData` |
| `-32602` | Invalid gas parameters | `currentMaxPriorityFee`, `currentMaxFee` |

Legacy `snake_case` field names (`inner_reason`, `revert_data`, `valid_until`, `valid_after`, `current_max_priority_fee`, `current_max_fee`) are **not** what Rundler serializes. Agents keying on snake_case will silently fail to branch.

## Integration Notes
- Ensure correct chain configuration and EntryPoint version.
- Monitor bundler latency and failures.
- Use `eth_getUserOperationByHash` to poll UserOp status; if still null after timeout, drop and replace with higher fees. If you're on the **Wallet APIs** path, retries auto-bump gas per network minimum (see `wallets-wallet-apis.md`); on the direct bundler path you compute the bump yourself.
- For `Replacement Underpriced` errors, increase both `maxFeePerGas` and `maxPriorityFeePerGas` by at least 10%.

## Related Files
- `wallets-smart-wallets.md`
- `wallets-gas-manager.md`
- `wallets-wallet-apis.md`

## Official Docs
- [Bundler Overview](https://www.alchemy.com/docs/wallets/low-level-infra/bundler/overview)
- [Bundler Sponsored Operations](https://www.alchemy.com/docs/wallets/transactions/sponsor-gas/bundler-sponsored-operations)
- [Bundler FAQs](https://www.alchemy.com/docs/wallets/reference/bundler-faqs)
- [Bundler RPC Errors](https://www.alchemy.com/docs/wallets/reference/bundler-rpc-errors) — canonical `data` field enumeration.
