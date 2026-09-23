---
id: references/webhooks-address-activity.md
name: 'Address Activity Webhooks'
description: 'Address activity webhooks notify you when specified addresses send or receive assets.'
tags:
  - alchemy
  - webhooks
  - webhook
related:
  - webhooks-webhook-payloads.md
  - webhooks-verify-signatures.md
  - data-transfers-api.md
updated: 2026-09-23
---
# Address Activity Webhooks

## Summary
Address activity webhooks notify you when specified addresses send or receive assets.

## Primary Use Cases
- Wallet notifications.
- Compliance monitoring.
- Trigger downstream indexers.

## Spec-Derived Fields
Create webhook (`POST /create-webhook`) with:
- `webhook_type: ADDRESS_ACTIVITY`
- `addresses[]` (required)

Update addresses (`PATCH /update-webhook-addresses`) with:
- `webhook_id` (required)
- `addresses_to_add[]` (required, empty array if none)
- `addresses_to_remove[]` (required, empty array if none)

## Chain Support
- Supports 30+ EVM chains (ETH, ERC-20, ERC-721, ERC-1155 transfers).
- Now available on Solana (no longer beta).

## Internal transfer support
Internal transfers (in addition to native + ERC-20 + ERC-721 + ERC-1155 top-level transfers) are emitted for a narrower chain set: Ethereum (Mainnet, Sepolia), Polygon (Mainnet, Amoy), Arbitrum (Mainnet, Sepolia), Optimism (Mainnet, Sepolia), Base (Mainnet, Sepolia), BNB Chain (Mainnet, Testnet), Avalanche (Mainnet, Fuji), Robinhood Chain (Mainnet, Testnet), and Arc (Mainnet, Testnet). `delegatecall` internal transfers and miner-reward internal transfers are not tracked. Other networks may add internal transfer coverage later — contact support to request a specific chain.

## Configuration Notes
- Provide one or more addresses to watch.
- Choose event categories (native, ERC-20, ERC-721, ERC-1155).
- Decide if you need internal transfers (see the chain list above).
- Maximum of 100,000 addresses per webhook.

## Payload Expectations
- Expect an array of activities with tx hash, from/to, value, token metadata.

## Operational Guidance
- De-dupe events by tx hash + log index.
- Store mapping of webhook ID to address list.

## Related Files
- `webhooks-webhook-payloads.md`
- `webhooks-verify-signatures.md`
- `data-transfers-api.md`

## Official Docs
- [Notify API Quickstart](https://www.alchemy.com/docs/reference/notify-api-quickstart)
