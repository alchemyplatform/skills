# MoonPay CLI Reference

**Tool:** `mp <command>`

Four surfaces, one CLI. These flows are non-overlapping with Alchemy first-party. For token swaps, prices, balances, transfers, or NFTs, route to `alchemy-api` or `lifi`.

---

## `mp buy` — fiat → crypto checkout

Generate a MoonPay checkout URL for buying crypto with a credit card or bank transfer. The user completes the purchase in their browser.

```bash
mp buy \
  --token <currency-code> \
  --amount <usd-amount> \
  --wallet <destination-address> \
  --email <buyer-email>
```

### Supported currency codes

`btc`, `sol`, `eth`, `trx`, `pol_polygon`, `usdc`, `usdc_sol`, `usdc_base`, `usdc_arbitrum`, `usdc_optimism`, `usdc_polygon`, `usdt_trx`, `eth_polygon`, `eth_optimism`, `eth_base`, `eth_arbitrum`

### Notes

- `--amount` is **in USD** (e.g., `50` = $50 worth of the token).
- `--token` uses MoonPay currency codes — not contract addresses, not Solana mints.
- The returned checkout URL handles KYC + payment processing client-side. The user opens it in a browser to complete.
- For token-to-token swaps (no fiat), route to the `lifi` ecosystem skill.

---

## `mp virtual-account *` — fiat on/off-ramp infrastructure

Comprehensive virtual account for converting between fiat (USD / EUR) and stablecoins (USDC / USDT / EURC) on Solana, Ethereum, Polygon, Base, Arbitrum.

### Setup flow

```bash
# 1. Create the account + start KYC (returns a verification URL)
mp virtual-account create

# 2. Check status — `status` field shows where you are; `nextStep` says what to do next
mp virtual-account retrieve

# 3. KYC verification
mp virtual-account kyc continue   # check status / re-fetch URL
mp virtual-account kyc restart    # restart if something went wrong
```

### Agreements (require user consent)

```bash
# List required agreements — shows name + URL for each
mp virtual-account agreement list

# Show the URL to the user. Wait for explicit confirmation they've read it.
# Then:
mp virtual-account agreement accept --contentId <content-id>

# View previously accepted
mp virtual-account agreement list --status accepted
```

> **Important:** never accept agreements on the user's behalf. The CLI requires explicit consent because these are legal terms.

### Wallet registration

```bash
# Creates verification message, signs locally, registers — one command
mp virtual-account wallet register --wallet main --chain solana

mp virtual-account wallet list
```

Supported chains: `solana`, `ethereum`, `polygon`, `base`, `arbitrum`.

### Onramp (fiat → stablecoin)

```bash
# Create — returns deposit account (bank IBAN / account number)
mp virtual-account onramp create \
  --name "My Onramp" \
  --fiat USD \
  --stablecoin USDC \
  --wallet <registered-wallet-address> \
  --chain solana

mp virtual-account onramp retrieve --onrampId <id>   # includes deposit account, fees, legal disclaimer
mp virtual-account onramp list
mp virtual-account onramp update --onrampId <id> --chain ethereum
mp virtual-account onramp cancel --onrampId <id>

# Open-banking payment
mp virtual-account onramp payment create \
  --onrampId <id> --amount 100 --fiat USD

mp virtual-account onramp payment retrieve --onrampId <id> --paymentId <payment-id>
```

### Bank-account registration (for offramps)

```bash
# Register a USD bank account (ACH)
mp virtual-account bank-account register \
  --currency USD --type ACH \
  --accountNumber <number> --routingNumber <number> \
  --providerName "Chase" --providerCountry US \
  --givenName John --familyName Doe \
  --email john@example.com --phoneNumber +14155551234 \
  --address.street "123 Main St" --address.city "New York" \
  --address.state NY --address.country US --address.postalCode 10001

mp virtual-account bank-account list
mp virtual-account bank-account delete --bankAccountId <id>
```

### Offramp (stablecoin → fiat)

```bash
mp virtual-account offramp create \
  --name "My Offramp" \
  --bankAccountId <bank-account-id> \
  --stablecoin USDC \
  --chain solana

mp virtual-account offramp list
mp virtual-account offramp retrieve --offrampId <id>
mp virtual-account offramp update --offrampId <id> --fiat EUR
mp virtual-account offramp cancel --offrampId <id>

# Send stablecoin to an approved offramp (signs + broadcasts locally)
mp virtual-account offramp initiate \
  --wallet main --offrampId <id> --amount 100
```

### Other

```bash
mp virtual-account transaction list
```

---

## `mp deposit *` — permissionless multi-chain deposit links

Generate a deposit link that exposes addresses on Solana, Ethereum, Bitcoin, and Tron. Anyone can send any token; Helio auto-converts to the chosen stablecoin and settles to the destination wallet on the chosen chain.

No login required.

```bash
mp deposit create \
  --name <label> \
  --wallet <destination-address> \
  --chain <destination-chain> \
  --token <stablecoin>
```

### Supported destination chains

`solana`, `ethereum`, `base`, `polygon`, `arbitrum`, `bnb`

### Supported tokens

- `USDC` — all chains
- `USDT` — all chains
- `USDC.e` — bridged USDC, Polygon only

### How it works

1. `mp deposit create` returns a deposit URL + per-chain addresses + QR codes.
2. Sender pays from Solana / Ethereum / Bitcoin / Tron — any token.
3. Helio auto-converts to the chosen stablecoin and delivers to the destination wallet.
4. `mp deposit transaction list --id <deposit-id>` shows incoming payments.

### Notes

- No login or account required — deposits are permissionless.
- Each `deposit create` generates unique addresses — don't reuse addresses from different deposits.
- The deposit URL opens a web page where senders pick how to pay.

---

## `mp commerce *` — Solana Pay Shopify checkout

Browse Solana Pay-enabled Shopify stores, manage a cart, and pay with crypto (USDC) via Helio. Entire flow runs from the CLI; no browser needed.

```bash
# 1. Browse
mp commerce store list

# 2. Search for products
mp commerce product search --store <store> --query <search-term>
mp commerce product retrieve --store <store> --productId <product-id>

# 3. Cart management
mp commerce cart add \
  --store <store> --variantId <variant-id> --quantity <n> \
  --cartId <cart-id>           # omit to create a new cart

mp commerce cart retrieve --store <store> --cartId <cart-id>
mp commerce cart remove   --store <store> --cartId <cart-id> --lineId <line-id>

# 4. Checkout (signs + submits Solana Pay tx)
mp commerce checkout \
  --wallet <wallet-name> \
  --store <store> --cartId <cart-id> --chain solana \
  --email <buyer-email> \
  --firstName <first> --lastName <last> \
  --address <street-address> --city <city> --postalCode <zip> --country <country-name>
```

### How it works

1. Stores expose a Shopify MCP endpoint for browsing + cart.
2. `cart add` creates or updates a cart (no auth needed; cart ID is the handle).
3. `checkout` calls the API to start a Helio payment, signs the tx locally, submits.
4. Helio pays gas — the buyer only pays the item price in USDC.
5. Returns a tx signature + order confirmation URL.

### Notes

- `--country` takes full country names (e.g., "United States", "Netherlands").
- Checkout takes 30-90 seconds — the API automates the Shopify checkout flow.
- Currently Solana-only for payment, even on multi-chain stores.

---

## Out of scope here (use upstream `moonpay/skills` directly if you need them)

The upstream `moonpay/skills` repo includes ~30+ skills. The non-overlapping subset we mirror is the four CLI surfaces above. The rest are out of scope for this curated skill because they overlap with Alchemy first-party or other ecosystem skills:

| Upstream skill | Why scope_out | Use |
| --- | --- | --- |
| `moonpay-swap-tokens` | Overlaps with `lifi` (broader DEX + bridge coverage) | `lifi` ecosystem skill |
| `moonpay-trading-automation` | Overlaps with multiple swap / trading surfaces | `lifi` + `alchemy-api` |
| `moonpay-auth` | Overlaps with Alchemy Wallets / Account Kit | `alchemy-api` |
| `moonpay-x402` | Overlaps with our `agentic-gateway` (already does x402) | `agentic-gateway` |
| `moonpay-discover-tokens` | Partial overlap with Token API | `alchemy-api` (Token API) |
| `moonpay-block-explorer` | Overlaps with general blockchain reads | `alchemy-api` (JSON-RPC) |
| `moonpay-check-wallet` | Overlaps with Portfolio API | `alchemy-api` (Portfolio API) |
| `moonpay-price-alerts` | Partial overlap with Prices API | `alchemy-api` (Prices API) |
| `moonpay-hardware-wallet` | Wallet territory | `alchemy-api` (Wallets) |
| `moonpay-prediction-market`, `moonpay-fund-polymarket` | Niche, prediction-market specific | upstream directly if needed |
| `moonpay-mcp`, `moonpay-export-data`, `moonpay-feedback`, `moonpay-missions`, `moonpay-scout`, `moonpay-upgrade`, `moonpay-wallet-statusline*` | Internal product features or unrelated tooling | upstream directly if needed |

To install the full MoonPay surface alongside this curated subset:

```bash
npx skills add moonpay/skills
```
