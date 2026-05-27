---
id: references/wallets-gas-manager.md
name: 'Gas Manager'
description: 'Gas Manager (paymaster) enables gas sponsorship and cost control for smart wallet flows, including ERC-20 token gas payments and BSO (Bundler Sponsorship) policies.'
tags:
  - alchemy
  - wallets
related:
  - wallets-smart-wallets.md
  - wallets-bundler.md
  - wallets-wallet-apis.md
updated: 2026-05-27
---
# Gas Manager

## Summary
Gas Manager (paymaster) enables gas sponsorship and cost control for smart wallet flows.

## Primary Use Cases
- Gasless user onboarding.
- Sponsoring specific methods or contracts.
- ERC-20 token gas payments (pay gas with any supported token).
- BSO (Bundler Sponsorship) policies for EIP-7702 undelegation and other native-gas-free flows.

## Live Source of Truth — Dashboard
The list of supported ERC-20 tokens and networks rotates more often than docs can keep up. Treat the **Tokens & Networks** panel of the [Gas Manager Dashboard](https://dashboard.alchemy.com/gas-manager) as the authoritative source for what's available right now. The published docs FAQ enumerates the same tokens but lags by a release or two.

## ERC-20 Gas Payment Supported Networks
Native USDC/USDT/wETH coverage spans Ethereum mainnet/Sepolia, Arbitrum (Mainnet, Sepolia, **Nova**), Optimism, Base, Polygon, BNB, Celo, Worldchain, Stable mainnet, Stable testnet, and Monad testnet. Any token supported by Alchemy's [Token Prices By Address API](https://www.alchemy.com/docs/data/prices-api/prices-api-endpoints/prices-api-endpoints/get-token-prices-by-address) can be enabled via the [Admin API `create-policy`](https://www.alchemy.com/docs/wallets/api/gas-manager-admin-api/admin-api-endpoints/create-policy) endpoint even if it's not in the Dashboard preset list.

For a token or network that isn't supported, contact `wallets@alchemy.com`.

## BSO Chain Support
Bundler Sponsored Operations (BSOs) are a bundler feature and are supported on **every chain that has both bundler and gas sponsorship support**, with the exception of **MegaETH** (coming soon). Always confirm against the live [Wallet APIs supported chains](https://www.alchemy.com/docs/wallets/supported-chains) matrix before launch.

### BSO request shape (triple-zero gas fields)
To opt into BSO on a user operation, set ALL THREE of `maxFeePerGas`, `maxPriorityFeePerGas`, **and** `preVerificationGas` to `0n` (client SDK) or `"0x0"` (raw JSON-RPC). The bundler treats triple-zero as the signal to cover gas under the BSO policy. The `x-alchemy-policy-id` header goes on the **bundler** transport (no `createPaymasterClient` — BSO does not use a paymaster contract).

```ts
// v5 client SDK
await bundlerClient.sendUserOperation({
  calls: [{ to: "0x000000000000000000000000000000000000dEaD", data: "0x" }],
  maxFeePerGas: 0n,
  maxPriorityFeePerGas: 0n,
  preVerificationGas: 0n,
});
```

If you forget `preVerificationGas`, the bundler returns `precheck failed: preVerificationGas too low` (`-32000`).

## ERC-20 Token Gas Payment — Revert Risk
When using **post-operation** mode for ERC-20 gas payments:
- If a token approval is batched with your calls and any call reverts, the approval is also reverted. The paymaster cannot collect the token payment, and **you (the policy owner) pay the gas cost** without receiving token compensation.
- If a sufficient allowance already exists (e.g., from threshold mode), the paymaster collects payment even if the batch reverts.
- **Use post-operation** when operations are unlikely to revert (works with all ERC-20 tokens).
- **Use pre-operation** when operations may revert — the token transfer happens before execution so the paymaster is always compensated.

## Integration Notes
- Define strict sponsorship policies.
- Monitor for abuse and enforce caps.

## Related Files
- `wallets-smart-wallets.md`
- `wallets-bundler.md`
- `wallets-wallet-apis.md`

## Official Docs
- [Gas Manager Admin API](https://www.alchemy.com/docs/wallets/low-level-infra/gas-manager/policy-management/api-endpoints)
- [Pay Gas With Any Token](https://www.alchemy.com/docs/wallets/transactions/pay-gas-with-any-token)
- [Bundler Sponsored Operations](https://www.alchemy.com/docs/wallets/bundler-api/bundler-sponsored-operations)
- [Gas Manager FAQs](https://www.alchemy.com/docs/wallets/gas-manager-admin-api/gas-manager-faqs)
