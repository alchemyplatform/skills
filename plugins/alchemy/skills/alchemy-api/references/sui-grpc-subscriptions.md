---
id: references/sui-grpc-subscriptions.md
name: 'Sui gRPC Subscriptions'
description: 'SubscriptionService for streaming real-time Sui checkpoint data via server-streaming gRPC.'
tags:
  - alchemy
  - sui
  - grpc
related:
  - sui-grpc-overview.md
updated: 2026-09-02
---
# Sui gRPC — Subscriptions (SubscriptionService)

## Summary
Stream real-time Sui activity from the current tip via server-streaming gRPC. Three subscribe methods cover checkpoints, transactions, and events.

## Methods

| gRPC Method | JSON-RPC Equivalent | Description |
| --- | --- | --- |
| `SubscribeCheckpoints` | No equivalent | Stream new checkpoints as they are finalized |
| `SubscribeTransactions` | `suix_subscribeTransaction` | Stream new transactions as they are executed |
| `SubscribeEvents` | `suix_subscribeEvent` | Stream new Move events as they are emitted |

## Key Patterns
- All three are long-lived server-streaming RPCs — the connection stays open and delivers items from the current tip in real time.
- Use `read_mask` to control which item fields are included on each streamed message.
- The response envelope on each subscribe includes a `cursor` for resuming the stream after disconnection — persist the last-seen cursor per client and pass it back on reconnect.
- Handle reconnection and backpressure in your client. Subscribe RPCs do not automatically survive network partitions.
- These are the tip-following counterparts to the `List*` server-streaming methods in `sui-grpc-objects-and-ledger.md`, which cover ranged historical queries.

## Filtering

- `SubscribeTransactions`: pass a `TransactionFilter` (from-address, to-address, package/module/function, input/changed-object filters) to narrow the stream. Without a filter, every mainnet transaction is delivered.
- `SubscribeEvents`: pass an `EventFilter` (package, module, event type, sender, transaction) to scope to specific Move event streams.

Both filters mirror the shapes in `sui.rpc.v2.filter` on `MystenLabs/sui-apis`.

## Service Path
`sui.rpc.v2.SubscriptionService`

## Related Files
- `sui-grpc-overview.md`

## Official Docs
- [Subscriptions](https://www.alchemy.com/docs/reference/sui-grpc-subscriptions)
