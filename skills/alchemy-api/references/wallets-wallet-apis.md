---
id: references/wallets-wallet-apis.md
name: 'Wallet APIs'
description: 'High-level wallet APIs enable programmatic wallet operations such as signing, transaction preparation, or account management. This guide stays minimal and focuses on integration awareness.'
tags:
  - alchemy
  - wallets
related:
  - wallets-account-kit.md
  - operational-auth-and-keys.md
updated: 2026-08-24
---
# Wallet APIs

## Summary
High-level wallet APIs (Wallet APIs **v5**, GA) enable programmatic wallet operations: signing, transaction preparation, account management, EIP-7702 delegation/undelegation, and (new in v5) Solana transactions. This is the recommended stack for new builds; Account Kit v4 is deprecated in-nav (only Export Private Key remains as a documented v4 page). See `wallets-account-kit.md` for v4 vs v5 routing.

## Primary Use Cases
- Server-side transaction preparation (EVM and Solana).
- Delegated signing or session-based flows (including existing session keys — see [Legacy session keys with Wallet APIs](https://www.alchemy.com/docs/wallets/smart-wallets/session-keys/legacy-session-keys) for migrating pre-existing session-key setups).
- EIP-7702 account delegation and undelegation.
- Cross-chain swaps and ERC-20 gas payments via `wallet_prepareCalls` capabilities.

## v5 SDK package surface
The v5 stack lives under the `@alchemy/*` scope. Common imports:

| Package | Use |
|---|---|
| `@alchemy/wallet-apis` | High-level client SDK (`createWalletClient`, `prepareCalls`, `sendPreparedCalls`). |
| `@alchemy/smart-accounts` | MAv2 / Light Account / smart contract account factories. |
| `@alchemy/aa-infra` | Low-level bundler client; replaces `@account-kit/infra`. |
| `viem` | Underlying chain primitives (`parseEther`, `parseUnits`, `bigint` helpers). |

## Solana support
`wallet_prepareCalls` and `wallet_sendPreparedCalls` accept Solana payloads alongside EVM via an `anyOf` schema. See `wallets-solana-notes.md` for full request/SDK details. Summary:

- `chainId` uses CAIP-2 form: `"solana:mainnet"` or `"solana:devnet"`.
- `from` is a base58 Solana address; `0x` hex is rejected.
- **Field name is `calls` for both EVM and Solana** — the shape of each call item is what differs. Solana items are `{ programId, accounts: [{pubkey, isSigner, isWritable}], data }` with `data` as `0x`-hex instruction bytes.
- Prepared response carries `type: "solana-transaction-v0"`. `signatureRequest.data` is the v0 tx wire bytes (hex).
- Sign with Ed25519 and submit a base58 signature via `signature: { type: "ed25519", data: "<base58>" }`.
- SDK: `createSmartWalletClient({ chain: "solana:devnet", signer, paymaster })` exposes `sendCalls({ calls: [...] })` and `client.solanaAccount`. Solana signer adapters live at `@alchemy/wallet-apis/solana` (`fromKitSigner`, `fromWalletAdapter`, `fromKeypair`, `fromWalletStandard`); Privy's `@privy-io/react-auth/solana` wallets work without an adapter.
- Solana sponsorship is GA — point `paymaster: { policyId }` at a Solana Gas Manager policy. Per-request override via `capabilities.paymaster.policyId`. Top-level System Program / Associated Token Program account creation gets rent sponsored automatically; CPI-rent prefunding (`capabilities.paymaster.prefundRent: true`) is allowlisted (contact support).

## Encoding: bigint vs hex
- **Client SDK (`@alchemy/wallet-apis` v5)**: takes `bigint` for amount fields like `fromAmount`, `minimumToAmount`, `value`. Use viem helpers: `parseEther("1.5")`, `parseUnits("100", 6)`.
- **Raw JSON-RPC**: amounts and gas fields are `0x`-prefixed hex strings (e.g., `"value": "0x16345785d8a0000"`).

If you see `expected bigint, got string`, you're passing a hex string to the SDK — convert via `BigInt(hex)` or use viem helpers.

## EIP-7702 Undelegation
Undelegation removes smart contract delegation from an EIP-7702 account by delegating to the zero address (`0x0000...0000`), restoring it to a plain EOA. Key details:
- Gas is sponsored through a **BSO (Bundler Sponsorship) policy** — the account does not need native tokens.
- Requires **enterprise plan** — sponsored undelegation is gated to enterprise customers.
- Available via both the client SDK (`@alchemy/wallet-apis`) and REST API.
- For advanced control, use `wallet_prepareCalls` + `wallet_sendPreparedCalls` to inspect and sign the authorization separately.

## Chain selection
- For EVM, pass a viem chain object to `createSmartWalletClient({ chain })`. `viem/chains` exports include named entries like `hyperEvm` (HyperEVM mainnet) and `hyperliquidEvmTestnet`. Pass `hyperEvm` directly — no custom chain config needed.
- For Solana, pass the CAIP-2 string `"solana:mainnet"` or `"solana:devnet"`.
- Confirm the target network is enabled on your Alchemy app and that any gas policy is linked to the same app.

## Non-7702 mode (request-then-send)
Some chains (e.g. HyperEVM today) don't support EIP-7702 with Wallet APIs. On those chains, request a smart account address first, then pass that address as `account` on subsequent `sendCalls`:

```ts
const { address } = await client.requestAccount({
  creationHint: { accountType: "sma-b" }, // MAv2 (the v5 default)
});

await client.sendCalls({
  account: address,
  calls: [{ to: target, value: 0n, data: "0x..." }],
});
```

`accountType` values are self-serve as of 2026-05: `sma-b` (default, MAv2), `7702`, `la-v2`, `la-v2-multi-owner`, `ma-v1-multi-owner`, `la-v1.1.0`, `la-v1.0.2`, `la-v1.0.1`. Caching the returned address per signer avoids extra `requestAccount` calls on every send.

### `wallet_requestAccount` and `createAdditional`
By default, `wallet_requestAccount` returns the signer's **oldest existing** account, even if you pass a different `accountType` via `creationHint`. To force creation of a NEW account of the requested type when the signer already has an account, pass `createAdditional: true`:

```ts
const { address } = await client.requestAccount({
  creationHint: { accountType: "7702" },
  createAdditional: true,
});
```

Without `createAdditional: true`, the returned account may not match the requested type. If your flow depends on a specific account type, either always pass `createAdditional: true` or check `accountType` in the response before proceeding.

## Retries and automatic gas bumping
When a call is stuck and you re-prepare it, the Wallet APIs server **automatically bumps gas fees** — the client no longer needs to compute a higher fee. The bump rule is **10% higher OR the current network minimum, whichever is higher**, applied per-field to `maxFeePerGas` and `maxPriorityFeePerGas`.

- Do NOT override fees on retry unless you want to force a specific level.
- This auto-bump is **Wallet APIs specific**. The direct bundler-API path (`eth_sendUserOperation`) still requires you to compute and submit higher fees yourself; see `wallets-bundler.md`.

## Tracking sends: call ID, not transaction hash
v5 Wallet API sends return a **call ID** (`callId`), not a `transactionHash`. Track and poll by call ID (`wallet_getCallsStatus`); the transaction hash appears in the receipt only after the bundler includes the UserOp on-chain.

```ts
const { callId } = await client.sendCalls({ calls: [...] });
const status = await client.getCallsStatus({ id: callId });
// status.receipts[0].transactionHash appears once mined
```

Do not build integrations that expect `receipts[0].transactionHash` to be available immediately after `sendCalls` returns.

## Integration Notes
- Prefer client-side signing for user security.
- Use server-side APIs only with strong access controls.
- Privy + Wallet APIs: convert a Privy embedded EVM wallet with `toViemAccount({ wallet })` from `@privy-io/react-auth` and pass the returned `LocalAccount` as `signer`. For Solana, use the wallet returned by `useWallets` from `@privy-io/react-auth/solana` directly (no adapter).

## Common Errors & Troubleshooting
When users report Wallet API failures, check for these common error patterns:

| Error | Code | Cause & Fix |
|---|---|---|
| `replacement underpriced` | -32602 | Sending a call from the same sender before a pending call confirms. Wait for confirmation or increase fees. |
| `execution reverted` | -32521 | Calls revert on-chain. Verify correct method, contract address, chain, and ABI encoding. |
| `AA23 reverted` | -32500 | Sender signature validation reverted or OOG. Check signature and gas limits. |
| `AA25 invalid account nonce` | -32500 | Nonce reuse. Use a fresh nonce. |
| `Policy ID(s) not found` | -32600 | Gas sponsorship policy misconfigured. Ensure API key matches the policy's app, policy is active, and network is allowed. |
| `invalid account signature` | -32507 | Incorrect signature. Verify signing matches the Wallet API quickstart flow. |
| `precheck failed: sender balance` | -32000 | Sender balance too low for gas. Add funds or include a Gas Sponsorship Policy ID. |
| `maxFeePerGas too low` | -32000 | Base fee changed between prepare and send. Re-prepare immediately before sending. |
| `maxPriorityFeePerGas too low` | -32000 | Priority fee changed. Re-prepare before sending. |
| `preVerificationGas too low` | -32000 | Gas estimate stale. Re-prepare the call. |
| `total gas limit too high` | -32000 | Combined gas exceeds max. Split into multiple operations. |
| `EIP-7702 not enabled` | -32602 | Chain doesn't support EIP-7702. Use `wallet_requestAccount` for a smart contract account instead. |
| `EIP-7702 authorization signature is invalid` | -32000 | Malformed or wrong-key signature for EIP-7702 authorization. |
| `EIP-7702 nonce mismatch` | -32000 | Another tx sent between prepare and send. Avoid concurrent txs from the same account. |

**Key pattern:** Most `precheck failed` errors (-32000) resolve by re-preparing the call immediately before sending — minimize delay between `wallet_prepareCalls` and `wallet_sendPreparedCalls`.

## Related Files
- `wallets-account-kit.md` (v4 vs v5 routing)
- `wallets-solana-notes.md`
- `wallets-gas-manager.md`
- `operational-auth-and-keys.md`

## Official Docs
- [Account Kit Wallet Client](https://www.alchemy.com/docs/wallets/reference/account-kit/wallet-client)
- [Wallet API Errors](https://www.alchemy.com/docs/wallets/troubleshooting/wallet-apis-errors)
- [Legacy session keys with Wallet APIs](https://www.alchemy.com/docs/wallets/smart-wallets/session-keys/legacy-session-keys) — use existing session keys with Wallet APIs.
- [v5 migration guide](https://www.alchemy.com/docs/wallets/resources/migration-v5)
- [Swap Tokens (v5)](https://www.alchemy.com/docs/wallets/transactions/swap-tokens) — bigint amount conventions for the client SDK.
- [Send Solana transactions](https://www.alchemy.com/docs/wallets/transactions/send-transactions) — SDK and raw-API flows for Solana.
- [Solana signer adapters](https://www.alchemy.com/docs/wallets/solana/signers) — `fromKitSigner` / `fromWalletAdapter` / `fromKeypair` / `fromWalletStandard`.
- [Solana sponsorship](https://www.alchemy.com/docs/wallets/transactions/sponsor-gas/solana) — client-level + per-request paymaster, rent sponsorship rules.
- [Hyperliquid Transactions Quickstart](https://www.alchemy.com/docs/wallets/recipes/hyperliquid-wallets) — Privy + v5 + non-7702 `requestAccount` pattern on HyperEVM.
