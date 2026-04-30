# Chainlink Data Feeds Reference

**Transport:** on-chain `eth_call` via `alchemy-api` JSON-RPC (or `agentic-gateway`). Chainlink doesn't host RPC; this skill provides the contract addresses, ABI, and safety patterns.

For off-chain spot prices for portfolio valuation or UI display, route to `alchemy-api` (Prices API) instead — Chainlink Data Feeds are oracle reads intended for **smart contract consumption**, not for off-chain valuation.

---

## `AggregatorV3Interface`

The standard interface every Chainlink Data Feed implements:

```solidity
interface AggregatorV3Interface {
  function decimals() external view returns (uint8);
  function description() external view returns (string memory);
  function version() external view returns (uint256);

  function latestRoundData() external view returns (
    uint80  roundId,
    int256  answer,
    uint256 startedAt,
    uint256 updatedAt,
    uint80  answeredInRound  // DEPRECATED — do not use
  );

  function getRoundData(uint80 _roundId) external view returns (
    uint80  roundId,
    int256  answer,
    uint256 startedAt,
    uint256 updatedAt,
    uint80  answeredInRound
  );
}
```

Function selectors (for direct `eth_call`):

- `decimals()` → `0x313ce567`
- `description()` → `0x7284e416`
- `version()` → `0x54fd4d50`
- `latestRoundData()` → `0xfeaf968c`
- `getRoundData(uint80)` → `0x9a6fc8f5`

---

## Reading a feed off-chain (via `alchemy-api`)

```bash
# 1. Look up the feed address for your chain + pair
#    (e.g., ETH/USD on Ethereum mainnet: 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419)
#    Source: https://docs.chain.link/data-feeds/price-feeds/addresses

# 2. Call latestRoundData()
curl https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{ "jsonrpc": "2.0", "id": 1, "method": "eth_call",
        "params": [{ "to": "0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419",
                     "data": "0xfeaf968c" }, "latest"] }'

# 3. Decode (ABI: roundId uint80, answer int256, startedAt uint256, updatedAt uint256, answeredInRound uint80)

# 4. Always also call decimals() and validate updatedAt against the feed's heartbeat
curl https://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY \
  -H "Content-Type: application/json" \
  -d '{ "jsonrpc": "2.0", "id": 2, "method": "eth_call",
        "params": [{ "to": "0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419",
                     "data": "0x313ce567" }, "latest"] }'
```

---

## Solidity consumer (canonical pattern)

```solidity
// SPDX-License-Identifier: MIT
// UNAUDITED — review before mainnet use.
pragma solidity ^0.8.20;

import { AggregatorV3Interface } from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

contract PriceConsumer {
    AggregatorV3Interface public immutable feed;

    /// Heartbeat in seconds for the specific feed (look this up in Chainlink docs).
    /// e.g., ETH/USD on mainnet ≈ 3600 (1 hour); less-liquid pairs may be 86400 (24 hours).
    uint256 public immutable heartbeat;

    /// Tolerated staleness window (typically 1.5x – 2x heartbeat).
    uint256 public immutable staleAfter;

    error PriceStale(uint256 updatedAt, uint256 staleAfter);
    error InvalidPrice(int256 answer);

    constructor(address _feed, uint256 _heartbeat) {
        feed       = AggregatorV3Interface(_feed);
        heartbeat  = _heartbeat;
        staleAfter = _heartbeat * 2;
    }

    /// Returns price scaled to the feed's reported decimals.
    function latestPrice() public view returns (int256 answer, uint8 decimals_) {
        (, int256 _answer,, uint256 _updatedAt,) = feed.latestRoundData();

        // Mandatory: staleness check
        if (block.timestamp - _updatedAt > staleAfter) {
            revert PriceStale(_updatedAt, staleAfter);
        }

        // Mandatory: invalid-price check
        if (_answer <= 0) revert InvalidPrice(_answer);

        // Mandatory: dynamic decimals
        decimals_ = feed.decimals();
        return (_answer, decimals_);
    }
}
```

### L2 sequencer uptime addition (Arbitrum / Optimism / Base / Scroll / etc.)

```solidity
import { AggregatorV3Interface } from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

AggregatorV3Interface public immutable sequencerUptimeFeed;
uint256 public constant GRACE_PERIOD = 3600; // 1 hour after sequencer recovery

error SequencerDown();
error GracePeriodNotOver(uint256 startedAt, uint256 graceEnds);

function _requireSequencerUp() internal view {
    (, int256 sequencerStatus, uint256 startedAt,,) = sequencerUptimeFeed.latestRoundData();

    // 0 = up, 1 = down
    if (sequencerStatus == 1) revert SequencerDown();

    // Even if up, wait for the grace period after recovery
    if (block.timestamp - startedAt <= GRACE_PERIOD) {
        revert GracePeriodNotOver(startedAt, startedAt + GRACE_PERIOD);
    }
}
```

Sequencer Uptime Feed addresses (look up at https://docs.chain.link/data-feeds/l2-sequencer-feeds):
- Arbitrum One: `0xFdB631F5EE196F0ed6FAa767959853A9F217697D`
- Optimism: `0x371EAD81c9102C9BF4874A9075FFFf170F2Ee389`
- Base: `0xBCF85224fc0756B9Fa45aA7892530B47e10b6433`
- Scroll: `0x45c2b8C204568A03Dc7A2E32B71D67Fe97F908A9`

---

## Feed types

| Type | Use for | Reference doc |
| --- | --- | --- |
| Standard price feeds | Token / fiat / commodity prices | [Price Feed addresses](https://docs.chain.link/data-feeds/price-feeds/addresses) |
| **MVR (Multi-Variable Response) bundle feeds** | Multi-asset bundles via `BundleAggregatorProxy` | [MVR Feeds](https://docs.chain.link/data-feeds/mvr-feeds) |
| **SVR / OEV feeds** | Smart Value Recapture / OEV recapture | [SVR Feeds](https://docs.chain.link/data-feeds/svr) |
| **SmartData / RWA feeds** | NAV, reserves, tokenized equity (e.g., USDM, AUSD) | [SmartData](https://docs.chain.link/data-feeds/smartdata) |
| **Rates & Volatility feeds** | Interest rates (`stETH/ETH`, `eUSD/USD`), realized volatility | [Rates & Vol](https://docs.chain.link/data-feeds/rates-feeds) |
| **L2 Sequencer Uptime feeds** | L2 health gating | [L2 Sequencer Feeds](https://docs.chain.link/data-feeds/l2-sequencer-feeds) |

---

## Safety defaults (non-negotiable)

Every consumer contract or read script must:

1. **Validate `updatedAt` against a staleness threshold** (typically `2 * heartbeat`). Reject reads where `block.timestamp - updatedAt > staleAfter`.
2. **Call `decimals()` dynamically** — never hardcode (different feeds have different decimal counts).
3. **On L2 chains**, include the L2 Sequencer Uptime check with a grace period after recovery.
4. **Reject `answer <= 0`** as invalid.
5. **Do not use `answeredInRound`** — deprecated. The current freshness validator is `updatedAt`.
6. **Mark example code as unaudited** — remind users to commission a security review before mainnet.

---

## Multi-chain (non-EVM)

Chainlink Data Feeds are also available on Solana, Aptos, StarkNet, and Tron — with chain-specific access patterns (e.g., reading via Solana SDK, not `eth_call`). For non-EVM feed access, refer to:

- [Solana feeds](https://docs.chain.link/data-feeds/solana)
- [Aptos feeds](https://docs.chain.link/data-feeds/aptos)
- [StarkNet feeds](https://docs.chain.link/data-feeds/starknet)
- [Tron feeds](https://docs.chain.link/data-feeds/tron)

For Solana specifically, route reads through `alchemy-api` Solana RPC; the feed-account layout differs from EVM AggregatorV3Interface.

---

## Out of scope here

- **Data Streams** (Chainlink's sub-second pull-oracle product) — different access pattern (off-chain HTTP API + on-chain verifier contract). Install upstream `smartcontractkit/chainlink-agent-skills` if you need this.
- **CRE / ACE / configurator / deployer** — Chainlink Runtime Environment skills; out of scope for this curated skill.
