---
id: references/node-utility-api.md
name: 'Utility API'
description: 'Convenience RPC methods that reduce round trips for common tasks like bulk transaction receipt retrieval.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
related:
  - node-json-rpc.md
  - operational-rate-limits-and-compute-units.md
updated: 2026-02-05
---
# Utility API

## Summary
Convenience RPC methods that reduce round trips for common tasks like bulk transaction receipt retrieval.

## Primary Use Cases
- Fetch all transaction receipts for a block in a single call.
- Reduce multi-call patterns that would otherwise use `eth_getTransactionReceipt` in a loop.

## When To Use / Not Use
- Use when you need receipts for all transactions in a block.
- Avoid if you only need a single receipt.

## Auth & Setup
- Same JSON-RPC endpoint as standard node API.

## Endpoints / Methods
- `alchemy_getTransactionReceipts` (bulk receipts for a block)

## Spec-Derived Parameters
`alchemy_getTransactionReceipts` accepts one of:
- `blockNumber` (hex string)
- `blockHash` (32-byte hex string)

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_getTransactionReceipts","params":[{"blockNumber":"latest"}]}'
```

## Error Handling
- Same JSON-RPC error handling as standard requests.

## Performance / Limits / Compute Units
- Bulk responses can be large; watch response sizes.

## Gotchas & Edge Cases
- Some networks may not support this method.

## Testing / Mocking
- Use a known block with small tx count.

## Related Files
- `node-json-rpc.md`
- `operational-rate-limits-and-compute-units.md`

## Official Docs
- [alchemy_getTransactionReceipts](https://www.alchemy.com/docs/reference/alchemy-gettransactionreceipts)
