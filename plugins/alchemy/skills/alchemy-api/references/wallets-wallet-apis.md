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
updated: 2026-05-27
---
# Wallet APIs

## Summary
High-level wallet APIs (Wallet APIs **v5**, GA) enable programmatic wallet operations: signing, transaction preparation, account management, EIP-7702 delegation/undelegation, and (new in v5) Solana transactions. This is the recommended stack for new builds; Account Kit v4 stays for signer-only flows. See `wallets-account-kit.md` for v4 vs v5 routing.

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
`wallet_prepareCalls` and `wallet_sendPreparedCalls` accept Solana payloads alongside EVM via an `anyOf` schema:

- `chainId` uses CAIP-2 form: `"solana:mainnet"` (mainnet) or `"solana:devnet"` (devnet).
- `from` is a base58-encoded Solana address (32–44 chars), not a `0x` hex string.
- The prepared transaction is returned with `type: "solana-transaction-v0"` (Solana v0 versioned transaction). EVM responses still use `type: "user-operation-v0.7"` etc.
- Sign the returned tx with the user's Solana keypair, then submit it back through `wallet_sendPreparedCalls`.
- Request shape: a single positional object with `calls` (EVM) **or** `instructions` (Solana, an array of raw Solana instructions); the call's `to`/`from`/encoding fields and the response shape are gated on whether you sent EVM or Solana inputs.

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

## Integration Notes
- Prefer client-side signing for user security.
- Use server-side APIs only with strong access controls.

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
