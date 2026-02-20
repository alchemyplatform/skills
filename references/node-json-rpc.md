---
id: references/node-json-rpc.md
name: 'JSON-RPC (EVM)'
description: 'Use Alchemy''s EVM JSON-RPC endpoints for standard blockchain reads and writes (e.g., `eth_call`, `eth_getLogs`, `eth_sendRawTransaction`). This is the baseline for any EVM integration.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
  - json-rpc
related:
  - node-enhanced-apis.md
  - data-transfers-api.md
  - operational-rate-limits-and-compute-units.md
updated: 2026-02-05
---
# JSON-RPC (EVM)

## Summary
Use Alchemy's EVM JSON-RPC endpoints for standard blockchain reads and writes (e.g., `eth_call`, `eth_getLogs`, `eth_sendRawTransaction`). This is the baseline for any EVM integration.

## Primary Use Cases
- Read on-chain state (balances, contract storage, events).
- Submit raw transactions.
- Build indexing or analytics pipelines.

## When To Use / Not Use
- Use when you need raw, canonical blockchain data.
- Prefer Data APIs for wallet analytics or historical transfer history to avoid log scanning.

## Auth & Setup
- Base URL (HTTPS): `https://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`
- Base URL (WSS): `wss://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`
- Network examples: `eth-mainnet`, `polygon-mainnet`, `arb-mainnet`, `opt-mainnet`, `base-mainnet`.

## Endpoints / Methods
- Any standard Ethereum JSON-RPC method supported by the target chain.
- Common reads: `eth_blockNumber`, `eth_getBalance`, `eth_call`, `eth_getLogs`.
- Writes: `eth_sendRawTransaction`.

## Parameters
- Block identifiers: `latest`, `finalized`, `safe`, or hex block numbers.
- Quantities are hex strings (e.g., `0x10`).

## Pagination / Filtering
- Use `eth_getLogs` with block ranges. For large ranges, chunk requests and respect limits.

## Example Requests
```bash
curl -s https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

```ts
const url = `https://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`;
const body = {
  jsonrpc: "2.0",
  id: 1,
  method: "eth_getBalance",
  params: ["0x00000000219ab540356cbb839cbe05303d7705fa", "latest"],
};

const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});
const json = await res.json();
```

## Example Responses
- Standard JSON-RPC response with `{ jsonrpc, id, result }` or `{ error }`.

## Error Handling
- Handle HTTP `429` (rate limit) with backoff.
- Handle JSON-RPC errors in the response body.

## Performance / Limits / Compute Units
- Batch compatible JSON-RPC requests where supported.
- Prefer Data APIs for high-volume historical queries.
- Avoid tight polling loops; if you must poll, use seconds-level intervals with backoff and jitter.

## Gotchas & Edge Cases
- Some networks may not support every JSON-RPC method.
- `eth_getLogs` over large ranges can be expensive.

## Testing / Mocking
- Use a fixed block number to make tests deterministic.
- Use local fixtures for JSON-RPC responses.

## Agentic Gateway (x402)
This API is available through Alchemy's Agentic Gateway without an API key.
- Base URL: `https://x402.alchemy.com/{chainNetwork}/v2`
- Auth: `Authorization: SIWE <token>` header (see [gateway-authentication](gateway-authentication.md))
- Payment: x402 USDC micropayment (see [gateway-payment](gateway-payment.md))
- All standard and enhanced JSON-RPC methods are supported via the gateway.

## Related Files
- `node-enhanced-apis.md`
- `data-transfers-api.md`
- `operational-rate-limits-and-compute-units.md`
- `gateway-overview.md`
- `gateway-reference.md`

## Official Docs
- [How to Read Data with JSON-RPC](https://www.alchemy.com/docs/how-to-read-data-with-json-rpc)
- [Chain APIs Overview](https://www.alchemy.com/docs/reference/chain-apis-overview)
