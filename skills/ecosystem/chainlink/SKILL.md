---
name: chainlink
description: Read Chainlink Data Feeds (on-chain price feeds via `AggregatorV3Interface`) with the correct safety defaults — staleness checks, `decimals()` lookup, L2 sequencer uptime — and use Chainlink CCIP for cross-chain messaging and programmable token transfers. Covers Data Feeds (price, rates, volatility, SmartData / RWA, MVR bundle, SVR/OEV) and CCIP (sender / receiver contracts, fee estimation, message status, CCT setup). NOT for off-chain spot token prices for valuation, token metadata, current wallet balances, transaction history, or NFT data — for those use `alchemy-cli` (live), `alchemy-mcp`, `alchemy-api` (app code), or `agentic-gateway` (no API key). For non-CCIP cross-chain bridges, use the `lifi` ecosystem skill.
license: MIT
compatibility: No Chainlink-specific API key required. Reads use `AggregatorV3Interface` on-chain — transport via `alchemy-api` JSON-RPC (`$ALCHEMY_API_KEY`) or `agentic-gateway`. CCIP writes require user wallet for signing. Optional `@chainlink/mcp-server` MCP tool for SDK-driven monitoring / discovery.
metadata:
  author: chainlink
  version: "0.1"
  provider: chainlink
  partner: "true"
---

# Chainlink (Data Feeds + CCIP)

Chainlink Data Feeds expose on-chain price + rate oracles via `AggregatorV3Interface`. Chainlink CCIP enables cross-chain messages and programmable token transfers between connected chains. This skill covers both surfaces with the safety patterns Chainlink mandates. For off-chain spot prices (portfolio valuation, historical prices), token metadata, balances, transfers, or NFTs, use the corresponding Alchemy skill instead. For non-CCIP cross-chain bridges, use the `lifi` ecosystem skill.

| | |
| --- | --- |
| **Data Feeds transport** | On-chain `eth_call` to feed contracts via `alchemy-api` JSON-RPC (or `agentic-gateway` x402/MPP) |
| **Data Feeds interface** | `AggregatorV3Interface` (`latestRoundData`, `decimals`, `description`, `version`) |
| **CCIP routers** | Per-chain `Router` contracts; fees via `getFee()`, sends via `ccipSend()` |
| **MCP (optional)** | `@chainlink/mcp-server` exposes `ccip_sdk` tool for monitoring / discovery |
| **Safety floor** | Staleness check, dynamic `decimals()`, L2 sequencer uptime, no `answeredInRound` |

## When to use this skill

Use `chainlink` when **any** of the following are true:

- The user wants to **read a Chainlink price feed** on EVM (or Solana / StarkNet / Aptos / Tron)
- The user wants to **generate a Chainlink consumer contract** with the right safety defaults
- The user wants **MVR bundle**, **SVR/OEV**, **SmartData/RWA**, **rates**, or **volatility** feeds
- The user wants to **send a CCIP cross-chain message** or **transfer tokens** via CCIP
- The user wants to **set up a Cross-Chain Token (CCT)**, configure pools, set rate limits
- The user wants to **monitor CCIP message status** or check route connectivity

## When NOT to use this skill (handoff)

| Need | Use instead |
| --- | --- |
| Off-chain spot token prices for valuation (portfolio totals, USD displays) | `alchemy-api` (Prices API) — aggregates many sources with simpler API |
| Historical token prices, OHLCV-like intervals | `alchemy-api` (Prices API) |
| Cross-chain bridges via non-CCIP routes (more bridges, more chains) | `lifi` (ecosystem skill) |
| Token metadata, search, list by chain | `alchemy-api` (Token API) |
| Current wallet balances | `alchemy-api` (Portfolio / Token API) |
| Transaction history (transfers in / out) | `alchemy-api` (Transfers API) |
| NFT metadata / floor / ownership | `alchemy-api` (NFT API) |
| Live RPC reads (block #, gas, generic `eth_call`) | `alchemy-cli` (live) or `alchemy-api` (JSON-RPC) |
| Account abstraction (bundlers, gas managers) | `alchemy-api` |
| Pre-execution simulation beyond CCIP fee estimation | `alchemy-api` (Simulation API) |
| Signed transaction submission | the app's wallet / `alchemy-api` |

## Scope contract

**This skill covers (`scope_in`):**

- **Data Feeds:** `AggregatorV3Interface.latestRoundData()` reads with mandatory safety defaults (staleness, dynamic decimals, L2 sequencer uptime), feed-address discovery, MVR bundle feeds, SVR/OEV feeds, SmartData/RWA, rates and volatility feeds, multi-chain (EVM + Solana + StarkNet + Aptos + Tron) feed-reading patterns, consumer contract generation
- **CCIP:** sender / receiver contracts (Solidity), `Router.ccipSend()`, fee estimation via `getFee()`, programmable token transfers, CCT (Cross-Chain Token) creation and pool setup, rate-limit configuration, route connectivity checks, supported-token discovery, message status / monitoring

**This skill does NOT cover (`scope_out`):**

- Off-chain spot token prices for portfolio valuation → handoff: `alchemy-api` (Prices API). Chainlink Data Feeds are on-chain oracles intended for smart contract consumption; for "what is ETH worth in USD right now in my UI", `alchemy-api` Prices API is the right path.
- Historical / time-series token prices → handoff: `alchemy-api` (Prices API)
- Token metadata, balances, transfers, NFT data → handoff: `alchemy-api`
- General `eth_call` / RPC reads (Chainlink doesn't host the chain) → handoff: `alchemy-api` JSON-RPC (use this transport for Data Feeds reads)
- Cross-chain bridges via non-CCIP routes → handoff: `lifi` (ecosystem skill)
- Data Streams (sub-second pull-oracle feeds), CRE / ACE / configurator / deployer skills → out of scope for this curated skill; users who want them can install upstream `smartcontractkit/chainlink-agent-skills` alongside
- Account abstraction → handoff: `alchemy-api` (Wallets / Bundler / Gas Manager)
- Pre-execution simulation beyond CCIP fee estimation → handoff: `alchemy-api` (Simulation API)
- Signed tx submission → user's wallet (CCIP returns calldata; app signs and submits)

## Setup

No Chainlink-specific API key. Reads need an EVM RPC; use `$ALCHEMY_API_KEY`:

```bash
export ALCHEMY_API_KEY="..."
```

Optional: install the Chainlink MCP server for `ccip_sdk` tool access from your AI client (Claude Code, Cursor, etc.):

```bash
npm install -g @chainlink/mcp-server
# Configure your MCP client to point at it (see Chainlink docs)
```

## Endpoint reference

### Data Feeds → [references/data-feeds.md](./references/data-feeds.md)

| Surface | Use for |
| --- | --- |
| `AggregatorV3Interface.latestRoundData()` | Read latest price + `updatedAt` + `roundId` from a feed contract |
| `AggregatorV3Interface.decimals()` | Dynamic decimal lookup (never hardcode) |
| `AggregatorV3Interface.description()` | Feed identifier (e.g., `"ETH / USD"`) |
| L2 Sequencer Uptime Feed | Mandatory check on Arbitrum / Optimism / Base / Scroll before trusting a feed |
| MVR `BundleAggregatorProxy` | Multi-variable bundle feeds |
| SVR / OEV proxies | Smart Value Recapture (MEV recapture) feeds |

### CCIP → [references/ccip.md](./references/ccip.md)

| Surface | Use for |
| --- | --- |
| `Router.getFee(...)` | Estimate fee for a cross-chain message before sending |
| `Router.ccipSend(...)` | Send a cross-chain message (data, tokens, or both) |
| `IAny2EVMMessageReceiver.ccipReceive(...)` | Receive messages on the destination chain |
| CCT pool contracts | Programmable token transfers — burn-mint, lock-release, rate-limited |
| Off-chain message status | Poll via `@chainlink/mcp-server` (`ccip_sdk`) or CCIP Explorer |

## Safety defaults (Data Feeds — non-negotiable)

These MUST be in any consumer contract or read-script generated:

1. **Staleness check** — compare `updatedAt` against a threshold derived from the feed's heartbeat. Never skip.
2. **Dynamic `decimals()`** — never hardcode. Different feeds use different decimal counts.
3. **L2 sequencer uptime** — on Arbitrum / Optimism / Base / Scroll / etc., always include a sequencer-uptime check with a grace period after recovery.
4. **Don't use `answeredInRound`** — deprecated; do not use for freshness validation.
5. **Mark example code unaudited** — remind users that generated code requires a security review before mainnet.

## Quick examples

### Read ETH/USD on Ethereum (via `alchemy-api` JSON-RPC)

```bash
# ETH/USD feed: 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419 (Ethereum mainnet)
curl https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_call",
    "params": [{
      "to":   "0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419",
      "data": "0xfeaf968c"
    }, "latest"]
  }'
# Selector 0xfeaf968c = latestRoundData()
# Returns: (roundId, answer, startedAt, updatedAt, answeredInRound) ABI-encoded
# Always also call decimals() (selector 0x313ce567) to scale `answer` correctly
```

### Generate a Solidity consumer (with safety defaults baked in)

See [references/data-feeds.md](./references/data-feeds.md) for the canonical Solidity template — staleness check, `decimals()` call, L2 sequencer-uptime check, and the inline comment block flagging that the example is unaudited.

### Estimate a CCIP fee + send a cross-chain message

See [references/ccip.md](./references/ccip.md) for sender contract patterns, `Router.getFee` calldata construction, and the approval protocol Chainlink recommends before any on-chain CCIP action.

## Common gotchas

- **Hardcoded decimals are a bug**: ETH/USD is 8 decimals on most chains, but rates feeds (e.g., stETH/ETH) are 18. Always call `decimals()`.
- **Staleness varies by feed**: BTC/USD heartbeat ≈ 1 hour; less-liquid pairs can be 24 hours. The skill recommends `block.timestamp - updatedAt < 2 * heartbeat` as a starting threshold; tune per feed.
- **`answeredInRound`** is deprecated — do not use for freshness validation.
- **L2 sequencer feeds are separate addresses** from the price feeds. Skipping them on L2 chains is unsafe — the price feed will keep returning stale data while the L2 sequencer is down.
- **Mainnet writes via CCIP**: this skill's scope_in includes contract generation and fee estimation, but the upstream skill refuses mainnet write actions in v0. Treat any `ccipSend` to mainnet as testnet-first; require explicit user approval for any on-chain action.
- **Off-chain prices ≠ on-chain feed**: an `alchemy-api` Prices API quote is fine for "show ETH = $3,200 in the UI". A Chainlink Data Feed read is needed when a **smart contract** decides whether to liquidate based on the price — the on-chain oracle is the source of truth for on-chain logic.
- **Non-EVM chains**: Solana, Aptos, StarkNet, and Tron have different feed-access patterns — see [references/data-feeds.md](./references/data-feeds.md) for chain-specific guidance.

## Routing back to Alchemy

If during a session the user's need shifts to surfaces this skill doesn't cover:

- For **off-chain prices for valuation / display**: `alchemy-api` (Prices API)
- For **token metadata, balances, transfer history, NFTs**: `alchemy-api`
- For **generic JSON-RPC or live debugging**: `alchemy-cli` (live) or `alchemy-api`
- For **app code without an API key**: `agentic-gateway`
- For **non-CCIP cross-chain bridges**: `lifi` (ecosystem skill)

The actual transport for Data Feeds reads (`eth_call` to `AggregatorV3Interface`) is `alchemy-api` JSON-RPC — Chainlink's value is the *what to call and how to interpret it* layer, not RPC hosting.

---

> **Maintenance:** Chainlink Labs maintains the underlying contracts and SDKs; this skill itself is maintained jointly by Alchemy and Chainlink Labs. File issues against `alchemyplatform/skills` with `[ecosystem/chainlink]` in the title. Data Streams, CRE, ACE, and configurator/deployer skills from upstream are intentionally not mirrored here — install `smartcontractkit/chainlink-agent-skills` alongside if you need them.
