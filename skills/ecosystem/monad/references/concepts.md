# Monad Architecture Concepts

Monad is Ethereum-compatible (no Solidity changes required), but its architecture introduces behaviors that affect how apps must be built.

## Quick reference

| Concept | Summary |
| --- | --- |
| **Async execution** | Consensus and execution are decoupled. Block leaders propose blocks **before** executing them. State view is delayed by ~3 blocks. **Newly funded accounts need ~1.2s** before they can send transactions. |
| **Parallel execution** | Optimistic concurrency. Transactions run in parallel where state-disjoint; conflicts re-run. Identical results to Ethereum. **No contract changes needed.** |
| **Block states** | Proposed → Voted → Finalized → Verified. Maps to JSON-RPC tags `pending`/`latest` (proposed), `safe` (voted), `finalized` (committed). |
| **Reserve balance** | EOAs maintain a **10 MON floor**. Low-balance accounts are limited to ~1 tx per 1.2s. Emptying transactions revert if they'd take the balance below 10 MON. |
| **EIP-7702** | EOAs delegate to contracts for smart-wallet features (session keys, gas sponsorship, batch txs). 10 MON floor still applies. **No `CREATE`/`CREATE2` in delegated context.** |
| **Real-time data** | Three sources: (1) Geth-compatible WebSocket — the default for most apps; (2) Monad extended WebSocket — richer event payloads; (3) Execution Events SDK — speculative pre-finality EVM traces. |

## Block state → JSON-RPC tag mapping

When making `eth_call` / `eth_getBlock*` / `eth_getBalance` and friends:

| RPC tag | Monad state | Use when |
| --- | --- | --- |
| `pending` | Proposed (not yet voted) | Optimistic UI — show what's likely about to land |
| `latest` | Proposed → executed (~3 block delay between proposal and visible state) | Default for most reads |
| `safe` | Voted by validators | Defensive UI for high-value displays |
| `finalized` | Finalized (committed by consensus) | Settlement guarantees, bridges, irrevocable actions |

## Async execution gotchas

The most common failure mode: **fund an account, then immediately try to send a tx — the tx fails**. State visibility lags execution by ~3 blocks (~1.2s). Wait one finalization cycle before relying on a balance change.

Apps that show "funded ✓" → "send tx" UX should either:
1. Poll `eth_getBalance` against `latest` until the balance reaches the expected value (1-2 seconds), OR
2. Insert an explicit 1.5–2s delay between funding and dependent transactions.

## EIP-7702 (delegated EOAs)

EIP-7702 is natively supported on Monad mainnet from day one. An EOA signs a delegation to a smart contract — subsequent transactions from that EOA execute the contract's logic. This unlocks smart-wallet features (session keys, paymasters, batch calls) without migrating to a separate smart account address.

Caveats:
- The 10 MON reserve balance floor still applies to delegated EOAs.
- `CREATE` and `CREATE2` are **not** available inside delegated execution. Use canonical factories (CreateX, Create2Deployer — see [`addresses.md`](./addresses.md)) for deterministic deployments.

For Alchemy's ERC-4337 smart-account path (which is independent of EIP-7702 and uses a separate smart-account address), use `alchemy-cli` or `alchemy-api`.

## Full upstream coverage

Each concept has a detailed reference in upstream monskills:

- async-execution.md — newly funded accounts, funding delays, 3-block state delay
- parallel-execution.md — what changes (nothing) for existing Solidity
- block-states.md — choosing between `latest` / `safe` / `finalized` tags
- reserve-balance.md — 10 MON floor, emptying transactions, low-balance throttling
- eip-7702.md — smart wallet delegation, session keys, gas sponsorship caveats
- realtime-data.md — Geth WS vs Monad extended WS vs Execution Events SDK
- execution-events.md — BLOCK_START / QC / FINALIZED lifecycle events

Browse them via the `concepts/` skill in [`therealharpaljadeja/monskills`](https://github.com/therealharpaljadeja/monskills/tree/main/concepts).
