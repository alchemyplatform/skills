# Alchemy Skills

Agent Skills for integrating [Alchemy](https://www.alchemy.com/) APIs into AI-powered applications. This repo contains two skills:

## Skills

### `skills/alchemy-api`
Traditional API key-based access to all Alchemy products: EVM JSON-RPC, Token API, NFT API, Prices API, Portfolio API, Webhooks, Solana, and more.

- **Auth**: API key in URL or header
- **Setup**: Create a free key at [dashboard.alchemy.com](https://dashboard.alchemy.com/)
- **Entry point**: [`skills/alchemy-api/SKILL.md`](skills/alchemy-api/SKILL.md)

### `skills/agentic-gateway`
Lets agents easily access Alchemy's developer platform. Uses SIWE (Sign-In With Ethereum) authentication and x402 USDC payments.

- **Auth**: SIWE token + x402 USDC payment
- **Setup**: Generate a wallet and fund it with USDC
- **Entry point**: [`skills/agentic-gateway/SKILL.md`](skills/agentic-gateway/SKILL.md)

## Which skill should I use?

| Scenario | Skill |
|---|---|
| I have an Alchemy API key | `alchemy-api` |
| I'm building a traditional server or dApp | `alchemy-api` |
| I'm an agent that needs easy access to Alchemy's developer platform | `agentic-gateway` |
| I need API method details (params, responses) | `alchemy-api` (then cross-ref gateway URLs) |
| I need SIWE auth or x402 payment setup | `agentic-gateway` |

## Installation

```bash
npx skills add alchemyplatform/skills --yes
```

## Specification

These skills follow the [Agent Skills specification](https://agentskills.io/specification). See [spec/agent-skills-spec.md](spec/agent-skills-spec.md) for details.

## Official Links

- [Developer docs](https://www.alchemy.com/docs)
- [Get Started guide](https://www.alchemy.com/docs/get-started)
- [Create a free API key](https://dashboard.alchemy.com/)

## License

MIT
