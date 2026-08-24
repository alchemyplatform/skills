---
id: references/solana-websocket-subscriptions.md
name: 'Solana WebSocket Subscriptions (PubSub)'
description: 'Solana PubSub WebSocket subscriptions for real-time account, program, log, signature, slot, and root updates. Native `*Subscribe` / `*Unsubscribe` methods, not `eth_subscribe`.'
tags:
  - alchemy
  - solana
  - json-rpc
  - websocket
related:
  - solana-rpc.md
  - node-websocket-subscriptions.md
  - solana-grpc-overview.md
updated: 2026-05-06
---
# Solana WebSocket Subscriptions (PubSub)

Solana exposes a PubSub WebSocket API alongside its standard HTTP JSON-RPC. Each subscription opens a long-lived stream that pushes notifications when on-chain state changes.

**Base URL**: `wss://solana-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY` (or `solana-devnet`)

Solana uses native `*Subscribe` / `*Unsubscribe` methods. There is **no** `eth_subscribe` on Solana. Each `*Subscribe` call returns a numeric `subscription` id; pass it to the matching `*Unsubscribe` to cancel the stream.

## When to Use Which Stream
| Goal | Use |
|---|---|
| Watch one account's lamports / data change | `accountSubscribe` |
| Watch all accounts owned by a program | `programSubscribe` (always filter — see below) |
| Watch transaction logs that mention a pubkey | `logsSubscribe` |
| Wait for a single transaction to reach commitment | `signatureSubscribe` (auto-cancels after firing) |
| Track validator slot progress | `slotSubscribe` |
| Track the validator's new root slot | `rootSubscribe` |
| High-throughput indexing across the cluster | switch to Yellowstone gRPC — see `solana-grpc-overview.md` |

## Methods

### `accountSubscribe` / `accountUnsubscribe`
Notifies on lamports or data change for one account.

- Params: `pubkey` (base-58), optional `{ commitment, encoding }`.
- `encoding`: `base58`, `base64`, `base64+zstd`, or `jsonParsed`.

### `programSubscribe` / `programUnsubscribe`
Notifies on changes to any account owned by a program. **Always pair with `dataSize` and/or `memcmp` filters** — unfiltered streams produce extreme volumes and bandwidth bills.

- Params: program `pubkey`, optional `{ commitment, encoding, filters }`.
- Filters mirror `getProgramAccounts`: `[{ dataSize: <bytes> }, { memcmp: { offset, bytes } }]`.

### `logsSubscribe` / `logsUnsubscribe`
Notifies when committed transaction logs match a filter.

- Filter: `"all"` (excluding vote txs), `"allWithVotes"`, or `{ "mentions": ["<pubkey>"] }` with exactly one pubkey.
- Notification payload includes `signature`, `err`, `logs[]`.

### `signatureSubscribe` / `signatureUnsubscribe`
Notifies once when a specific transaction signature reaches the requested commitment, then auto-cancels.

- Params: `signature` (base-58), optional `{ commitment, enableReceivedNotification }`.
- `enableReceivedNotification: true` adds an additional `received` notification before commitment.

### `slotSubscribe` / `slotUnsubscribe`
Notifies when the validator processes a new slot. Notification payload: `{ slot, parent, root }`. No params.

### `rootSubscribe` / `rootUnsubscribe`
Notifies when the validator sets a new root slot. Notification payload: a single slot number. No params.

## Request Format
PubSub uses JSON-RPC 2.0 over a persistent WebSocket. After a subscribe request succeeds, the server returns the numeric subscription id in the response, then pushes notifications under a method-specific name (`accountNotification`, `logsNotification`, etc.) keyed by that id.

```json
// subscribe
{"jsonrpc":"2.0","id":1,"method":"accountSubscribe","params":["<pubkey>", {"encoding":"jsonParsed","commitment":"finalized"}]}

// subscribe response
{"jsonrpc":"2.0","result":23784,"id":1}

// notification (later, asynchronously)
{"jsonrpc":"2.0","method":"accountNotification","params":{"result":{...},"subscription":23784}}

// unsubscribe
{"jsonrpc":"2.0","id":2,"method":"accountUnsubscribe","params":[23784]}
```

## SDK Mapping (`@solana/web3.js`)
The `Connection` class wraps subscribe/unsubscribe in typed listeners. Pass `wsEndpoint` alongside the HTTP URL.

| RPC method | `Connection` helper | Removal helper |
|---|---|---|
| `accountSubscribe` | `onAccountChange` | `removeAccountChangeListener` |
| `programSubscribe` | `onProgramAccountChange` | `removeProgramAccountChangeListener` |
| `logsSubscribe` | `onLogs` | `removeOnLogsListener` |
| `signatureSubscribe` | `onSignature` | `removeSignatureListener` (auto-removed on fire) |
| `slotSubscribe` | `onSlotChange` | `removeSlotChangeListener` |
| `rootSubscribe` | `onRootChange` | `removeRootChangeListener` |

## Pricing
Solana PubSub subscriptions are billed by **bandwidth** — pro-rated to the byte (1 byte = 0.0002 CU). Broad streams (especially unfiltered `programSubscribe`) can scale costs quickly. Recommended controls:

- Always filter `programSubscribe` with `dataSize` / `memcmp`.
- Use a more specific stream over a broader one (`accountSubscribe` vs `programSubscribe`, `signatureSubscribe` vs `logsSubscribe`).
- Keep payloads small: `encoding: "base64"` over `jsonParsed` when you don't need parsed data.
- Set [usage limits](https://www.alchemy.com/docs/how-to-set-usage-limits-and-alerts-for-your-account) before deploying high-volume streams.

## Notes
- Subscriptions are stateful. Handle reconnects and resubscribe after reconnect; you may receive duplicate notifications immediately after reconnect.
- Commitment defaults to `finalized` on every subscribe method that accepts it.
- For high-throughput cluster-wide indexing, prefer Yellowstone gRPC (`solana-grpc-overview.md`) — PubSub is a per-connection RPC, not a firehose.

## Official Docs
- [Solana Subscription API Endpoints](https://www.alchemy.com/docs/reference/solana-subscription-api-endpoints)
- [Compute Unit Costs — Solana WebSocket Subscriptions](https://www.alchemy.com/docs/reference/compute-unit-costs#solana-websocket-subscriptions)
- [Solana Foundation PubSub reference](https://solana.com/docs/rpc/websocket)
