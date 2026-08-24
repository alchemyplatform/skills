---
id: references/solana-wallets.md
name: 'Solana Wallet Integration'
description: 'High-level guidance for integrating Solana wallets and signing transactions, including the v5 Wallet APIs path.'
tags:
  - alchemy
  - solana
related:
  - wallets-solana-notes.md
  - wallets-wallet-apis.md
  - solana-rpc.md
  - ecosystem-solana-web3js.md
updated: 2026-06-03
---
# Solana Wallet Integration

## Summary
High-level guidance for integrating Solana wallets and signing transactions. The recommended path for new builds is Wallet APIs **v5** (`@alchemy/wallet-apis`), which supports sponsored Solana transactions, embedded-wallet signers, and the same `sendCalls` / `prepareCalls` surface used on EVM. Direct `@solana/web3.js` integration is still fine when you don't need smart accounts or sponsorship.

## Primary Use Cases
- Wallet connect flows for dApps.
- Program interactions and signing.
- Sponsored Solana transactions via the v5 Wallet APIs client.

## v5 Wallet APIs path (recommended for sponsored / smart-wallet flows)
```ts
import {
  alchemyWalletTransport,
  createSmartWalletClient,
} from "@alchemy/wallet-apis";
import { fromKitSigner } from "@alchemy/wallet-apis/solana";

const client = createSmartWalletClient({
  signer: fromKitSigner(kitSigner),
  transport: alchemyWalletTransport({ apiKey: ALCHEMY_API_KEY }),
  chain: "solana:devnet",                        // or "solana:mainnet"
  paymaster: { policyId: SOLANA_POLICY_ID },     // optional
});

await client.sendCalls({ calls: [/* Solana call items */] });
```

Pick the signer adapter that matches your source (from `@alchemy/wallet-apis/solana`):

- `@solana/kit` `KeyPairSigner` / `TransactionPartialSigner` → `fromKitSigner`
- `@solana/wallet-adapter-react` `useWallet()` or injected `window.phantom.solana` → `fromWalletAdapter`
- Raw Ed25519 keypair / KMS → `fromKeypair`
- Wallet Standard wallet → `fromWalletStandard`
- Privy `@privy-io/react-auth/solana` wallet → pass directly, no adapter

See `wallets-solana-notes.md` for the full Solana request shape, sponsorship rules, and gotchas.

## Direct `@solana/web3.js` path (no smart account)
- Use `Transaction` or `VersionedTransaction` based on program requirements.
- Always specify the recent blockhash.
- Handle signature confirmation with appropriate `commitment`.

## Gotchas
- Solana payloads on Wallet APIs use CAIP-2 chain IDs (`solana:mainnet` / `solana:devnet`), not decimal `chainId`.
- Instruction `data` in Wallet APIs `calls[].data` is `0x`-hex, NOT base58.
- The Wallet APIs SDK accepts the field name `calls` on Solana — not `instructions`.

## Related Files
- `wallets-solana-notes.md`
- `wallets-wallet-apis.md`
- `solana-rpc.md`
- `ecosystem-solana-web3js.md`

## Official Docs
- [Send Solana transactions](https://www.alchemy.com/docs/wallets/transactions/send-transactions)
- [Solana signer adapters](https://www.alchemy.com/docs/wallets/solana/signers)
- [Solana API Quickstart](https://www.alchemy.com/docs/reference/solana-api-quickstart)
