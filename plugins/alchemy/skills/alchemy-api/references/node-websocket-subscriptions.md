---
id: references/node-websocket-subscriptions.md
name: 'WebSocket Subscriptions'
description: 'Use WebSockets for real-time blockchain events without polling. EVM uses eth_subscribe; Solana uses native *Subscribe / *Unsubscribe PubSub methods.'
tags:
  - alchemy
  - node-apis
  - evm
  - solana
  - rpc
related:
  - node-json-rpc.md
  - solana-rpc.md
  - webhooks-details.md
updated: 2026-04-30
---
# WebSocket Subscriptions

Real-time blockchain events via WebSocket. No polling required. EVM chains use `eth_subscribe` / `eth_unsubscribe`; Solana uses native `*Subscribe` / `*Unsubscribe` PubSub methods (see [Solana PubSub Subscriptions](#solana-pubsub-subscriptions)).

**Base URL**: `wss://<network>.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Billing & Scope Guidance
Alchemy bills WebSocket subscriptions on the bandwidth they deliver, so broad streams can scale compute unit usage quickly. Keep subscriptions narrow by default:

- Prefer filtered subscriptions (address, topic, or `alchemy_minedTransactions` filters) over network-wide streams.
- Prefer hash-only payloads when full transaction objects are not required (e.g., `hashesOnly: true` for `alchemy_pendingTransactions` / `alchemy_minedTransactions`).
- Set [usage limits](https://www.alchemy.com/docs/how-to-set-usage-limits-and-alerts-for-your-account) and alerts before deploying high-volume streams.
- Broad subscription streams can generate far more ongoing traffic than equivalent HTTP polling, because the server keeps pushing every matching event until you unsubscribe.

See [Compute Unit Costs — Webhooks and Subscription APIs](https://www.alchemy.com/docs/reference/compute-unit-costs#webhooks-and-subscription-apis) for pricing details.

---

## `eth_subscribe`

Creates a subscription for real-time events.

### Subscription Types

| Type | Description |
|------|-------------|
| `newHeads` | New block headers as they are mined |
| `logs` | Event logs matching a filter |
| `newPendingTransactions` | Pending transaction hashes (high volume) |
| `alchemy_minedTransactions` | Mined transactions matching a filter (Alchemy-specific) |

---

### `newHeads`

Subscribe to new block headers.

#### Request

```json
{ "jsonrpc": "2.0", "id": 1, "method": "eth_subscribe", "params": ["newHeads"] }
```

#### Notification

```json
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x1234...",
    "result": {
      "number": "0x1312D00",
      "hash": "0x...",
      "parentHash": "0x...",
      "timestamp": "0x...",
      "gasLimit": "0x...",
      "gasUsed": "0x...",
      "baseFeePerGas": "0x...",
      "miner": "0x..."
    }
  }
}
```

---

### `logs`

Subscribe to event logs matching a filter.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address` | string or string[] | No | Contract address(es) to filter |
| `topics` | array | No | Up to 4 topic filters (`null` = any) |

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_subscribe",
  "params": [
    "logs",
    {
      "address": "0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
      "topics": ["0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"]
    }
  ]
}
```

#### Notification

```json
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x5678...",
    "result": {
      "address": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
      "topics": ["0xddf252ad...", "0x000...from...", "0x000...to..."],
      "data": "0x00000000000000000000000000000000000000000000000000000000000f4240",
      "blockNumber": "0x1312D01",
      "transactionHash": "0x...",
      "logIndex": "0x0",
      "removed": false
    }
  }
}
```

---

### `newPendingTransactions`

Subscribe to pending transaction hashes.

#### Request

```json
{ "jsonrpc": "2.0", "id": 1, "method": "eth_subscribe", "params": ["newPendingTransactions"] }
```

#### Notification

```json
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x9abc...",
    "result": "0x3847245c01829b043431067fb2bfa95f7b5bdc7e..."
  }
}
```

---

## `eth_unsubscribe`

Cancels a subscription.

### Request

```json
{ "jsonrpc": "2.0", "id": 1, "method": "eth_unsubscribe", "params": ["0x1234...subscription_id..."] }
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": true }
```

---

## Example (Node.js)

```ts
import WebSocket from "ws";

const ws = new WebSocket(
  `wss://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`
);

ws.on("open", () => {
  ws.send(JSON.stringify({
    jsonrpc: "2.0", id: 1, method: "eth_subscribe", params: ["newHeads"]
  }));
});

ws.on("message", (data) => {
  const msg = JSON.parse(data.toString());
  if (msg.method === "eth_subscription") {
    console.log("New block:", msg.params.result.number);
  }
});
```

---

## Solana PubSub Subscriptions

Solana exposes a separate PubSub WebSocket API. Use the Solana base URL:

```text
wss://solana-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY
wss://solana-devnet.g.alchemy.com/v2/$ALCHEMY_API_KEY
```

Do **not** use `eth_subscribe` on Solana. Each method is named `<event>Subscribe` and is paired with a matching `<event>Unsubscribe`. A successful subscribe returns a numeric subscription id; pass that id to the unsubscribe call to cancel the stream. Most methods accept an optional `commitment` parameter (defaults to `finalized`).

### Methods

| Category | Method | Purpose |
|----------|--------|---------|
| Accounts | `accountSubscribe` | Notify when one account's lamports or data change. |
| Accounts | `programSubscribe` | Notify on accounts owned by a program. Scope with `dataSize` / `memcmp` filters; unfiltered streams can be very high bandwidth. |
| Transactions | `logsSubscribe` | Notify on transaction log messages matching a filter (`all`, `allWithVotes`, or `{ mentions: [pubkey] }`). |
| Transactions | `signatureSubscribe` | Notify on status changes for a single transaction signature. Auto-completes once the signature reaches the requested commitment. |
| Cluster | `slotSubscribe` | Notify when the validator processes a new slot. |
| Cluster | `rootSubscribe` | Notify when the validator sets a new root slot. |

Notifications arrive as JSON-RPC messages with method-specific names (`accountNotification`, `programNotification`, `logsNotification`, `signatureNotification`, `slotNotification`, `rootNotification`). The payload is in `params.result` with the matching `params.subscription` id.

### Example (`@solana/web3.js`)

```ts
import { Connection, PublicKey } from "@solana/web3.js";

const connection = new Connection(
  `https://solana-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`,
  {
    wsEndpoint: `wss://solana-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`,
    commitment: "confirmed",
  }
);

const pubkey = new PublicKey("So11111111111111111111111111111111111111112");
const subId = await connection.onAccountChange(pubkey, (accountInfo, ctx) => {
  console.log("slot:", ctx.slot, "lamports:", accountInfo.lamports);
});

// Later: await connection.removeAccountChangeListener(subId);
```

### Solana subscription billing

Solana WebSocket subscriptions are billed by **bandwidth** (per byte delivered), not per-message. Broad streams — especially unfiltered `programSubscribe` and `logsSubscribe` `all` / `allWithVotes` — can scale CU usage quickly.

- Always scope with `mentions` / `mentionsAccountOrProgram` / `dataSize` / `memcmp` where supported.
- Keep payloads small: prefer `encoding: "base64"` over `jsonParsed`, and `transactionDetails: "signatures"` when you don't need full tx data.
- Set [usage limits](https://www.alchemy.com/docs/how-to-set-usage-limits-and-alerts-for-your-account) before deploying broad subscriptions in production.
- See [Compute Unit Costs — Solana WebSocket Subscriptions](https://www.alchemy.com/docs/reference/compute-unit-costs#solana-websocket-subscriptions) for the per-byte CU rate.

For higher-throughput, structured Solana streaming (account / slot / transaction streams with server-side filters), consider Yellowstone gRPC instead — see the `solana-grpc-*` references.

---

## Notes

- Subscriptions are stateful. Handle reconnects and resubscribe after reconnect.
- You may receive duplicate events on reconnect. De-duplicate by block hash or log index.
- `newPendingTransactions` is very high volume. Use tight filters if available, or switch to `alchemy_pendingTransactions` with `addresses` and `hashesOnly: true` to keep bandwidth (and billing) predictable.
- If WebSockets are unavailable, fall back to HTTP polling with coarse intervals and backoff.

## Official Docs
- [Subscription API Overview](https://www.alchemy.com/docs/reference/subscription-api)
- [eth_subscribe](https://www.alchemy.com/docs/chains/ethereum/ethereum-api-endpoints/eth-subscribe)
- [Solana Subscription API Endpoints](https://www.alchemy.com/docs/reference/solana-subscription-api-endpoints)
- [accountSubscribe](https://www.alchemy.com/docs/reference/account-subscribe), [programSubscribe](https://www.alchemy.com/docs/reference/program-subscribe), [logsSubscribe](https://www.alchemy.com/docs/reference/logs-subscribe), [signatureSubscribe](https://www.alchemy.com/docs/reference/signature-subscribe), [slotSubscribe](https://www.alchemy.com/docs/reference/slot-subscribe), [rootSubscribe](https://www.alchemy.com/docs/reference/root-subscribe)
