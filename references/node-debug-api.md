---
id: references/node-debug-api.md
name: 'Debug API'
description: 'Debug methods provide execution-level traces for a transaction or call. Use these for simulation, gas profiling, and internal call inspection.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
related:
  - node-trace-api.md
  - data-simulation-api.md
updated: 2026-02-05
---
# Debug API

## Summary
Debug methods provide execution-level traces for a transaction or call. Use these for simulation, gas profiling, and internal call inspection.

## Primary Use Cases
- Trace internal contract calls and opcodes.
- Preflight analysis for risky transactions.
- Gas cost attribution.

## When To Use / Not Use
- Use for debugging and simulation.
- Avoid in latency-critical user requests.

## Auth & Setup
- Same JSON-RPC endpoint as standard node API.

## Endpoints / Methods
Common debug methods (verify network support):
- `debug_traceTransaction`
- `debug_traceCall`
- `debug_traceBlockByNumber`
- `debug_traceBlockByHash`

## Spec-Derived Parameters
`debug_traceTransaction`:
- `transactionHash` (32-byte hex)
- `options` (object):
- `tracer` enum: `callTracer`, `prestateTracer`
- `tracerConfig.onlyTopCall` (boolean)
- `timeout` (duration string)

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"debug_traceTransaction","params":["0x<tx_hash>", {"tracer":"callTracer"}]}'
```

## Error Handling
- Some trace calls may time out or be unsupported on certain networks.

## Performance / Limits / Compute Units
- Debug traces are expensive; cache results if you can.

## Gotchas & Edge Cases
- Trace output formats vary by tracer.

## Testing / Mocking
- Use a small, known transaction for deterministic traces.

## Related Files
- `node-trace-api.md`
- `data-simulation-api.md`

## Official Docs
- [debug_traceTransaction](https://www.alchemy.com/docs/reference/debug-tracetransaction)
