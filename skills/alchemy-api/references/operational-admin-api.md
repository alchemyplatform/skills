---
id: references/operational-admin-api.md
name: 'Admin API'
description: 'The Alchemy Admin API is the programmatic surface for account and app operations: managing apps, listing available chains, and reading usage/metering data. Public REST endpoints on the unified api.g.alchemy.com gateway with Bearer-token auth.'
tags:
  - alchemy
  - operational
  - admin
  - operations
related:
  - operational-auth-and-keys.md
  - operational-supported-networks.md
updated: 2026-08-24
---
# Admin API

The **Admin API** is Alchemy's programmatic surface for account, app, and usage operations. Distinct from the RPC/Data APIs which are keyed off `<network>.g.alchemy.com/v2/<key>`, the Admin API lives on the unified gateway:

**Base URL**: `https://api.g.alchemy.com/admin-api/v1`

**Auth**: `Authorization: Bearer <admin-token>` (browser-session-derived, or via the CLI's `alchemy auth login` path). This is NOT the same as your RPC API key. The URL-embedded key pattern (`.../v2/<key>`) does NOT work here.

The public Admin API surface (as of 2026-08) is grouped into three product areas:

- **Apps** — enumerate, create, update, and manage Alchemy apps and their allowlists.
- **Chains** — read the canonical list of chains Alchemy supports and their per-chain metadata (used by `apps create` / `apps update` to name allowed networks).
- **Usage** — read compute-unit consumption per app, network, product, and time range (billable data-plane metrics).

Apps and Chains are documented and self-serve as of 2026-08. Usage is also self-serve for app-scoped queries.

## When to use this vs the CLI

- Live agent work (browse apps, tweak allowlists, look up usage in a session) → use the `alchemy-cli` skill (`alchemy app *`, `alchemy usage *`). The CLI wraps this API cleanly and handles auth for you.
- App code that manages Alchemy resources from your own backend (multi-tenant SaaS creating apps per customer, dashboards visualizing usage, CI/CD provisioning) → hit the REST endpoints directly.

## Apps

Path prefix: `/admin-api/v1/apps`.

| Endpoint | Method | What it does |
|---|---|---|
| `/apps` | `GET` | List all apps on the account. Query params: `cursor`, `limit`, `search`, `id`. |
| `/apps` | `POST` | Create an app. Body: `{ "name", "description", "networks": ["eth-mainnet", ...], "products": [...] }`. |
| `/apps/{appId}` | `GET` | Read a single app's config: name, description, allowed networks, allowlists (address / origin / IP), products, keys. |
| `/apps/{appId}` | `PATCH` | Update app metadata (name, description). |
| `/apps/{appId}` | `DELETE` | Delete an app. |
| `/apps/{appId}/networks` | `PATCH` | Set the app's allowed networks (replaces the full list). |
| `/apps/{appId}/allowlists/addresses` | `PATCH` | Set the app's address allowlist. |
| `/apps/{appId}/allowlists/origins` | `PATCH` | Set the app's origin allowlist. |
| `/apps/{appId}/allowlists/ips` | `PATCH` | Set the app's IP allowlist. |
| `/apps/{appId}/configured-networks` | `GET` | List which networks are actually configured on this app (subset of the account's allowed networks). |

Network slugs in the `networks` array come from the Chains endpoint (see below), NOT from the per-chain RPC URL pattern.

## Chains

Path prefix: `/admin-api/v1/chains`.

| Endpoint | Method | What it does |
|---|---|---|
| `/chains` | `GET` | List every chain/network Alchemy supports, keyed by Admin API chain identifier. Includes the mapping between the human-readable name (`Ethereum Mainnet`) and the slug used in `apps create`/`apps update` (`eth-mainnet`). |
| `/chains/{chainId}` | `GET` | Read a single chain's metadata. |

The Admin API chain identifiers (e.g., `eth-mainnet`, `base-mainnet`, `polygon-mainnet`, `solana-mainnet`, `bitcoin-mainnet`, `stellar-mainnet`) also match the RPC URL slugs. Full canonical list changes as new networks launch — treat this endpoint as source of truth.

## Usage

Path prefix: `/admin-api/v1/usage`.

| Endpoint | Method | What it does |
|---|---|---|
| `/usage/summary` | `GET` | Aggregate usage over a date range. Accepts `startDate`, `endDate`, optional `appIds[]`, `networks[]`, `products[]`, `metrics=amount,usd`. |
| `/usage/timeseries` | `GET` | Time-bucketed usage across a range. Accepts the same filters as `/summary` plus `granularity=minute\|hour\|day\|month`, `groupBy`, `methods[]`, `requestTypes[]`. |

### Range limits on `timeseries`

| Plan | Max lookback |
|---|---|
| Free | 7 days |
| Pay as You Go | 30 days |
| Longer | Contact sales |

**Behavior when range exceeds the limit**: the server returns the allowed range without erroring. The response payload includes both the requested range and the served range so clients can detect truncation. Do NOT rely on receiving an HTTP error when overshooting.

### `metrics` semantics (2026-08 change)

- `/usage/summary` accepts both `amount` (CU count) and `usd` (dollar-equivalent), comma-separated.
- `/usage/timeseries` accepts **only** `amount`. The `usd` metric was removed from the timeseries surface in 2026-08 — passing it will fail or be silently dropped. Aggregate `amount` client-side and multiply by the per-product rate if you need USD equivalents on a timeseries.

### Time-series products

The `products[]` filter accepts these product identifiers (each is metered separately with distinct CU rates):

| Product ID | Meter | Note |
|---|---|---|
| `SUPERNODE_CU` | Compute Units | Standard RPC / Data traffic. |
| `SUPERNODE_ENHANCED_CU` | Enhanced CU | Chain-specific enhanced methods. |
| `NOTIFY_CU` | Notify CU | Webhooks. |
| `BUNDLER_CU` | Bundler CU | Wallet APIs bundler ops. |
| `PAYMASTER_CU` | Paymaster CU | Gas Manager. |
| `SUBSCRIPTION_BYTES` | Bytes | WebSocket subscription bandwidth (except Flashblocks per-event pricing). |

The Admin API overview page in the docs lists the full canonical table; refer there for updates.

## Common Errors

| Status / Code | Cause | Fix |
|---|---|---|
| `401 Unauthorized` | Missing or invalid Bearer token | Re-run `alchemy auth login` or provide a fresh token. |
| `403 Forbidden` | Account lacks admin permission on this resource | Only account admins can hit these endpoints; regular members do not have access. |
| `404 Not Found` | Wrong `appId`, wrong slug, or the resource doesn't exist on this account | Verify via `GET /apps` or `GET /chains`. |
| `422 Unprocessable Entity` | Body validation failed | Read the response `errors[]` for field-level messages; common cause is an unknown network slug or a malformed allowlist entry. |

## API Key Rotation

The API key on an app can be rotated from the dashboard: **Settings → Rotate API key**. The rotation UI is admin-only.

- After rotation, the **previous key remains valid for 2 minutes** — this is the grace window for rolling deploys and cache warmups.
- Post-cutoff, requests with the old key return `401 Invalid API Key`.
- Rotation does NOT reset app configuration (networks, allowlists, products); it only replaces the key material.
- Verify the new key with `alchemy config get api-key` (via the CLI) or an authenticated dashboard reveal.

## Notes

- Keep Bearer tokens out of client-side code — this is a server / dashboard / CLI surface, not a public-key surface.
- Admin API traffic does NOT count against your app's compute-unit quota; it's separately quota'd at the account level (rate-limited but not billable).
- The Admin API URLs are stable — `/docs/admin-api/apps/...`, `/docs/admin-api/chains/...`, `/docs/admin-api/usage/...` on the docs site.

## Other ways to access this API

- **Live agent work via CLI** (`alchemy-cli` skill, when `@alchemy/cli` is installed):
  - `alchemy app list`, `alchemy app get <id>`, `alchemy app create ...`, `alchemy app update ...`, `alchemy app configured-networks`, `alchemy app chains`, `alchemy app select <id>`
  - `alchemy usage summary`, `alchemy usage timeseries`
- **Live agent work via MCP** (`alchemy-mcp` skill, when MCP is wired in but CLI isn't): the `select_app`, `list_apps`, `create_app`, etc. tools call these endpoints under the hood.

## Official Docs
- [Admin API — Apps](https://www.alchemy.com/docs/admin-api/apps)
- [Admin API — Chains](https://www.alchemy.com/docs/admin-api/chains)
- [Admin API — Usage](https://www.alchemy.com/docs/admin-api/usage)
