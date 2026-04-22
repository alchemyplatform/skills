---
name: <your-skill>
description: One-paragraph description of what this skill does and when to use it. ≤500 characters. Include keywords agents will search for. Make non-coverage explicit (e.g. "NOT for X — use <other-skill> instead").
license: MIT
compatibility: List required env vars (e.g. $YOUR_API_KEY), runtime deps, and network requirements.
metadata:
  author: <your-org>
  version: "0.1"
  provider: <your-org>
  partner: "true"
  expires: "YYYY-MM-DD"
---

# <Your Skill Title>

> **Replace this whole file** when you copy this template into `skills/ecosystem/skills/<your-skill>/`. Update the `name` (must match the parent directory name), the `description`, and every section below.

Brief intro to what this skill does in 1–2 sentences.

## When to use this skill

Use `<your-skill>` when **all** of the following are true:

- <condition>
- <condition>

## When NOT to use this skill (handoff)

Replace the `<your-non-coverage>` row with anything *your* skill doesn't cover. The first-party Alchemy rows are the standard handoffs every ecosystem skill should include — keep them as-is unless your `scope_out` is unusual.

| Need | Use instead |
| --- | --- |
| Real-time blockchain reads, node-level fresh | `alchemy-cli` (live work) or `alchemy-api` (app code) |
| Writes / signed transactions | `alchemy-api` (with API key) or `agentic-gateway` (without) |
| Account abstraction, simulation, NFT metadata, prices, webhooks | `alchemy-api` |
| <your-non-coverage> | `<other-skill>` |

## Scope contract

**This skill covers (`scope_in`):**

- <bullet>
- <bullet>

**This skill does NOT cover (`scope_out`):**

- Real-time / node-fresh reads → handoff: `alchemy-cli` or `alchemy-api`
- Writes / signed transactions → handoff: `alchemy-api` or `agentic-gateway`
- <your-non-coverage> → handoff: `<other-skill>`

## Setup

Document any auth, env vars, or dependencies a user needs before the skill works.

```bash
export YOUR_API_KEY=<your-key>
```

## Reference

Detailed coverage of each surface area, one file per topic, lives in [`./references/`](./references/).
