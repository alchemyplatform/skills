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

## `mp token *` — same-chain swap + cross-chain bridge

Same-chain swaps and cross-chain bridges via [swaps.xyz](https://swaps.xyz). Builds the unsigned tx, handles ERC-20 approvals, signs locally, broadcasts.

### Same-chain swap

```bash
mp token swap \
  --wallet <wallet-name> \
  --chain <chain> \
  --from-token <token-address> \
  --from-amount <amount> \
  --to-token <token-address>
```

Use `--to-amount` instead of `--from-amount` for exact-output swaps.

### Cross-chain bridge

```bash
mp token bridge \
  --from-wallet <wallet-name> \
  --from-chain <chain> \
  --from-token <token-address> \
  --from-amount <amount> \
  --to-chain <chain> \
  --to-token <token-address> \
  --to-wallet <wallet-name>     # optional; defaults to from-wallet
```

Same `--to-amount` exact-out support.

### Examples

```bash
# SOL → USDC on Solana
mp token swap \
  --wallet main --chain solana \
  --from-token So11111111111111111111111111111111111111111 \
  --from-amount 0.1 \
  --to-token EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v

# ETH on Ethereum → USDC.e on Polygon (cross-chain)
mp token bridge \
  --from-wallet funded --from-chain ethereum \
  --from-token 0x0000000000000000000000000000000000000000 \
  --from-amount 0.003 \
  --to-chain polygon \
  --to-token 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174

# ERC-20 swap (auto-approves first time)
mp token swap \
  --wallet funded --chain polygon \
  --from-token 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174 \
  --from-amount 5 \
  --to-token 0x0000000000000000000000000000000000000000
```

### Supported chains

`solana`, `ethereum`, `base`, `polygon`, `arbitrum`, `optimism`, `bnb`, `avalanche`, `bitcoin` (bridges only).

### Helpers

```bash
# Resolve names → addresses
mp token search --query "USDC" --chain solana

# Check balances pre-swap
mp token balance list --wallet <address> --chain <chain>
```

### Native-token addresses

- EVM chains: `0x0000000000000000000000000000000000000000`
- Solana: `So11111111111111111111111111111111111111111`

`token swap` calls `token bridge` under the hood with `from-chain == to-chain`.

---

## Trading automation — `mp` + cron / launchd

Compose the `mp` CLI with OS scheduling to run unattended trading strategies: dollar-cost averaging, limit orders, and stop losses. The skill generates shell scripts at `~/.config/moonpay/scripts/` and schedules them — no separate tooling needed.

### Prerequisites

```bash
mp user retrieve                                      # authenticated
mp token balance list --wallet <name> --chain <chain> # funded
which mp                                              # absolute path for cron / launchd
which jq                                              # JSON parsing
```

### Base script pattern

```bash
#!/bin/bash
set -euo pipefail

MP="$(which mp)"  # absolute path; cron / launchd have minimal PATH
LOG="$HOME/.config/moonpay/logs/trading.log"
mkdir -p "$(dirname "$LOG")"

log() { echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] $*" >> "$LOG"; }

# --- Config (agent fills these in per strategy) ---
WALLET="main"
CHAIN="solana"
FROM_TOKEN="EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"  # USDC
TO_TOKEN="So11111111111111111111111111111111111111111"     # SOL
AMOUNT=5

# --- Execute ---
log "SWAP: $AMOUNT $FROM_TOKEN -> $TO_TOKEN on $CHAIN"
RESULT=$("$MP" --json token swap \
  --wallet "$WALLET" --chain "$CHAIN" \
  --from-token "$FROM_TOKEN" --from-amount "$AMOUNT" \
  --to-token "$TO_TOKEN" 2>&1) || {
  log "FAILED: $RESULT"
  exit 1
}
log "OK: $RESULT"
```

`mp --json` outputs single-line JSON for `jq` parsing.

### DCA (dollar-cost averaging)

> "Buy $5 of SOL every day at 9am UTC"

Use the base script as-is. Schedule:

```bash
# Linux (crontab)
(crontab -l 2>/dev/null; \
 echo '0 9 * * * ~/.config/moonpay/scripts/dca-sol.sh # moonpay:dca-sol') | crontab -

# Common cron intervals:
#   0 * * * *      hourly
#   0 */4 * * *    every 4 hours
#   0 9 * * *      daily at 9am
#   0 9 * * 1      weekly Monday 9am
```

```xml
<!-- macOS launchd: ~/Library/LaunchAgents/com.moonpay.dca-sol.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.moonpay.dca-sol</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/bash</string>
    <string>/Users/USERNAME/.config/moonpay/scripts/dca-sol.sh</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
  <key>StandardErrorPath</key>
  <string>/Users/USERNAME/.config/moonpay/logs/dca-sol.err</string>
</dict>
</plist>
```

Load: `launchctl load ~/Library/LaunchAgents/com.moonpay.dca-sol.plist`. Tilde does **not** expand in plists — use `echo $HOME` for the absolute path.

### Limit order

> "Buy $50 of SOL when price drops below $80"

Pattern: poll price every 5 min; execute swap when condition met; self-disable after fill.

```bash
# Inside the base script, replace the Execute block with:
PRICE=$("$MP" --json token search --query "$TOKEN" --chain "$CHAIN" \
  | jq -r '.items[0].marketData.price')

if [ -z "$PRICE" ] || [ "$PRICE" = "null" ]; then
  log "LIMIT: price fetch failed, skipping"
  exit 0
fi

if (( $(echo "$PRICE < $TARGET_PRICE" | bc -l) )); then
  log "LIMIT: price $PRICE < $TARGET_PRICE — executing buy"
  RESULT=$("$MP" --json token swap \
    --wallet "$WALLET" --chain "$CHAIN" \
    --from-token "$BUY_WITH" --from-amount "$BUY_AMOUNT" \
    --to-token "$TOKEN" 2>&1) || { log "FAILED: $RESULT"; exit 1; }
  log "OK: bought at $PRICE"

  # Self-disable after fill
  if [[ "$OSTYPE" == "darwin"* ]]; then
    launchctl unload "$HOME/Library/LaunchAgents/com.moonpay.${SCRIPT_NAME}.plist" 2>/dev/null || true
  else
    crontab -l | grep -v "$SCRIPT_NAME" | crontab -
  fi
else
  log "LIMIT: price $PRICE >= $TARGET_PRICE — waiting"
fi
```

Schedule every 5 min:
- cron: `*/5 * * * * ~/.config/moonpay/scripts/limit-buy-sol.sh # moonpay:limit-buy-sol`
- launchd: `<key>StartInterval</key><integer>300</integer>` instead of `StartCalendarInterval`

### Stop loss

Same structure as limit order, but:
- Trigger when `price < trigger_price`
- Sell entire balance: query via `mp token balance list`, then `mp token swap` from sell-token to a stablecoin
- Self-disable after fill

### Managing automations

```bash
# List active
launchctl list | grep moonpay      # macOS
crontab -l | grep moonpay          # Linux

# Remove
launchctl unload ~/Library/LaunchAgents/com.moonpay.dca-sol.plist
rm ~/Library/LaunchAgents/com.moonpay.dca-sol.plist
crontab -l | grep -v "moonpay:dca-sol" | crontab -

# Pause / resume (macOS)
launchctl unload ~/Library/LaunchAgents/com.moonpay.dca-sol.plist
launchctl load ~/Library/LaunchAgents/com.moonpay.dca-sol.plist

# Logs
tail -50 ~/.config/moonpay/logs/trading.log
```

### Tips

- Start with a small DCA amount before scaling up.
- Keychain access requires an active user session — don't schedule on a machine that auto-locks aggressively.
- macOS launchd fires even after sleep; cron does not.
- Always tag cron entries with `# moonpay:{name}` so they're easy to find / remove.
- Use `bc -l` for decimal price comparison; bash can't compare floats natively. If `bc` isn't available: `awk "BEGIN {exit !($PRICE < $TARGET)}"`.

---

## Out of scope here (use upstream `moonpay/skills` directly if you need them)

The upstream `moonpay/skills` repo includes ~30+ skills. We mirror six surfaces above (buy, virtual-account, deposit, commerce, swap/bridge, trading-automation). The rest are out of scope for this curated skill because they overlap with Alchemy first-party or are niche / internal:

| Upstream skill | Why scope_out | Use |
| --- | --- | --- |
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
