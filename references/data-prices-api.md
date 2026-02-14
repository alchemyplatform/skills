---
id: references/data-prices-api.md
name: 'Prices API'
description: 'Query token prices for current and historical data using Alchemy''s Prices API.'
tags:
  - alchemy
  - data-apis
  - data
related:
  - recipes-get-prices-current-historical.md
updated: 2026-02-05
---
# Prices API

## Summary
Query token prices for current and historical data using Alchemy's Prices API.

## Primary Use Cases
- Portfolio valuation.
- Price charts and historical analytics.
- Risk calculations.

## When To Use / Not Use
- Use for spot and historical prices.
- Avoid if you require exchange-level order book data.

## Auth & Setup
- Base URL: `https://api.g.alchemy.com/prices/v1/$ALCHEMY_API_KEY`

## Network Selection
- Symbol-based endpoints are network-agnostic.
- Address-based endpoints require a `network` per token address.

## Endpoints / Methods (Spec-Derived)
### `GET /tokens/by-symbol`
- Query param: `symbols` (comma-separated string, e.g., `ETH,BTC`)

### `POST /tokens/by-address`
- Body field: `addresses[]` (array of `{ network, address }`)
- Limits: max 25 addresses, max 3 networks

### `POST /tokens/historical`
- Body supports one of:
- `{ symbol, startTime, endTime, interval?, withMarketData? }`
- `{ network, address, startTime, endTime, interval?, withMarketData? }`
- `startTime` / `endTime` can be ISO-8601 strings or seconds since epoch.
- `interval` enum: `5m`, `1h`, `1d`
- Max ranges: `5m` (7d), `1h` (30d), `1d` (1yr)

## Example Requests
```bash
curl -s "https://api.g.alchemy.com/prices/v1/$ALCHEMY_API_KEY/tokens/by-symbol?symbols=ETH,BTC"
```

```bash
curl -s -X POST "https://api.g.alchemy.com/prices/v1/$ALCHEMY_API_KEY/tokens/historical" \
  -H "Content-Type: application/json" \
  -d '{"symbol":"ETH","startTime":"2024-01-01T00:00:00Z","endTime":"2024-01-02T00:00:00Z"}'
```

```bash
curl -s -X POST "https://api.g.alchemy.com/prices/v1/$ALCHEMY_API_KEY/tokens/by-address" \
  -H "Content-Type: application/json" \
  -d '{"addresses":[{"network":"eth-mainnet","address":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"}]}'
```

## Error Handling
- Handle `429` and invalid symbol errors.

## Performance / Limits / Compute Units
- Prefer larger intervals for long ranges.

## Gotchas & Edge Cases
- Not all tokens are supported; fallback to a supported symbol list.

## Testing / Mocking
- Use fixed date ranges to keep results stable.

## Related Files
- `recipes-get-prices-current-historical.md`

## Official Docs
- [Prices API Reference](https://www.alchemy.com/docs/reference/prices-api)
