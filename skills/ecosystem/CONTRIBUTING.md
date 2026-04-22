# Contributing an Ecosystem Skill

This guide is for Alchemy ecosystem partners adding skills to this repo.

## Curation principle (read this first)

The ecosystem is **curated for complementary, non-overlapping capabilities**. Before you start, confirm your skill covers something Alchemy does not provide first-party.

We do **not** accept ecosystem skills that compete with first-party Alchemy capabilities, including:

- EVM JSON-RPC and node-level reads
- Solana RPC and DAS
- Writes / signed transactions / transaction submission
- Transaction simulation
- Account abstraction (bundlers, gas managers, paymasters)
- NFT metadata APIs
- Token prices (overlap zone — do not ship a competing prices skill)
- Webhooks and address-activity streams
- Wallet APIs / Account Kit equivalents

This is **not** a statement about partners — it is a statement about *which of a partner's skills we accept here*. A partner can have products that overlap with Alchemy in other parts of their catalog. That doesn't disqualify them. It only means we will not accept the overlapping ones into this repo. We will happily accept their non-overlapping skills.

Why: agents route on `description` and `scope_in`. Two skills claiming the same capability create routing ambiguity that usually fails silently (wrong skill picked, wrong answer returned). A curated complementary catalog keeps routing deterministic.

If you're unsure whether your skill overlaps, open a draft PR and ask. We'd rather have the conversation early than reject at review.

## Quick start

1. Fork `alchemyplatform/skills`
2. Confirm your skill is non-overlapping (see above)
3. Copy `skills/ecosystem/TEMPLATE/` to `skills/ecosystem/skills/<your-skill>/`
4. Fill in `SKILL.md` (instructions, scope contract, routing back to first-party). Optionally fill in `agents/openai.yaml` for a polished Codex picker entry — see "OpenAI/Codex manifest" below
5. Add your skill's path to [`/.claude-plugin/plugin.json`](../../.claude-plugin/plugin.json) at the repo root
6. Open a PR

## Skill contract

Every ecosystem `SKILL.md` must include the following beyond the standard [Agent Skills](https://agentskills.io/specification) frontmatter, under `metadata:`:

```yaml
---
name: <your-skill>
description: <one paragraph; ≤500 chars; what it does and when to use>
license: MIT
compatibility: <env vars, deps, network requirements>
metadata:
  author: <your-org>
  version: "0.1"
  provider: <your-org>           # your company / project name
  partner: "true"                # always "true" for ecosystem skills
  expires: <YYYY-MM-DD>          # ~6 months out; bump on each meaningful PR
---
```

In the body of the `SKILL.md`, every ecosystem skill must include a **`## Scope contract`** section that explicitly lists:

- **What this skill covers** (`scope_in` — what an agent should reach for this skill to do)
- **What this skill does NOT cover, with handoff** (`scope_out` — for each non-coverage, name the skill an agent should hand off to)

This is what allows agents to route correctly between first-party Alchemy skills and ecosystem skills without confusion.

The standard handoffs to first-party Alchemy skills (use these in your `scope_out`):

| If the user needs… | Hand off to |
| --- | --- |
| Live agent work in this session, CLI installed | `alchemy-cli` |
| Live agent work in this session, MCP only | `alchemy-mcp` |
| App code with an Alchemy API key | `alchemy-api` |
| App code without an API key (autonomous agent / x402 / MPP) | `agentic-gateway` |

## Manifest entry

After adding the skill folder, add its path to `/.claude-plugin/plugin.json`:

```json
{
  "name": "alchemy",
  "skills": [
    "./skills/ecosystem/skills/<your-skill>"
  ]
}
```

Without this entry, `npx skills add alchemyplatform/skills` will not discover your skill (the Vercel CLI does not recursively walk `./skills/`; it only checks direct children).

## OpenAI/Codex manifest

The TEMPLATE includes `agents/openai.yaml`, which is consumed by [OpenAI Codex](https://developers.openai.com/codex/skills) to render your skill in the picker. **Recommended but not required** — if you skip it, your skill still installs and works; Codex falls back to deriving picker text from your `SKILL.md` frontmatter, which is denser and less picker-friendly but functional.

The Vercel `skills` CLI bundles this file with the rest of the skill when installing with `--agent codex`; Codex reads it from the installed location.

```yaml
interface:
  display_name: "<Your Skill Display Name>"
  short_description: "<one-line summary, 25–64 chars, for the picker>"
  default_prompt: "<one-sentence prompt injected when this skill is selected>"
```

`display_name` and `short_description` should be picker-friendly — the `SKILL.md` frontmatter `description` is too dense for that surface (it is optimized for agent disambiguation, often 300–500 chars). `default_prompt` is what Codex pre-fills when a user picks your skill.

Optional fields in the OpenAI spec: `icon_small`, `icon_large` (relative paths to icon assets) and `brand_color` (hex accent). Not required.

## License

Ecosystem skills are MIT-licensed by default. If you require a different license, document it in your skill's `LICENSE.txt` and call it out in the PR.

## Review

PRs are reviewed by the Alchemy skills team. Maintenance and bugfixes after merge are the partner's responsibility. The `expires` date is a forcing function — when it lapses, agents will be prompted to refetch your skill, so keep it current.
