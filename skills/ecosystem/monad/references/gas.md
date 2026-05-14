# Gas Pricing on Monad

Monad is EIP-1559 compatible but diverges from Ethereum in ways that **cost users real money** if ignored.

## The critical difference: `gas_limit` is what users pay

On Ethereum, users pay for gas **actually consumed**. On Monad, users pay based on the **gas limit** they set:

```
gas_paid = gas_limit * price_per_gas
```

This exists because block leaders build blocks **before** executing them (async execution), so actual gas usage isn't known at inclusion time — `gas_limit` is the contractually-committed price ceiling.

**Implications for developers:**

- Setting an unnecessarily high `gas_limit` directly costs users more MON.
- For transactions with known fixed costs, **hardcode the limit**. Native MON transfers are always 21,000 — hardcode it; do not rely on `eth_estimateGas` (some wallets fall back to a much higher limit on estimation failure).
- When you must estimate, keep the buffer small (≤10%).

```typescript
// Native transfer — hardcode the limit
const tx = { to, value: parseEther("1.0"), gasLimit: 21_000n };

// Contract call — tight buffer
const estimate = await publicClient.estimateGas({ to, data });
const gasLimit = estimate + estimate / 10n; // 10% max
```

## EIP-1559 pricing

Standard type 2 transactions:

```
price_per_gas = min(base_price_per_gas + priority_price_per_gas, max_price_per_gas)
```

The user signs `priority_price_per_gas` (tip) and `max_price_per_gas` (cap). The network controls `base_price_per_gas` for all transactions in a block.

## Block and transaction limits

| Parameter | Value |
| --- | --- |
| Block gas limit | 200M gas |
| Transaction gas limit | 30M gas |
| Minimum base fee | 100 MON-gwei (100 × 10⁻⁹ MON) |

Block gas limit is ~6.7× Ethereum's, so blocks fit substantially more transactions.

## Base fee controller

Monad's base fee controller **increases more slowly and decreases more quickly** than Ethereum's, to prevent blockspace underutilization from overpricing. Parameters:

| Param | Value |
| --- | --- |
| `max_step_size` | 1/28 |
| `target` | 160M gas (80% of block capacity) |
| `beta` | 0.96 |
| `epsilon` | 160M |

Exponential smoothing (`beta = 0.96`) tracks historical variance in block fullness, producing smoother transitions than Ethereum's simpler mechanism. **In practice: gas prices on Monad are more stable and recover faster after spikes.**

## Transaction ordering

The default Monad client uses a **Priority Gas Auction**: transactions sorted by descending total gas price (base + priority).

## Opcode repricing

Monad **raises** a few opcode prices (rather than discounting everything) to reflect different resource scarcity:

### Cold state access (3-4× more expensive)

| Operation | Ethereum | Monad |
| --- | --- | --- |
| Account access (cold) | 2,600 gas | **10,100 gas** |
| Storage access (cold) | 2,100 gas | **8,100 gas** |
| Account access (warm) | 100 gas | 100 gas (unchanged) |
| Storage access (warm) | 100 gas | 100 gas (unchanged) |

**Affected opcodes:** `BALANCE`, `EXTCODESIZE`, `EXTCODECOPY`, `EXTCODEHASH`, `CALL`, `CALLCODE`, `DELEGATECALL`, `STATICCALL`, `SELFDESTRUCT`, `SLOAD`, `SSTORE`.

**What this means:** Contracts that touch many cold storage slots or call many external contracts cost significantly more on Monad. Patterns that read a slot once (cold) then reuse it (warm) are fine. Patterns that do single cold reads across many slots are expensive.

### Cryptographic precompiles (2-5× more expensive)

| Precompile | Address | Ethereum | Monad | Multiplier |
| --- | --- | --- | --- | --- |
| ecRecover | `0x01` | 3,000 | 6,000 | 2× |
| ecAdd | `0x06` | 150 | 300 | 2× |
| ecMul | `0x07` | 6,000 | 30,000 | **5×** |
| ecPairing | `0x08` | 45,000 | 225,000 | **5×** |
| blake2f | `0x09` | rounds × 1 | rounds × 2 | 2× |
| point evaluation | `0x0a` | 50,000 | 200,000 | 4× |

**What this means:** Signature-verification-heavy contracts (`ecRecover`) cost 2× the gas. On-chain ZK verification (`ecMul`, `ecPairing`, point evaluation) is **4-5×** more expensive — Ethereum gas estimates will be significantly low.

## Showing gas costs in UI

Calculate from `gas_limit`, **not** `gas_used` from receipts (which is what Ethereum docs and most wallets do by default):

```typescript
const gasCost = gasLimit * gasPrice; // not receipt.gasUsed * gasPrice
```

## Upstream reference

Full monskills `gas/` skill: https://github.com/therealharpaljadeja/monskills/blob/main/gas/SKILL.md
