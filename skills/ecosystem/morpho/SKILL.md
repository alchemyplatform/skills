---
name: morpho
description: Drive the Morpho lending protocol from the terminal via `npx @morpho-org/cli@latest` — query vaults / markets / positions / health factor and prepare unsigned Morpho transactions (deposit, withdraw, supply, borrow, repay, supply-collateral, withdraw-collateral) with built-in simulation on Ethereum and Base. Covers Morpho Blue isolated lending markets, Morpho Vaults v2 (ERC-4626), and MetaMorpho (vault v1). NOT for token prices, token metadata, current wallet balances outside Morpho positions, transaction transfer history, NFTs, or live RPC reads — for those use `alchemy-cli` (live), `alchemy-mcp`, `alchemy-api` (app code), or `agentic-gateway` (no API key). No API key required; needs Node.js and `npx`.
license: MIT
compatibility: Requires Node.js (≥ 18) and `npx` available locally. No API key required. Network access required. Supported chains via the CLI: `ethereum`, `base`. CLI prepares unsigned transactions; the user signs and submits via their wallet.
metadata:
  author: morpho
  version: "0.1"
  provider: morpho
  partner: "true"
---

# Morpho (Lending — Vaults, Markets, Positions)

Morpho is a permissionless lending protocol with isolated markets and curated vaults. This skill drives the official `@morpho-org/cli` to query protocol state and prepare unsigned Morpho transactions with simulation. For token prices, balances outside Morpho, transfers, NFTs, or live RPC, use the corresponding Alchemy skill instead.

| | |
| --- | --- |
| **Tool** | `npx @morpho-org/cli@latest <command>` |
| **Auth** | None — public reads + unsigned-tx prep |
| **Chains** | `ethereum`, `base` (every command requires `--chain`) |
| **Output** | JSON to stdout (use `--fields` / `--sort` / `--limit` to shape) |
| **Status** | Experimental (pre-v1.0); command syntax may evolve |

## When to use this skill

Use `morpho` when **any** of the following are true:

- The user wants to **explore Morpho vault APYs / TVL / allocations** ("best USDC vault on Base")
- The user wants to **compare Morpho Blue markets** ("ETH/USDC markets on mainnet")
- The user wants to **inspect Morpho positions** or **health factor** ("what are my Morpho positions")
- The user wants to **prepare a Morpho operation** — deposit, withdraw, supply, borrow, repay, supply/withdraw collateral
- The user wants to **simulate** a Morpho operation before signing

## When NOT to use this skill (handoff)

| Need | Use instead |
| --- | --- |
| Token prices (current, historical, by-timestamp) | `alchemy-api` (Prices API) |
| Token metadata, search, list by chain | `alchemy-api` (Token API) |
| Wallet balances *outside* Morpho positions | `alchemy-api` (Portfolio / Token API) |
| Transaction history (transfers in / out) | `alchemy-api` (Transfers API) |
| NFT metadata / floor / ownership | `alchemy-api` (NFT API) |
| Live blockchain reads (block #, gas, `eth_call`) | `alchemy-cli` (live) or `alchemy-api` (JSON-RPC) |
| Other lending protocols (Aave, Compound, Spark, etc.) | their respective skills, or `alchemy-api` JSON-RPC |
| Curator / allocator / market creation, liquidations, flashloans, rewards | upstream `@morpho-org/morpho-builder` skill (lower-level SDKs) |
| Pre-execution simulation beyond Morpho's built-in | `alchemy-api` (Simulation API) |
| Account abstraction (bundlers, gas managers) | `alchemy-api` |
| Signed transaction submission | the app's wallet / `alchemy-api` |

## Scope contract

**This skill covers (`scope_in`):**

- Reads: `query-vaults`, `get-vault`, `query-markets`, `get-market`, `get-positions`, `get-token-balance`, `health-check`, `get-supported-chains`
- Writes (unsigned tx prep with simulation by default): `prepare-deposit`, `prepare-withdraw`, `prepare-supply`, `prepare-borrow`, `prepare-repay`, `prepare-supply-collateral`, `prepare-withdraw-collateral`
- Standalone simulation: `simulate-transactions`
- Coverage: Morpho Blue isolated markets, Morpho Vaults v2 (ERC-4626), MetaMorpho (vault v1)

**This skill does NOT cover (`scope_out`):**

- Token prices → handoff: `alchemy-api` (Prices API)
- Token metadata, list, search → handoff: `alchemy-api` (Token API)
- Wallet balances outside Morpho positions → handoff: `alchemy-api` (Portfolio / Token API)
- Transaction transfer history → handoff: `alchemy-api` (Transfers API)
- NFT data → handoff: `alchemy-api` (NFT API)
- Live RPC reads (block details, gas, contract calls outside Morpho) → handoff: `alchemy-cli` or `alchemy-api` (JSON-RPC)
- Other lending protocols → not covered; use protocol-specific skills or raw RPC
- Curator / allocator ops, market creation, liquidations, flashloans, pre-liquidation, rewards claiming → out of scope (use upstream `@morpho-org/morpho-builder` skill or lower-level Morpho SDKs)
- Pre-execution simulation beyond Morpho's built-in → handoff: `alchemy-api` (Simulation API)
- Account abstraction → handoff: `alchemy-api` (Wallets / Bundler / Gas Manager)
- Signed tx submission → user's wallet (CLI returns unsigned payloads; app signs)

## Setup

No API key required. Needs Node.js (≥ 18) and `npx`:

```bash
node --version   # should be ≥ 18
npx @morpho-org/cli@latest health-check
```

The CLI is fetched via `npx` on each invocation; pin a version with `@<version>` if you want stability. Prefer `@latest` during this skill's experimental window.

## Endpoint reference → [references/cli.md](./references/cli.md)

### Read commands

| Command | Use for |
| --- | --- |
| `query-vaults` | List MetaMorpho / Vault v2 vaults — sortable by APY / TVL |
| `get-vault` | Single vault details (allocation, fees, curator, role list) |
| `query-markets` | List Morpho Blue markets — filter by loan/collateral asset, sort by APY / liquidity / utilization |
| `get-market` | Single market details (LLTV, IRM, oracle, supply / borrow / collateral assets) |
| `get-positions` | All Morpho positions for an address (across vaults + markets) |
| `get-token-balance` | Token balance for a specific (user, token) pair |
| `health-check` | Sanity check the CLI + RPC connection |
| `get-supported-chains` | List supported chain names |

### Write commands (prepare unsigned txs; simulation runs by default)

| Command | Use for |
| --- | --- |
| `prepare-deposit` | Deposit into a vault (V1 or V2) — returns unsigned tx + simulation outcome |
| `prepare-withdraw` | Withdraw from a vault; supports `--amount max` |
| `prepare-supply` | Supply loan asset to a Morpho Blue market |
| `prepare-borrow` | Borrow against existing collateral (warns if health factor < 1.1) |
| `prepare-repay` | Repay borrow; supports `--amount max` |
| `prepare-supply-collateral` | Supply collateral to a market |
| `prepare-withdraw-collateral` | Withdraw collateral; supports `--amount max` |

### Standalone simulation

| Command | Use for |
| --- | --- |
| `simulate-transactions` | Re-simulate with different parameters or simulate arbitrary tx bundles |

## Quick examples

### Find the best USDC vault on Base

```bash
npx @morpho-org/cli@latest query-vaults \
  --chain base \
  --asset-symbol USDC \
  --sort apy_desc \
  --limit 5 \
  --fields address,name,symbol,apyPct,tvlUsd,feePct
```

### Inspect a user's Morpho positions on mainnet

```bash
npx @morpho-org/cli@latest get-positions \
  --chain ethereum \
  --user-address 0xUserAddress
```

### Prepare a vault deposit (with simulation)

```bash
npx @morpho-org/cli@latest prepare-deposit \
  --chain base \
  --vault-address 0xVaultAddress \
  --user-address 0xUserAddress \
  --amount 1000
```

The response includes:
- `summary` — human-readable description
- `transactions` — array of unsigned payloads to sign (includes any required ERC-20 approvals)
- `simulated`, `simulationOk` — was simulation run; did it succeed
- `outcome.vault.{sharesReceived, assetsReceived, positionAssets, positionShares}` — post-execution effect
- `warnings[]` — partial-fill notices (especially for `prepare-withdraw --amount max`)

### Borrow against collateral (with health-factor check)

```bash
npx @morpho-org/cli@latest prepare-borrow \
  --chain base \
  --market-id 0xMarketId \
  --user-address 0xUserAddress \
  --borrow-amount 1
```

The response's `outcome.market` includes `healthFactor`, `isHealthy`, `maxBorrowable`, and before/after `utilizationPct` + `borrowApyPct`. Surface these to the user before they sign.

## Common gotchas

- **`--chain` is required** on every command — no default. Use chain **names** (`ethereum`, `base`), not chain IDs.
- **Amount units**: `--amount` is human-readable (`1000` for 1,000 USDC), not raw units. The CLI handles decimals.
- **Decimal-applied vs. raw values in output**: `TokenAmount` values, `*Pct` fields, and `*Usd` fields are already converted (a USDC value of `"1000"` means $1,000). The **only** raw integer strings are inside `outcome.market.{supplied, borrowed, collateral}` and `outcome.vault.{sharesReceived, assetsReceived, positionShares}` — those need `/10^decimals` for display.
- **Token decimals vary**: USDC/USDT = 6, WBTC/cbBTC = 8, WETH/DAI = 18. Read decimals from response metadata; never assume.
- **Always check simulation before presenting** — `prepare-*` runs simulation by default. Inspect `simulationOk` (`true` / `false`) and `outcome.*.revertReason` if it failed.
- **Health-factor warnings**: borrows with `healthFactor < 1.1` should be flagged to the user. The CLI warns; surface the warning verbatim.
- **Partial withdrawals**: when `prepare-withdraw --amount max` can't withdraw the full balance (vault liquidity shortfall), the response is still valid — it represents the largest withdrawable amount right now. Surface `warnings[]` and offer to wait or accept partial.
- **Vault v1 vs v2 ABIs are incompatible**: when interacting via raw RPC, use `metaMorphoAbi` for v1 (`Vault`) and `vaultV2Abi` for v2 (`VaultV2`). The CLI handles this internally.
- **Singleton contract** for Morpho Blue: `0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb` — same address on Ethereum (chain 1) and Base (chain 8453) via CREATE2.

## Routing back to Alchemy

If during a session the user's need shifts to surfaces this skill doesn't cover (token prices, balances outside Morpho, transaction history, NFT data) or to live RPC / writes / simulation outside Morpho, hand off:

- `alchemy-cli` — live agent work in the current session via the local CLI
- `alchemy-mcp` — live work via the hosted MCP server when CLI is not installed
- `alchemy-api` — application code with an Alchemy API key
- `agentic-gateway` — application code without an API key (x402 / MPP)

For curator / allocator ops, market creation, liquidations, flashloans, or rewards claiming, refer the user to the upstream `@morpho-org/morpho-builder` skill (lower-level SDKs).

---

> **Maintenance:** Morpho maintains the underlying `@morpho-org/cli`; this skill itself is maintained jointly by Alchemy and Morpho. File issues against `alchemyplatform/skills` with `[ecosystem/morpho]` in the title. CLI is pre-v1.0 — schemas may evolve; verify outputs against the live CLI when in doubt.
