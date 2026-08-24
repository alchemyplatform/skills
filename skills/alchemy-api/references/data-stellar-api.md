---
id: references/data-stellar-api.md
name: 'Stellar Data API'
description: 'Query indexed Stellar data (transfers, balances, NFT holdings) across native XLM, classic trustline assets, and Soroban contract tokens without running an indexer.'
tags:
  - alchemy
  - data-apis
  - data
  - stellar
related:
  - data-overview.md
  - solana-overview.md
updated: 2026-06-24
---
# Stellar Data API

Query indexed Stellar data: transfers, balances, and NFT holdings across native XLM, classic (SEP-0011 trustline) assets, and Soroban contract tokens. REST endpoints (POST with JSON body).

**Base URL**: `https://api.g.alchemy.com` (unified gateway, NOT the per-network `<network>.g.alchemy.com/v2/<key>` shape used by EVM Data APIs).

**Auth**: API key as a Bearer token in the `Authorization` header (`Authorization: Bearer <your-api-key>`). No API key in the URL.

**Networks** (request body field): `mainnet`, `testnet`.

**Common shape**: every endpoint returns `200 OK` with `{ "data": { ... } }`. Pagination uses an opaque `pageKey` cursor; pass back verbatim to fetch the next page.

When to use this vs `Stellar API` (JSON-RPC): use the Data API for historical reads (transfer history, current balances, NFT inventories) without running your own indexer. Use the [Stellar JSON-RPC](https://www.alchemy.com/docs/stellar/stellar-api-overview) for live ledger data, transaction submission, simulation, and Soroban contract calls.

---

## `POST /data/stellar/transfers` (`getStellarTransfers`)

Returns transfers (native, classic, and/or Soroban) matching the filters, ordered by ledger.

### Request body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `network` | string | Yes | — | `"mainnet"` or `"testnet"` |
| `fromAddress` | string | No | — | G… account address (sender filter) |
| `toAddress` | string | No | — | G… account address (recipient filter) |
| `transferType` | string[] | No | all | Subset of `["native", "classic", "soroban"]` |
| `assetCode` | string | No | — | Classic asset code (e.g. `"USDC"`) |
| `assetIssuer` | string | No | — | G… account address of classic asset issuer |
| `contractId` | string | No | — | C… Soroban contract id |
| `fromLedger` | integer | No | — | Inclusive lower bound on ledger sequence |
| `toLedger` | integer | No | — | Inclusive upper bound on ledger sequence |
| `order` | string | No | `"desc"` | `"asc"` or `"desc"` by ledger |
| `limit` | integer | No | `1000` | 1–1000 transfers per page |
| `pageKey` | string | No | — | Opaque cursor from a prior response |
| `excludeFailedTransactions` | boolean | No | `false` | Omit transfers from failed transactions when `true` |

### Request

```bash
curl -s -X POST https://api.g.alchemy.com/data/stellar/transfers \
  -H "Authorization: Bearer $ALCHEMY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "network": "mainnet",
    "fromAddress": "GAAZI4TCR3TY5OJHCTJC2A4QSY6CJWJH5IAJTGKIN2ER7LBNVKOCCWN7",
    "limit": 100,
    "order": "desc"
  }'
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `data.transfers[]` | array | Matching transfers |
| `data.transfers[].ledgerSequence` | integer | Ledger sequence the op was included in |
| `data.transfers[].txHash` | string | Transaction hash |
| `data.transfers[].opIndex` | integer | Operation index within the transaction |
| `data.transfers[].timestamp` | string | ISO-8601 ledger close time |
| `data.transfers[].transferType` | string | `"native"`, `"classic"`, or `"soroban"` |
| `data.transfers[].assetCode` | string | `"XLM"` for native, classic code for classic, absent for soroban |
| `data.transfers[].assetIssuer` | string | Issuer account for classic transfers |
| `data.transfers[].contractId` | string | Soroban contract id for soroban transfers |
| `data.transfers[].from` | string | Sender account |
| `data.transfers[].to` | string | Recipient account |
| `data.transfers[].amount` | string | 7-decimal display units for native/classic; base-unit integer string for raw Soroban |
| `data.transfers[].success` | boolean | Whether the enclosing transaction succeeded |
| `data.pageKey` | string | Cursor for the next page, absent when exhausted |

---

## `POST /data/stellar/balances` (`getStellarBalances`)

Returns the unified balance set for an account: native XLM, classic trustlines, and Soroban contract tokens. Native XLM is emitted only on the first page (when `pageKey` is omitted).

### Request body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `network` | string | Yes | — | `"mainnet"` or `"testnet"` |
| `address` | string | Yes | — | G… account address to fetch balances for |
| `limit` | integer | No | — | Max non-native entries per page (1–1000) |
| `pageKey` | string | No | — | Opaque cursor; omit for the first page (which includes native XLM) |

### Request

```bash
curl -s -X POST https://api.g.alchemy.com/data/stellar/balances \
  -H "Authorization: Bearer $ALCHEMY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "network": "mainnet",
    "address": "GAAZI4TCR3TY5OJHCTJC2A4QSY6CJWJH5IAJTGKIN2ER7LBNVKOCCWN7",
    "limit": 100
  }'
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `data.balances[]` | array | Mixed native / classic / soroban balance rows |
| `data.balances[].type` | string | `"native"`, `"classic"`, or `"soroban"` (discriminates which fields below are populated) |
| `data.balances[].assetCode` | string | `"XLM"` for native; the code for classic; absent for soroban |
| `data.balances[].assetIssuer` | string | Issuer account (classic only) |
| `data.balances[].contractId` | string | Soroban contract id (soroban only) |
| `data.balances[].balance` | string | 7-decimal display units for native/classic; base-unit integer string for raw Soroban |
| `data.balances[].trustlineExists` | boolean | Native/classic only |
| `data.balances[].authorized` | boolean | Classic only — whether the trustline is authorized |
| `data.balances[].limit` | string | Classic only — trustline limit |
| `data.asOfLedger` | integer | Ledger sequence the balances were computed as of |
| `data.pageKey` | string | Cursor for the next page (omit native XLM); absent when exhausted |

---

## `POST /data/stellar/nfts` (`getStellarNfts`)

Returns NFT holdings for an account. Covers Soroban (SEP-0050) and classic (SEP-0039) NFTs.

### Request body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `network` | string | Yes | — | `"mainnet"` or `"testnet"` |
| `address` | string | Yes | — | G… account address |
| `contractId` | string | No | — | Restrict to a single Soroban contract |
| `assetCode` | string | No | — | Restrict to a classic asset code (must be paired with `assetIssuer`) |
| `assetIssuer` | string | No | — | Classic asset issuer account (paired with `assetCode`) |
| `limit` | integer | No | — | 1–1000 holdings per page |
| `pageKey` | string | No | — | Opaque cursor |

Filtering is exclusive: set `contractId` OR (`assetCode` + `assetIssuer`). Setting both classes of filter together is rejected.

### Request

```bash
curl -s -X POST https://api.g.alchemy.com/data/stellar/nfts \
  -H "Authorization: Bearer $ALCHEMY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "network": "mainnet",
    "address": "GAAZI4TCR3TY5OJHCTJC2A4QSY6CJWJH5IAJTGKIN2ER7LBNVKOCCWN7",
    "limit": 100
  }'
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `data.nfts[]` | array | NFT holdings |
| `data.nfts[].type` | string | `"soroban"` (contractId + tokenId) or `"classic"` (assetCode + assetIssuer) |
| `data.nfts[].contractId` | string | Soroban contract id (soroban holdings) |
| `data.nfts[].tokenId` | string | Token id within the Soroban contract (soroban holdings) |
| `data.nfts[].assetCode` | string | Classic asset code (classic holdings) |
| `data.nfts[].assetIssuer` | string | Classic asset issuer account (classic holdings) |
| `data.asOfLedger` | integer | Ledger sequence the holdings were computed as of |
| `data.pageKey` | string | Cursor for the next page; absent when exhausted |

---

## Notes

- Authentication is Bearer-token, NOT URL-embedded. Do not put the API key in the path; put it in the `Authorization` header.
- All three endpoints are POST with a JSON body, even read-only ones. GET is not supported.
- Amount precision is asymmetric: native and classic use 7-decimal display units (`"100.0000000"`); raw Soroban rows return base-unit integer strings (`"1000000000000"`) because contracts choose their own decimals.
- `transferType` and `contractId` on `getStellarTransfers` let you scope to a single Soroban contract's activity without scanning every operation.
- `getStellarBalances` returns native XLM only on the first page. If you paginate, do NOT re-include the XLM row from earlier pages — it's already been emitted.

## Official Docs
- [Stellar Data API Overview](https://www.alchemy.com/docs/reference/stellar-data-api-overview)
- [Stellar JSON-RPC overview](https://www.alchemy.com/docs/stellar/stellar-api-overview) (for live ledger data and contract calls)
