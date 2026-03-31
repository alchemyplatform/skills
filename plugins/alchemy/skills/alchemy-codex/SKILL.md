---
name: alchemy-codex
description: Codex-facing entry point for building with Alchemy. Routes to `alchemy-api` for API key-based access or `agentic-gateway` for gateway-based access when the user has not chosen an auth path yet.
license: MIT
compatibility: Requires network access. Works across Codex, Claude Code, and API-based agent workflows.
metadata:
  author: alchemyplatform
  version: "1.0"
---
# Alchemy Codex

Use this skill when the user wants to build with Alchemy in Codex but has not yet chosen how requests should be authenticated.

This skill routes to:

- `alchemy-api` for standard API key-based access to Alchemy JSON-RPC, Data APIs, Webhooks, and Wallets
- `agentic-gateway` for gateway-based access using an API key or wallet-based payment flows

## Required first question

Before making a network call or implementation choice, ask:

> Do you want to use an existing Alchemy API key, or should I use the agentic gateway flow instead?

Then route as follows:

- If the user chooses an API key, switch to `alchemy-api`
- If the user chooses the gateway flow, switch to `agentic-gateway`

## Use this skill for

- broad Alchemy integration requests where auth is still undecided
- onboarding a developer to the right Alchemy access path
- routing requests that mention Alchemy generally rather than a specific API surface

## Typical tasks after routing

- JSON-RPC reads and writes
- Token, NFT, Transfers, Prices, and Portfolio APIs
- transaction simulation and traces
- Webhooks / Notify setup
- Account Kit and wallet infrastructure
- agentic gateway access via x402 or MPP

## Codex bundle notes

- Treat this as a thin Codex router skill, not a place for detailed API or payment-flow instructions
- Route implementation details to `alchemy-api` or `agentic-gateway` as soon as the auth path is clear
- Keep this bundled copy aligned with `skills/alchemy-codex/SKILL.md`
