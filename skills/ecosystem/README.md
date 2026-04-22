# Ecosystem Skills

Third-party Agent Skills contributed by Alchemy ecosystem partners, **for capabilities Alchemy does not provide first-party**.

## Curation principle: complementary, not competitive

The ecosystem catalog is **curated for non-overlap**. We accept partner skills only for capabilities outside Alchemy's first-party surface — never for features that compete with our own.

A partner can have products that overlap with Alchemy elsewhere; that does not disqualify them from the ecosystem. It only constrains *which of their skills* we accept here. If a partner offers both overlapping and non-overlapping capabilities, we only accept the **non-overlapping** ones into this catalog.

Why: overlap confuses agents at routing time. Two skills claiming the same capability force the agent (or the user) to pick a winner mid-task, which usually goes badly. Keeping the catalog complementary means an agent can pick the right skill from the description alone.

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) → "Curation principle" for the precise rules.

## Layout

```
skills/ecosystem/
├── README.md            ← this file
├── CONTRIBUTING.md      ← how partners PR a new skill
├── TEMPLATE/            ← copy this to bootstrap a new skill
│   ├── SKILL.md
│   ├── agents/
│   │   └── openai.yaml  ← OpenAI/Codex picker manifest (recommended)
│   └── references/
└── skills/
    └── <partner>/       ← partner skill (e.g. allium/)
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        └── references/
```

Each partner gets one folder per skill. If a partner ships multiple non-overlapping skills, they live as siblings (e.g. `partner-data/`, `partner-staking/`).

## Discovery

Ecosystem skills live deeper than the Vercel `skills` CLI's default scan path (which only walks direct children of `./skills/`). Discovery for these paths is driven by [`/.claude-plugin/plugin.json`](../../.claude-plugin/plugin.json) at the repo root, which the CLI reads as a documented escape hatch (see [Plugin Manifest Discovery](https://github.com/vercel-labs/skills#plugin-manifest-discovery) in the CLI README).

When adding a new partner skill, add an entry to `plugin.json` so it gets installed by `npx skills add alchemyplatform/skills`. Without a manifest entry, the CLI will not find skills nested under `skills/ecosystem/skills/`.

## Current partners

- **Allium** — wallet PnL (current + historical, by-wallet and by-token), holdings timeseries, Hyperliquid trading data (info, fills, orders, orderbook), and custom SQL analytics over Allium's data warehouse ([`./skills/allium/`](./skills/allium/)). Curated subset of [allium-labs/skills](https://github.com/allium-labs/skills) — overlapping endpoints (prices, token metadata, current balances, transactions, NFT) are intentionally omitted and routed to first-party Alchemy skills.

## Routing for agents

| The user wants… | Skill |
| --- | --- |
| Real-time blockchain reads, writes, account abstraction, simulation | first-party Alchemy skills (`alchemy-cli`, `alchemy-mcp`, `alchemy-api`, `agentic-gateway`) |
| Token prices, token metadata, current wallet balances, transaction transfer history, NFT metadata | first-party `alchemy-api` |
| Wallet PnL (current or historical) | `allium` |
| Holdings timeseries history | `allium` |
| Hyperliquid trading data (fills, orders, positions, orderbook) | `allium` |
| Custom SQL on a blockchain data warehouse (DeFi, NFT, bridges, MEV, Solana staking) | `allium` |

See each skill's `SKILL.md` for its exact `scope_in` / `scope_out`.
