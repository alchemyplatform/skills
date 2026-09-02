---
id: references/sui-grpc-objects-and-ledger.md
name: 'Sui gRPC Objects and Ledger'
description: 'LedgerService methods for querying objects, transactions, checkpoints, and epochs on Sui via gRPC.'
tags:
  - alchemy
  - sui
  - grpc
related:
  - sui-grpc-overview.md
  - sui-grpc-transactions.md
updated: 2026-09-02
---
# Sui gRPC — Objects and Ledger (LedgerService)

## Summary
The `LedgerService` provides methods for querying objects, transactions, checkpoints, epochs, and chain metadata.

## Methods

| gRPC Method | JSON-RPC Equivalent | Description |
| --- | --- | --- |
| `GetObject` | `sui_getObject` | Fetch a single object by ID (optionally at a version) |
| `BatchGetObjects` | `sui_multiGetObjects` | Fetch multiple objects in one request |
| `GetTransaction` | `sui_getTransactionBlock` | Fetch a transaction by digest |
| `BatchGetTransactions` | `sui_multiGetTransactionBlocks` | Fetch multiple transactions |
| `GetCheckpoint` | `sui_getCheckpoint` | Fetch a checkpoint by sequence number or digest |
| `GetEpoch` | No equivalent | Fetch epoch metadata |
| `GetServiceInfo` | `sui_getChainIdentifier` | Chain metadata, current checkpoint height, epoch |
| `ListCheckpoints` | `sui_getLatestCheckpointSequenceNumber` + iteration | Server-streaming range of checkpoints |
| `ListTransactions` | `sui_queryTransactionBlocks` | Server-streaming range of transactions |
| `ListEvents` | `sui_queryEvents` | Server-streaming range of events |

### `List*` server-streaming methods

The three `List*` methods (`ListCheckpoints`, `ListTransactions`, `ListEvents`) are server-streaming RPCs that return an ordered range of results as a stream of individual messages. They replace the historical polling patterns for querying checkpoint / transaction / event history.

Common shape (all three):

| Field | Type | Description |
| --- | --- | --- |
| `read_mask` | `google.protobuf.FieldMask` | Subset of response fields to include. Recommended for bandwidth. |
| `query_options` | `QueryOptions` | Result ordering (`ASCENDING` / `DESCENDING`) and page size. |
| `watermark` | `Watermark` | Starting position (checkpoint sequence, tx digest, or event id). |
| `query_end` | `QueryEnd` | Optional bounding position — omit for open-ended streams up to head. |

Notes:

- Same `sui.rpc.v2.LedgerService` service path.
- Each streamed message is one item (a `Checkpoint`, `Transaction`, or `Event`) — process incrementally and hold state in the client if you need windowed aggregates.
- Cursor-safe: pass the last item's identifier as the next call's `watermark` to resume after a disconnect.
- These are the historical/query counterparts to the `SubscribeTransactions` / `SubscribeEvents` methods in `sui-grpc-subscriptions.md`, which stream new activity from the current tip.

## Key Patterns
- Use `read_mask` on all methods to request only needed fields (reduces response size).
- Use `BatchGetObjects` / `BatchGetTransactions` for multi-item lookups instead of individual calls.
- `GetServiceInfo` requires no fields — use it as a health check or to get current chain state.
- For range queries over history, use `ListCheckpoints` / `ListTransactions` / `ListEvents` — these are server-streaming and cursor-bounded via `QueryOptions` + `Watermark` + `QueryEnd`, so they scale to unbounded ranges without JSON-RPC's per-page overhead.

## Service Path
`sui.rpc.v2.LedgerService`

## Related Files
- `sui-grpc-overview.md`
- `sui-grpc-transactions.md`

## Official Docs
- [Objects and Ledger](https://www.alchemy.com/docs/reference/sui-grpc-objects-and-ledger)
