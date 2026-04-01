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
updated: 2026-04-01
---
# Wallet APIs

## Summary
High-level wallet APIs enable programmatic wallet operations such as signing, transaction preparation, or account management. This guide stays minimal and focuses on integration awareness.

## Primary Use Cases
- Server-side transaction preparation.
- Delegated signing or session-based flows.

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
- `wallets-account-kit.md`
- `operational-auth-and-keys.md`

## Official Docs
- [Account Kit Wallet Client](https://www.alchemy.com/docs/wallets/reference/account-kit/wallet-client)
- [Wallet API Errors](https://www.alchemy.com/docs/wallets/troubleshooting/wallet-apis-errors)
