# Morpho CLI Reference

**Tool:** `npx @morpho-org/cli@latest <command> [options]`

All commands output JSON to stdout. No private keys involved. Every command requires `--chain` (`ethereum` or `base`).

These commands are non-overlapping with Alchemy first-party. For prices, metadata, balances outside Morpho, transfers, or NFTs, route to `alchemy-api`.

---

## Reads

### `query-vaults`

List MetaMorpho (vault v1) and Vault v2 vaults, sorted by APY or TVL.

```bash
npx @morpho-org/cli@latest query-vaults \
  --chain base \
  [--asset-symbol USDC] \
  [--asset-address 0x...] \
  [--sort apy_desc | apy_asc | tvl_desc | tvl_asc] \
  [--limit 5] [--skip 0] \
  [--fields address,name,symbol,apyPct,tvl,tvlUsd,feePct]
```

### `get-vault`

```bash
npx @morpho-org/cli@latest get-vault --chain base --address 0xVault
```

Returns `address`, `name`, `symbol`, `decimals`, `apyPct`, `tvl`, `tvlUsd`, `feePct`, `curator`, `guardian`, `timelock`, `allocations` (per-market caps), and v2-specific role data when applicable.

### `query-markets`

```bash
npx @morpho-org/cli@latest query-markets \
  --chain base \
  --loan-asset 0x... \
  --collateral-asset 0x... \
  [--sort-by supplyApy | borrowApy | netSupplyApy | netBorrowApy | supplyAssetsUsd | borrowAssetsUsd | totalLiquidityUsd] \
  [--sort-direction asc | desc] \
  [--limit 10] [--skip 0] \
  [--fields supplyApy,borrowApy,totalSupply,totalBorrow,totalCollateral,totalLiquidity,supplyAssetsUsd,borrowAssetsUsd,collateralAssetsUsd,liquidityAssetsUsd]
```

### `get-market`

```bash
npx @morpho-org/cli@latest get-market --chain base --id 0xMarketId
```

Returns market parameters (`loanToken`, `collateralToken`, `oracle`, `irm`, `lltv`) plus current state (supply / borrow / collateral assets, utilization, APYs).

### `get-positions`

```bash
npx @morpho-org/cli@latest get-positions --chain base --user-address 0xUser
```

Returns all Morpho positions for the address — vault deposits and Morpho Blue market positions, with health factors and current values.

### `get-token-balance`

```bash
npx @morpho-org/cli@latest get-token-balance \
  --chain base \
  --user-address 0xUser \
  --token-address 0xToken
```

> **Note:** for general-purpose token balance reads outside Morpho positions, route to `alchemy-api` (Portfolio API) — it covers more chains and tokens.

### `health-check`

```bash
npx @morpho-org/cli@latest health-check
```

Sanity check the CLI install + RPC connectivity.

### `get-supported-chains`

```bash
npx @morpho-org/cli@latest get-supported-chains
```

Currently `ethereum` and `base`. Other chains may be added; query at runtime rather than hardcoding.

---

## Writes (prepare unsigned transactions)

All `prepare-*` commands return a flat `PreparedOperation` object with the same root keys:

- `operation` — operation type (e.g., `deposit`, `withdraw`)
- `summary` — human-readable description
- `requirements` — informational; approvals are already in `transactions`
- `transactions` — array of unsigned payloads to sign + submit (includes ERC-20 approvals if needed)
- `simulated` — whether simulation was run (default `true`)
- `simulationOk` — `true` / `false` if simulated
- `totalGasUsed` — total gas (only if `simulated: true`)
- `outcome` — discriminated by operation type (see below)
- `warnings[]` — partial-fill notices, health-factor warnings

### `outcome` shape by operation

| Operation | `outcome` shape | Key fields |
| --- | --- | --- |
| `deposit`, `withdraw` (vaults) | `outcome.vault` | `sharesReceived`, `assetsReceived`, `positionAssets`, `positionShares` |
| `supply`, `borrow`, `repay`, `supply_collateral`, `withdraw_collateral` (markets) | `outcome.market` | `healthFactor`, `isHealthy`, `maxBorrowable`, `utilizationBeforePct` → `utilizationAfterPct`, `borrowApyBeforePct` → `borrowApyAfterPct`, plus post-op `supplied` / `borrowed` / `collateral` (raw integer strings — divide by `10^decimals`) |

### Vault operations

```bash
# Deposit into a vault
npx @morpho-org/cli@latest prepare-deposit \
  --chain base --vault-address 0xVault --user-address 0xUser --amount 1000

# Withdraw from a vault — supports max
npx @morpho-org/cli@latest prepare-withdraw \
  --chain base --vault-address 0xVault --user-address 0xUser --amount max
```

### Morpho Blue market operations

```bash
# Supply loan asset to a market
npx @morpho-org/cli@latest prepare-supply \
  --chain base --market-id 0xMarket --user-address 0xUser --amount 5000

# Borrow against existing collateral (warns if HF < 1.1)
npx @morpho-org/cli@latest prepare-borrow \
  --chain base --market-id 0xMarket --user-address 0xUser --borrow-amount 1

# Repay
npx @morpho-org/cli@latest prepare-repay \
  --chain base --market-id 0xMarket --user-address 0xUser --amount max

# Supply collateral
npx @morpho-org/cli@latest prepare-supply-collateral \
  --chain base --market-id 0xMarket --user-address 0xUser --amount 5000

# Withdraw collateral
npx @morpho-org/cli@latest prepare-withdraw-collateral \
  --chain base --market-id 0xMarket --user-address 0xUser --amount max
```

### Skipping simulation

Add `--no-simulate` to skip simulation. The response will lack `simulationOk`, `totalGasUsed`, and most of `outcome`. Only skip when debugging or when speed is critical.

---

## Standalone simulation

```bash
npx @morpho-org/cli@latest simulate-transactions \
  --chain base \
  --from 0xUser \
  --transactions '<JSON array of {to, data, value} payloads>' \
  --analysis-context '<JSON describing what you want analyzed>'
```

Returns `allSucceeded` (note: **not** `simulationOk` like the `prepare-*` commands), per-tx revert reasons, and whatever analysis the context requested.

Use this when you need to re-simulate with different parameters, or simulate arbitrary tx bundles outside the `prepare-*` flow.

---

## Common simulation failures

| Revert | Cause | What to do |
| --- | --- | --- |
| `ERC20: insufficient allowance` | Missing approval | Re-prepare — CLI should include approvals automatically; if it didn't, file a bug |
| `ERC4626ExceededMaxWithdraw` | Vault liquidity insufficient | Reduce `--amount` (or accept the partial returned with `warnings[]`) |
| `insufficient balance` | User lacks tokens | Tell the user; route to `alchemy-api` Portfolio API to confirm balance |
| Custom error hex | Protocol-specific revert | Query state with `get-market` / `get-vault` to diagnose |

---

## Partial withdrawals

`prepare-withdraw --amount max` may return less than the full position when vault liquidity is locked elsewhere. The response is still valid — it represents the largest amount withdrawable right now. The CLI surfaces this in `warnings[]`.

When this happens:

1. Surface the warning **verbatim** to the user — do not silently accept a smaller withdrawal.
2. Offer two paths: (a) accept the partial amount (use `outcome.vault.assetsReceived` for the concrete figure, or re-run with `--amount <value>` at ~99% of that figure as a buffer against interest accrual), or (b) wait for more liquidity to unlock.
3. Never invent an amount by parsing `summary` — that's a human sentence, not a machine-readable field.

---

## Out of scope here

Curator / allocator / owner ops on vaults (`setCap`, `reallocate`, role management, timelocks), Morpho Blue **market creation**, **liquidations**, **flashloans**, **pre-liquidation**, and **rewards claiming** are intentionally not exposed by the CLI. Use the upstream `@morpho-org/morpho-builder` skill (lower-level SDKs) for those flows.
