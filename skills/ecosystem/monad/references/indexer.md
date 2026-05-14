# Monad Indexer (HyperIndex on Envio Cloud)

For activity feeds, leaderboards, transaction history, analytics dashboards, or anything else that needs to read **historical** on-chain events on Monad — Alchemy does not yet provide a Transfers / NFT / Token API for Monad mainnet, so the Monad-native path is HyperIndex deployed on Envio Cloud.

## When to use HyperIndex on Monad

- Building a frontend that needs more than a single `eth_call` can provide — pagination across history, multi-contract joins, filtered streams, leaderboards.
- Capturing on-chain events as triggers for off-chain logic.
- Activity feeds, transaction history pages, analytics dashboards.

For one-shot reads or live state, use `alchemy-cli` / `alchemy-api` JSON-RPC instead.

## Prerequisites

The user must have **all four** of these in place before any indexer command works:

1. `envio-cloud` installed globally — `npm install -g envio-cloud`
2. `envio-cloud` logged in — `envio-cloud login` (browser flow, 30-day session)
3. GitHub CLI installed — `brew install gh` on macOS, or see https://cli.github.com
4. GitHub CLI logged in — `gh auth login`

GitHub is required because Envio Cloud deploys from GitHub.

**Do not install either CLI on the user's behalf, and do not run `login` for them.** If anything is missing, surface the exact command and wait.

The contract you're indexing must be **deployed AND verified** on a Monad explorer before `contract-import` runs. `contract-import` pulls the ABI from the explorer — unverified contract = import failure.

## Initialize an indexer

Run inside an `indexer/` folder at the project root.

### Monad mainnet

```bash
mkdir -p indexer && cd indexer
pnpx envio@3.0.0-alpha.21 init contract-import explorer \
  -b monad \
  -c <CONTRACT_ADDRESS> \
  -n <CONTRACT_NAME> \
  -l typescript \
  -d ./ -o ./ \
  --all-events --single-contract --api-token ""
```

### Monad testnet

```bash
mkdir -p indexer && cd indexer
pnpx envio@3.0.0-alpha.21 init contract-import explorer \
  -b monad-testnet \
  -c <CONTRACT_ADDRESS> \
  -n <CONTRACT_NAME> \
  -l typescript \
  -d ./ -o ./ \
  --all-events --single-contract --api-token ""
```

**Notes:**

- `<CONTRACT_NAME>` should match the Solidity contract name (e.g. `MyToken`).
- `--all-events` imports every event in the ABI. Narrow later by editing `config.yaml`.
- `--single-contract` scaffolds for one contract. Re-run for additional contracts or hand-edit the config.
- `--api-token ""` is intentional — leave it empty.
- `-l typescript` is a flag, not a positional — the CLI rejects bare `typescript`.
- Version is pinned to `envio@3.0.0-alpha.21` — use exactly this.

## Opt into transaction fields before writing handlers

By default `event.transaction.*` is typed `never` in generated handlers — accessing `event.transaction.hash` (or any other tx field) is a TypeScript error. Most frontends want the tx hash for explorer links and dedup, so opt in **before** writing handler code.

Add to `config.yaml` (top level, sibling of `networks:` / `contracts:`):

```yaml
field_selection:
  transaction_fields:
    - hash
```

After editing, re-run codegen so the types regenerate:

```bash
pnpm codegen
# or
pnpx envio@3.0.0-alpha.21 codegen
```

Add other fields the handlers need (`from`, `to`, `value`, `gasUsed`, `input`, etc.) to the same list. Envio only pulls and types the fields listed, so keep it minimal.

Full field list: https://docs.envio.dev/docs/HyperIndex/configuration-file#field-selection

## Exit-code contract

Every `envio-cloud` command follows the same convention — check this, not just stdout:

| Exit code | Meaning | Reaction |
| --- | --- | --- |
| `0` | Success | Continue |
| `1` | User error (bad args, unknown indexer, not logged in) | Read stderr, fix the input, retry |
| `2` | API / server error (not the user's fault) | Retry once. If it persists, surface to the user and stop. |

## What's covered in upstream monskills

The upstream `indexer/` skill in `therealharpaljadeja/monskills` covers in detail:

- **Workflow recipes** — first deploy, debugging failed deploys, env-var rotation, IP allowlisting.
- **CLI reference** — every `envio-cloud` command grouped by area (auth, context, indexer lifecycle, deployment, env, security).

Install the full skill via `npx skills add therealharpaljadeja/monskills` or browse: https://github.com/therealharpaljadeja/monskills/tree/main/indexer

## Official Envio docs

https://docs.envio.dev/docs/HyperIndex/envio-cloud-cli
