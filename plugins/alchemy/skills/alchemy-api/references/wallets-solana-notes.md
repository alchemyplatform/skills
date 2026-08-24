---
id: references/wallets-solana-notes.md
name: 'Solana Wallet Notes'
description: 'Solana wallet integration via the v5 Wallet APIs (wallet_prepareCalls / wallet_sendPreparedCalls / sendCalls). Covers signer adapters, sponsorship, and routing to Solana RPC and DAS.'
tags:
  - alchemy
  - wallets
  - solana
related:
  - wallets-wallet-apis.md
  - solana-rpc.md
  - solana-das-api.md
updated: 2026-06-03
---
# Solana Wallet Notes

## Summary
Wallet APIs **v5** (GA) supports Solana alongside EVM via the same `wallet_prepareCalls` / `wallet_sendPreparedCalls` JSON-RPC methods and the same `@alchemy/wallet-apis` client SDK. The request schema is `anyOf`-shaped on a single endpoint; Solana vs EVM is selected by the chain ID and the per-call payload shape. Account Kit v4 has no Solana support — v5 Wallet APIs is the only path.

## Request shape (Solana)
- `chainId` is CAIP-2: `"solana:mainnet"` or `"solana:devnet"`.
- `from` is a base58-encoded Solana address (32–44 chars). Validation rejects `0x`-style hex.
- **Field name is `calls`, not `instructions`**, matching the EVM payload. Each call item is shaped for Solana:
  ```json
  {
    "programId": "11111111111111111111111111111111",
    "accounts": [
      { "pubkey": "<base58>", "isSigner": true,  "isWritable": true },
      { "pubkey": "<base58>", "isSigner": false, "isWritable": true }
    ],
    "data": "0x<hex-encoded instruction data>"
  }
  ```
  Instruction `data` is `0x`-prefixed hex (NOT base58). Each account entry carries `pubkey` / `isSigner` / `isWritable`.
- The prepared response carries `type: "solana-transaction-v0"` (Solana v0 versioned transaction). EVM responses use `type: "user-operation-v0.7"` and similar.
- `result.signatureRequest.data` is the full v0 transaction wire-format bytes, hex-encoded. Sign these bytes with the user's Ed25519 key and pass a base58 signature back.
- `wallet_sendPreparedCalls` Solana signature field: `signature: { type: "ed25519", data: "<base58 sig>" }`.
- Call receipts return a base58 `signature` (Solana tx sig) instead of an EVM `transactionHash`.

## SDK surface (`@alchemy/wallet-apis` v5)
- `createSmartWalletClient({ signer, transport: alchemyWalletTransport({ apiKey }), chain: "solana:devnet" | "solana:mainnet", paymaster: { policyId } })` returns a client with `sendCalls`, `prepareCalls`, `waitForCallsStatus`.
- The client exposes `client.solanaAccount` (the base58 address) for use as `from`/`to` in call helpers.
- `sendCalls({ calls: [...] })` uses the same `calls` field as EVM; the shape of each item is Solana-style (programId / accounts / data) when `chain` is a CAIP-2 Solana ID.

## Solana signer adapters (`@alchemy/wallet-apis/solana`)
Pass the result as `signer` to `createSmartWalletClient`. The Solana signer interface is `{ address, signTransaction({ transaction: Uint8Array }): Promise<{ signedTransaction: Uint8Array }> }`.

| Source | Adapter |
|---|---|
| Privy Solana wallet (`@privy-io/react-auth/solana`) | **No adapter** — pass the wallet directly. |
| `useWallet()` from `@solana/wallet-adapter-react`, or injected providers like `window.phantom.solana` | `fromWalletAdapter({ publicKey, signTransaction })` |
| `@solana/kit` `KeyPairSigner` / `TransactionPartialSigner` | `fromKitSigner(kitSigner)` |
| Raw Ed25519 keypair / KMS (provides `address` + `signMessage(bytes) -> 64-byte sig`) | `fromKeypair({ address, signMessage })` |
| Low-level Wallet Standard wallet | `fromWalletStandard(wallet, account)` |

`fromKeypair`'s `signMessage` MUST return a real 64-byte Ed25519 signature; the server rejects all-zero or placeholder bytes.

## Sponsorship
Solana sponsorship is GA, not gated behind support. Create a Solana gas sponsorship policy in the Gas Manager dashboard, then:

- **Client-level**: `paymaster: { policyId: SOLANA_POLICY_ID }` on `createSmartWalletClient`.
- **Per-request override**: `capabilities.paymaster.policyId` on `sendCalls` / `prepareCalls` takes priority over the client-level policy.
- **Rent sponsorship**:
  - Top-level `SystemProgram.createAccount` and Associated Token Program `Create` / `CreateIdempotent` instructions are sponsored automatically — no extra param.
  - For accounts created inside a CPI, opt in with `capabilities.paymaster.prefundRent: true`. CPI rent prefunding is allowlisted — contact support@alchemy.com. Policies used with `prefundRent` must set `maxSpendPerTxnUsd`, and the transaction payer must be the user's wallet (same account the CPI debits).

## Low-level alternative
If you are not using the Wallet APIs client (e.g., you already have a serialized Solana transaction), use the lower-level `alchemy_requestFeePayer` flow to add a sponsored fee payer.

## Routing
- Wallet onboarding + smart-account flows on Solana → `wallets-wallet-apis.md` (v5).
- Solana JSON-RPC (account data, transactions, signatures, block info) → `solana-rpc.md`.
- Solana NFT / compressed-asset / DAS data → `solana-das-api.md`.

## Gotchas
- Don't mix CAIP-2 IDs and decimal `chainId` — Solana payloads MUST use `solana:<cluster>`.
- Field is `calls` (not `instructions`). The shape of each call item is what selects EVM vs Solana semantics.
- Instruction `data` is hex, NOT base58. Signature payload is base58, NOT hex.
- The same Wallet API method namespace (`wallet_*`) handles both EVM and Solana; ensure your transport/headers (especially `x-alchemy-policy-id` for sponsorship, when applicable) target the network you're calling.
- Account Kit v4 has no Solana path. Don't try to bolt Solana onto a v4 setup.

## Related Files
- `wallets-wallet-apis.md`
- `solana-rpc.md`
- `solana-das-api.md`

## Official Docs
- [Send Solana transactions](https://www.alchemy.com/docs/wallets/transactions/send-transactions)
- [Solana signer adapters](https://www.alchemy.com/docs/wallets/solana/signers)
- [Solana sponsorship](https://www.alchemy.com/docs/wallets/transactions/sponsor-gas/solana)
- [`wallet_prepareCalls`](https://www.alchemy.com/docs/wallets/api-reference/smart-wallets/wallet-api-endpoints/wallet-prepare-calls)
- [`wallet_sendPreparedCalls`](https://www.alchemy.com/docs/wallets/api-reference/smart-wallets/wallet-api-endpoints/wallet-send-prepared-calls)
- [Solana API Quickstart](https://www.alchemy.com/docs/reference/solana-api-quickstart)
