# Alchemy Skills

Agent Skills for integrating [Alchemy](https://www.alchemy.com/) APIs into AI-powered applications. This repo contains public skills for using Alchemy with the official CLI, API key, agentic gateway, and Codex plugin.

## Skills

### `skills/alchemy-cli`
CLI-first skill for agents with [`@alchemy/cli`](https://www.npmjs.com/package/@alchemy/cli) installed. Maps the Alchemy API surfaces to `alchemy <command>` invocations with structured JSON output.

- **Auth**: CLI manages auth internally (API key, access key, x402 wallet)
- **Setup**: `npm i -g @alchemy/cli`
- **Entry point**: [`skills/alchemy-cli/SKILL.md`](skills/alchemy-cli/SKILL.md)

### `skills/alchemy-api`
Traditional API key-based access to all Alchemy products: EVM JSON-RPC, Token API, NFT API, Prices API, Portfolio API, Webhooks, Solana, and more.

- **Auth**: API key in URL or header
- **Setup**: Create a free key at [dashboard.alchemy.com](https://dashboard.alchemy.com/)
- **Entry point**: [`skills/alchemy-api/SKILL.md`](skills/alchemy-api/SKILL.md)

### `skills/agentic-gateway`
Lets agents easily access Alchemy's developer platform. Supports three access methods: API key, x402 protocol (USDC payments), or MPP protocol (Tempo/Stripe payments). Prompts the user to choose their preferred protocol.

- **Auth**: API key, or SIWE/SIWS token + payment (x402 or MPP)
- **Protocols**: x402 (`@alchemy/x402` + `@x402/fetch`) or MPP (`mppx` + Tempo/Stripe)
- **Setup**: API key, or generate a wallet and fund it with USDC
- **Entry point**: [`skills/agentic-gateway/SKILL.md`](skills/agentic-gateway/SKILL.md)

### `skills/alchemy-codex`
Codex-facing router skill for users who know they want to build with Alchemy but have not yet chosen between a standard API key flow and the Alchemy gateway flow.

- **Auth**: Routes to the correct auth path before implementation
- **Setup**: No separate setup; delegates to the chosen Alchemy skill
- **Entry point**: [`skills/alchemy-codex/SKILL.md`](skills/alchemy-codex/SKILL.md)

## Which skill should I use?

| Scenario | Skill |
|---|---|
| I have `@alchemy/cli` installed | `alchemy-cli` |
| I have an Alchemy API key | `alchemy-api` |
| I'm building a traditional server or dApp | `alchemy-api` |
| I'm an agent that needs easy access to Alchemy's developer platform | `agentic-gateway` |
| I need API method details (params, responses) | `alchemy-api` (then cross-ref gateway URLs) |
| I need SIWE auth or x402/MPP payment setup | `agentic-gateway` |
| I want to use Alchemy in Codex but have not chosen an auth path yet | `alchemy-codex` |

## Installation

Install the CLI directly if you want the primary local Alchemy workflow:

```bash
npm i -g @alchemy/cli
```

Or install the skills collection with `npx`:

```bash
npx skills add alchemyplatform/skills --yes
```

Each source skill is self-contained and includes:

- `SKILL.md`
- `LICENSE.txt`
- `agents/openai.yaml` metadata for agent ecosystems that support it

## Specification

These skills follow the [Agent Skills specification](https://agentskills.io/specification). See [spec/agent-skills-spec.md](spec/agent-skills-spec.md) for details.

## Official Links

- [Developer docs](https://www.alchemy.com/docs)
- [Get Started guide](https://www.alchemy.com/docs/get-started)
- [Create a free API key](https://dashboard.alchemy.com/)
- [Install the Alchemy CLI](https://www.npmjs.com/package/@alchemy/cli)

## License

MIT
