---
id: references/node-websocket-subscriptions.md
name: 'WebSocket Subscriptions'
description: 'Use WebSockets for real-time blockchain events without polling. Best for pending transactions, new blocks, and logs.'
tags:
  - alchemy
  - node-apis
  - evm
  - rpc
related:
  - node-json-rpc.md
  - webhooks-details.md
updated: 2026-02-05
---
# WebSocket Subscriptions

## Summary
Use WebSockets for real-time blockchain events without polling. Best for pending transactions, new blocks, and logs.

## Primary Use Cases
- Watch new blocks or pending transactions.
- Subscribe to contract events (logs).
- Build real-time dashboards and alerting pipelines.

## When To Use / Not Use
- Use when you need near real-time updates.
- Avoid if you only need periodic data; HTTP may be simpler and cheaper.

## Auth & Setup
- Base URL: `wss://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Endpoints / Methods
- `eth_subscribe` with topics: `newHeads`, `logs`, `newPendingTransactions`.
- `eth_unsubscribe` to stop a subscription.

## Parameters
- `logs` subscriptions accept address and topics filter.

## Example Requests
```ts
import WebSocket from "ws";

const ws = new WebSocket(
  `wss://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`
);

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      jsonrpc: "2.0",
      id: 1,
      method: "eth_subscribe",
      params: ["newHeads"],
    })
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(data.toString());
  // msg.params.result contains block header
  console.log(msg);
});
```

## Error Handling
- Handle reconnects and resubscribe after reconnect.
- Treat malformed messages as recoverable errors.

## Performance / Limits / Compute Units
- Subscriptions are stateful; monitor connection health.
- Use tight filters for `logs` subscriptions.
- If WebSockets are unavailable, fall back to HTTP polling with a coarse interval and backoff.

## Gotchas & Edge Cases
- You may receive duplicate events on reconnect; de-dupe by block hash or log index.
- Pending tx subscriptions are high-volume.

## Testing / Mocking
- Use a testnet with known event emitters.
- Record/replay WebSocket frames for integration tests.

## Related Files
- `node-json-rpc.md`
- `webhooks-details.md`

## Official Docs
- [Subscription API Overview](https://www.alchemy.com/docs/reference/subscription-api)
- [eth_subscribe Reference](https://www.alchemy.com/docs/chains/ethereum/ethereum-api-endpoints/eth-subscribe)
