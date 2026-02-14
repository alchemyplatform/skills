---
id: references/solana-rpc.md
name: 'Solana JSON-RPC'
description: 'Standard Solana JSON-RPC endpoints for account, program, and transaction data.'
tags:
  - alchemy
  - solana
  - json-rpc
related:
  - solana-das-api.md
  - solana-wallets.md
updated: 2026-02-05
---
# Solana JSON-RPC

## Summary
Standard Solana JSON-RPC endpoints for account, program, and transaction data.

## Primary Use Cases
- Fetch SOL balances.
- Query account data and program state.
- Retrieve signatures and transaction details.

## Auth & Setup
- Base URL pattern: `https://solana-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY`

## Common Methods
- `getBalance`
- `getAccountInfo`
- `getProgramAccounts`
- `getSignaturesForAddress`
- `getTransaction`

## Example Requests
```bash
curl -s https://solana-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["4Nd1m...pubkey..."]}'
```

## Gotchas
- Solana returns base58 addresses and base64 account data.
- Use `encoding` and `commitment` parameters explicitly for consistency.

## Related Files
- `solana-das-api.md`
- `solana-wallets.md`

## Official Docs
- [Solana API Quickstart](https://www.alchemy.com/docs/reference/solana-api-quickstart)
