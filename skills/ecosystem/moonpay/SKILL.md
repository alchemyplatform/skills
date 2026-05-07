---
name: moonpay
description: Buy crypto with fiat, set up fiat on/off-ramp virtual accounts (KYC + USD/EUR ↔ USDC/USDT/EURC on Solana, Ethereum, Polygon, Base, Arbitrum), accept crypto via multi-chain deposit links that auto-convert to stablecoins, swap and bridge tokens (same-chain + cross-chain via swaps.xyz), shop on Solana Pay Shopify stores, and automate DCA / limit-order / stop-loss strategies via cron or launchd — all via the `mp` CLI. NOT for token metadata, balances, transaction history, NFT data, or live RPC reads — for those use `alchemy-cli` (live), `alchemy-mcp`, `alchemy-api` (app code), or `agentic-gateway` (no API key).
license: MIT
compatibility: Requires the MoonPay `mp` CLI (install per MoonPay docs). No Alchemy API key needed — reads + payment flows go through MoonPay's hosted infrastructure. KYC and user verification handled by MoonPay's checkout / virtual-account flows. Network access required.
metadata:
  author: moonpay
  version: "0.1"
  provider: moonpay
  partner: "true"
---

# MoonPay (Fiat Ramps + Payments + Swap/Bridge + Automation)

MoonPay provides fiat-to-crypto on-ramps, crypto-to-fiat off-ramps, multi-chain crypto deposit infrastructure, Solana Pay checkout for Shopify stores, same-chain swaps + cross-chain bridges (via swaps.xyz), and trading automation (DCA / limit orders / stop losses via cron or launchd). This skill covers all six surfaces via the official `mp` CLI. For balances, prices, transaction history, NFT data, or live RPC, use the corresponding Alchemy skill instead.

| | |
| --- | --- |
| **Tool** | `mp <command>` (MoonPay CLI) |
| **Auth** | KYC handled by MoonPay's hosted flows; CLI uses local wallet for signing where required |
| **Chains (stablecoin settlement)** | Solana, Ethereum, Polygon, Base, Arbitrum, BNB |
| **Stablecoins** | USDC, USDT, EURC (varies by chain) |
| **Reciprocity** | MoonPay already mirrors `alchemy-api` + `alchemy-agentic-gateway` in [moonpay/skills](https://github.com/moonpay/skills) |

## When to use this skill

Use `moonpay` when **any** of the following are true:

- The user wants to **buy crypto with fiat** (credit card / bank → BTC / SOL / ETH / USDC / etc.)
- The user wants to **set up a fiat on/off-ramp virtual account** (USD or EUR ↔ stablecoins, with KYC + bank-account registration)
- The user wants to **accept crypto payments** via multi-chain deposit links that auto-convert to stablecoins
- The user wants to **pay for goods on a Solana Pay-enabled Shopify store** with crypto
- The user wants to **swap or bridge tokens** (same-chain or cross-chain) via the MoonPay CLI
- The user wants to **automate trading** (DCA, limit order, stop loss) via cron / launchd

## When NOT to use this skill (handoff)

| Need | Use instead |
| --- | --- |
| Token prices (current, historical) | `alchemy-api` (Prices API) |
| Token metadata, search, list by chain | `alchemy-api` (Token API) |
| Current wallet balances | `alchemy-api` (Portfolio / Token API) |
| Transaction history (transfers in / out) | `alchemy-api` (Transfers API) |
| NFT metadata / floor / ownership | `alchemy-api` (NFT API) |
| Live blockchain reads (block #, gas, `eth_call`) | `alchemy-cli` (live) or `alchemy-api` (JSON-RPC) |
| Prediction-market integration | not in scope here (see upstream `moonpay-prediction-market` if needed) |
| Account abstraction (bundlers, gas managers) | `alchemy-api` (Wallets / Bundler / Gas Manager) |
| x402 / MPP (autonomous-agent payments) | `agentic-gateway` |
| Wallet auth / hardware wallet flows | `alchemy-api` (Wallets / Account Kit) |

## Scope contract

**This skill covers (`scope_in`):**

- **Buy crypto with fiat** (`mp buy`) — credit card / bank → crypto via MoonPay checkout URL. Supports BTC, SOL, ETH, USDC (multi-chain), USDT, MATIC, TRX.
- **Virtual account** (`mp virtual-account *`) — full fiat on/off-ramp: KYC, agreement acceptance, wallet registration, bank-account registration (ACH / SEPA / etc.), onramp creation (fiat → stablecoin), offramp creation (stablecoin → fiat), payment / settlement flows.
- **Multi-chain deposits** (`mp deposit *`) — permissionless deposit links that generate addresses on Solana / Ethereum / Bitcoin / Tron and auto-convert incoming crypto to a chosen stablecoin (USDC / USDT) on the destination chain.
- **Commerce** (`mp commerce *`) — browse Solana Pay-enabled Shopify stores, search products, manage cart, checkout with crypto (USDC) via Helio integration.
- **Swap & bridge** (`mp token swap` / `mp token bridge`) — same-chain swaps + cross-chain bridges via swaps.xyz across Solana, Ethereum, Base, Polygon, Arbitrum, Optimism, BNB, Avalanche, Bitcoin (bridges only). Auto-handles ERC-20 approvals; signs locally and broadcasts.
- **Trading automation** — shell-script + cron / launchd patterns composing `mp token swap` and `mp token search` for DCA, limit orders, and stop losses. Self-disabling for one-shot triggers (limit / stop). Logs to `~/.config/moonpay/logs/trading.log`.

**This skill does NOT cover (`scope_out`):**

- Token prices for valuation → handoff: `alchemy-api` (Prices API)
- Token metadata, list, search → handoff: `alchemy-api` (Token API). Not the upstream `moonpay-discover-tokens`.
- Wallet balances → handoff: `alchemy-api` (Portfolio / Token API). Not the upstream `moonpay-check-wallet`.
- Transaction transfer history → handoff: `alchemy-api` (Transfers API)
- NFT data → handoff: `alchemy-api` (NFT API)
- Live RPC / block-explorer reads → handoff: `alchemy-cli` or `alchemy-api`. Not the upstream `moonpay-block-explorer`.
- Wallet auth, hardware wallet integration → handoff: `alchemy-api` (Wallets / Account Kit). Not the upstream `moonpay-auth` / `moonpay-hardware-wallet`.
- x402 / autonomous-agent payments → handoff: `agentic-gateway`. Not the upstream `moonpay-x402`.
- Account abstraction → handoff: `alchemy-api`
- Prediction markets, price alerts, internal CLI features (mcp, scout, missions, statusline, feedback, upgrade, export-data) → out of scope here. Install `moonpay/skills` upstream alongside if you need them.

## Setup

Install the MoonPay `mp` CLI per [MoonPay's docs](https://docs.moonpay.com/). The CLI handles auth, wallet signing, and KYC URLs internally — there's no Alchemy API key required.

For **virtual account** flows (the most involved surface), KYC needs to be completed via the URL the CLI returns. Walk the user through it; don't accept agreements on their behalf.

> **Security:** when a `virtual-account agreement list` step surfaces an agreement URL, **always show the URL to the user and confirm they've reviewed it** before running `agreement accept`. Don't auto-accept on their behalf.

## Endpoint reference → [references/cli.md](./references/cli.md)

### `mp buy` — fiat → crypto checkout

| Command | Use for |
| --- | --- |
| `mp buy --token <code> --amount <usd> --wallet <addr> --email <buyer>` | Returns a MoonPay checkout URL the user opens in a browser to complete a card / bank purchase. |

### `mp virtual-account *` — fiat on/off-ramp

| Command | Use for |
| --- | --- |
| `mp virtual-account create` | Create the virtual account + start KYC |
| `mp virtual-account retrieve` | Check status (`status` + `nextStep`) |
| `mp virtual-account kyc {continue,restart}` | KYC flow control |
| `mp virtual-account agreement {list,accept}` | Required-agreements review and accept |
| `mp virtual-account wallet {register,list}` | Register a wallet for the account |
| `mp virtual-account onramp {create,retrieve,list,update,cancel}` | Fiat → stablecoin (returns deposit account / IBAN) |
| `mp virtual-account onramp payment {create,retrieve}` | Open-banking payment flow |
| `mp virtual-account bank-account {register,list,delete}` | Register fiat payout bank accounts |
| `mp virtual-account offramp {create,retrieve,list,update,cancel,initiate}` | Stablecoin → fiat |
| `mp virtual-account transaction list` | List transactions |

### `mp deposit *` — permissionless deposit links

| Command | Use for |
| --- | --- |
| `mp deposit create --name <label> --wallet <addr> --chain <chain> --token <stablecoin>` | Create a deposit link with multi-chain receive addresses; auto-converts to chosen stablecoin |
| `mp deposit transaction list --id <deposit-id>` | List incoming deposits |

Supported destination chains: `solana`, `ethereum`, `base`, `polygon`, `arbitrum`, `bnb`. Stablecoins: `USDC`, `USDT`, `USDC.e` (Polygon-bridged).

### `mp commerce *` — Shopify checkout via Solana Pay

| Command | Use for |
| --- | --- |
| `mp commerce store list` | List Solana Pay-enabled stores |
| `mp commerce product search --store <s> --query <q>` | Search products |
| `mp commerce product retrieve --store <s> --productId <id>` | Product details |
| `mp commerce cart {add,retrieve,remove}` | Cart management |
| `mp commerce checkout ...` | Sign + submit Solana Pay payment via Helio (gas paid by Helio) |

### `mp token *` — same-chain swap + cross-chain bridge

| Command | Use for |
| --- | --- |
| `mp token swap --wallet <w> --chain <c> --from-token <addr> --from-amount <n> --to-token <addr>` | Same-chain swap via swaps.xyz; auto-approves ERC-20 if needed. Supports `--to-amount` for exact-out. |
| `mp token bridge --from-wallet <w> --from-chain <c> --from-token <addr> --from-amount <n> --to-chain <c> --to-token <addr>` | Cross-chain bridge via swaps.xyz; signs + broadcasts locally. |
| `mp token search --query <q> --chain <c>` | Resolve token names → addresses (use before swap/bridge if user provides symbols). |
| `mp token balance list --wallet <addr> --chain <c>` | Check balances pre-swap. |

Supported chains: `solana`, `ethereum`, `base`, `polygon`, `arbitrum`, `optimism`, `bnb`, `avalanche`, `bitcoin` (bridges only).

### Trading automation — `mp` + cron / launchd

Generates shell scripts at `~/.config/moonpay/scripts/` that compose `mp token swap` (and `mp token search` for price reads) on a schedule. Logs to `~/.config/moonpay/logs/trading.log`.

| Strategy | Script pattern |
| --- | --- |
| **DCA** | Run `mp token swap` on cron / launchd at fixed intervals (e.g., `0 9 * * *` for daily). |
| **Limit order** | Poll price every 5 min via `mp token search`; execute swap when below target; self-disable after fill. |
| **Stop loss** | Poll price every 5 min; query balance via `mp token balance list`; sell entire balance when below trigger; self-disable after fill. |

See `references/cli.md` for full script templates (cron + launchd plist) and the self-disable patterns.

## Quick examples

### Buy $50 of SOL with a credit card

```bash
mp buy \
  --token sol \
  --amount 50 \
  --wallet 9WzDXwBbmkg8ZTbNMqUxvQRAyrZzDsGYdLVL9zYtAWWM \
  --email user@example.com
# Returns a checkout URL — open in the user's browser
```

### Set up a USD → USDC onramp on Base

```bash
mp virtual-account create                    # starts KYC; surfaces a verification URL
mp virtual-account agreement list            # show URLs to the user, get their OK
mp virtual-account agreement accept --contentId <id>
mp virtual-account wallet register --wallet main --chain base
mp virtual-account onramp create \
  --name "USD → USDC Base" --fiat USD --stablecoin USDC \
  --wallet <registered-address> --chain base
# Returns a fiat deposit account (e.g., a US bank IBAN) — wire / ACH funds in;
# they settle as USDC on Base to the registered wallet.
```

### Accept incoming crypto payments as USDC on Base

```bash
mp deposit create \
  --name "My Payments" \
  --wallet 0xf1D8...5036 \
  --chain base \
  --token USDC
# Returns a shareable deposit URL + per-chain addresses + QR codes.
# Senders can pay from Solana / Ethereum / Bitcoin / Tron;
# auto-converts to USDC on Base.
```

### Buy something from a Solana Pay Shopify store with USDC

```bash
mp commerce store list
mp commerce product search --store ryder.id --query "ryder"
mp commerce cart add --store ryder.id --variantId "gid://shopify/ProductVariant/..." --quantity 1
mp commerce checkout \
  --wallet main --store ryder.id --cartId <id> --chain solana \
  --email buyer@example.com --firstName John --lastName Doe \
  --address "123 Main St" --city Amsterdam --postalCode 1011 --country Netherlands
```

### Same-chain swap (SOL → USDC on Solana)

```bash
mp token swap \
  --wallet main --chain solana \
  --from-token So11111111111111111111111111111111111111111 \
  --from-amount 0.1 \
  --to-token EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
```

### Cross-chain bridge (ETH on Ethereum → USDC.e on Polygon)

```bash
mp token bridge \
  --from-wallet funded --from-chain ethereum \
  --from-token 0x0000000000000000000000000000000000000000 \
  --from-amount 0.003 \
  --to-chain polygon \
  --to-token 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174
```

### DCA — buy $5 of SOL daily at 9am

Generate a script at `~/.config/moonpay/scripts/dca-sol.sh` that calls `mp token swap`, then schedule with cron (`0 9 * * *`) or launchd. Full pattern (incl. logging + macOS launchd plist) in [references/cli.md](./references/cli.md).

## Common gotchas

- **`--token` uses MoonPay codes**, not contract addresses or mint addresses (e.g., `usdc_base`, not the contract). See `references/cli.md` for the supported token list.
- **`--amount` for `mp buy` is in USD**, not in token units (`--amount 50` = $50 worth of the token, not 50 SOL).
- **KYC is mandatory** for the virtual account flow. Surface the verification URL to the user; don't try to bypass.
- **Agreements need explicit user consent** — list URLs, get user confirmation they've read them, then accept. Don't auto-accept.
- **Commerce is Solana-only** for payment today, even if the store is multi-chain. The buyer pays in USDC on Solana.
- **Deposit destination chains** are `solana / ethereum / base / polygon / arbitrum / bnb`; sender chains are `solana / ethereum / bitcoin / tron`.
- **Helio infrastructure** powers the deposit + commerce flows under the hood. The buyer doesn't pay gas for commerce checkouts (Helio sponsors).
- **Native-token addresses for swap/bridge**: EVM chains use `0x0000000000000000000000000000000000000000`; Solana uses `So11111111111111111111111111111111111111111`.
- **swaps.xyz** is the routing engine behind `mp token swap` / `bridge`. ERC-20 approvals are sent automatically as a separate tx before the swap — expect 2 transactions for first-time approvals.
- **Trading automation requires the user session to be active** for OS keychain access. Don't schedule trades on a machine that locks aggressively. macOS launchd fires even after sleep; cron does not.

## Routing back to Alchemy

If during a session the user's need shifts to surfaces this skill doesn't cover (balances, prices, transaction history, NFT data, live RPC), hand off:

- `alchemy-cli` — live agent work in the current session via the local CLI
- `alchemy-mcp` — live work via the hosted MCP server when CLI is not installed
- `alchemy-api` — application code with an Alchemy API key
- `agentic-gateway` — application code without an API key (x402 / MPP)

---

> **Maintenance:** MoonPay maintains the `mp` CLI and the underlying APIs; this skill itself is maintained jointly by Alchemy and MoonPay. File issues against `alchemyplatform/skills` with `[ecosystem/moonpay]` in the title. Reciprocity: MoonPay already mirrors `alchemy-api` and `alchemy-agentic-gateway` inside [`moonpay/skills`](https://github.com/moonpay/skills).
