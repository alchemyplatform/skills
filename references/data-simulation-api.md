---
id: references/data-simulation-api.md
name: 'Simulation API'
description: 'Simulate transactions before submitting them on-chain. Use this for safety checks and user previews.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - node-debug-api.md
  - recipes-simulate-transaction.md
updated: 2026-02-05
---
# Simulation API

## Summary
Simulate transactions before submitting them on-chain. Use this for safety checks and user previews.

## Primary Use Cases
- Preflight checks before a wallet signs a transaction.
- Detect unexpected asset transfers.
- Show user-friendly previews of transaction effects.

## When To Use / Not Use
- Use for user-facing safety checks or internal risk analysis.
- Avoid in ultra-low-latency paths unless you can cache results.

## Auth & Setup
- JSON-RPC endpoint: `https://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Endpoints / Methods
- `alchemy_simulateAssetChanges`
- `alchemy_simulateExecution`
- `alchemy_simulateAssetChangesBundle`
- `alchemy_simulateExecutionBundle`

## Spec-Derived Parameters
### `alchemy_simulateAssetChanges`
- Param: `transaction` object (requires `to`)
- Common fields: `transaction.from`, `transaction.to`, `transaction.value`, `transaction.data`, `transaction.gas`, `transaction.gasPrice`, `transaction.maxFeePerGas`, `transaction.maxPriorityFeePerGas` (hex strings)

### `alchemy_simulateExecution`
- Param order: `format`, `transaction`, `blockTag`
- `format` enum: `NESTED`, `FLAT`
- `blockTag` enum: `latest`, `safe`, `finalized`, `earliest`
- `transaction` schema same as above (requires `to`)

### Bundle Methods
- `alchemy_simulateAssetChangesBundle` / `alchemy_simulateExecutionBundle`
- Params: `transactions` array (ordered, sequential effects)

## Example Requests
### Simulate Asset Changes
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_simulateAssetChanges","params":[{"from":"0x00000000219ab540356cbb839cbe05303d7705fa","to":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","data":"0x"}]}'
```

### Simulate Execution (Call Trace Focus)
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"alchemy_simulateExecution","params":["FLAT", {"from":"0x00000000219ab540356cbb839cbe05303d7705fa","to":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","data":"0x"}, "latest"]}'
```

## Error Handling
- Simulation can return revert reasons; surface these to users.
- Results can change as state changes; re-run if needed.

## Performance / Limits / Compute Units
- Simulation is more expensive than standard reads.

## Gotchas & Edge Cases
- Simulation does not guarantee real execution outcome if state changes.
- Bundles execute sequentially; earlier txs affect later txs.

## Testing / Mocking
- Use deterministic calldata and block numbers.

## Agentic Gateway (x402)
This API is available through Alchemy's Agentic Gateway without an API key.
- Base URL: `https://x402.alchemy.com/{chainNetwork}/v2` (Simulation methods are JSON-RPC)
- Auth: `Authorization: SIWE <token>` header (see [gateway-authentication](gateway-authentication.md))
- Payment: x402 USDC micropayment (see [gateway-payment](gateway-payment.md))

## Related Files
- `node-debug-api.md`
- `recipes-simulate-transaction.md`
- `gateway-overview.md`
- `gateway-reference.md`

## Official Docs
- [simulateAssetChanges](https://www.alchemy.com/docs/reference/simulateassetchanges-sdk)
