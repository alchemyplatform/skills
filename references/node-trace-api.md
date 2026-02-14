---
id: references/node-trace-api.md
name: 'Trace API'
description: 'Trace APIs expose internal call data and state changes for transactions and blocks. Useful for analytics and auditing.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
related:
  - node-debug-api.md
  - data-transfers-api.md
updated: 2026-02-05
---
# Trace API

## Summary
Trace APIs expose internal call data and state changes for transactions and blocks. Useful for analytics and auditing.

## Primary Use Cases
- Build internal-call graphs for transactions.
- Identify protocol-level fund flows beyond top-level transfers.

## When To Use / Not Use
- Use for offline analytics or deep inspection.
- Avoid in latency-sensitive user flows.

## Auth & Setup
- Same JSON-RPC endpoint as standard node API.

## Endpoints / Methods (Spec-Derived)
Common trace methods:
- `trace_transaction`
- `trace_block`
- `trace_call`
- `trace_filter`
- `trace_get`
- `trace_rawTransaction`
- `trace_replayBlockTransactions`
- `trace_replayTransaction`

## Spec-Derived Parameters
`trace_transaction`:
- `transactionHash` (32-byte hex)

`trace_filter`:
- `filter.fromBlock`, `filter.toBlock` (hex)
- `filter.fromAddress[]`, `filter.toAddress[]`
- `filter.after` (string cursor)
- `filter.count` (integer)

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"trace_transaction","params":["0x<tx_hash>"]}'
```

## Error Handling
- Methods may be unavailable on some networks.

## Performance / Limits / Compute Units
- Trace data is large; batch and cache.

## Gotchas & Edge Cases
- Different trace endpoints return different field schemas.

## Testing / Mocking
- Use fixed hashes and store trace fixtures.

## Related Files
- `node-debug-api.md`
- `data-transfers-api.md`

## Official Docs
- [trace_transaction](https://www.alchemy.com/docs/reference/what-is-trace_transaction)
