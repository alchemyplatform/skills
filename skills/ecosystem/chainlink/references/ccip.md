# Chainlink CCIP Reference

CCIP (Cross-Chain Interoperability Protocol) is Chainlink's cross-chain messaging + token-transfer layer. It powers messages, programmable token transfers, and CCT (Cross-Chain Token) deployments across connected chains.

These flows are non-overlapping with Alchemy first-party. For non-CCIP bridges (more chains, more bridges), route to the `lifi` ecosystem skill.

---

## Architecture (one paragraph)

A `Router` contract on the source chain accepts a CCIP message and forwards it (after fee payment) to an off-chain DON, which delivers it to a `Router` on the destination chain. The destination Router calls `ccipReceive(...)` on the receiver contract. Token transfers piggyback on the same message via `tokenAmounts[]`.

---

## Approval protocol (Chainlink-mandated)

Before **any** on-chain CCIP action (send / bridge / deploy / register), present a preflight summary to the user:

```text
Proposed on-chain action:
- Action:           ccipSend
- Network:          testnet
- Source chain:     Sepolia (chainSelector 16015286601757825753)
- Destination:      Base Sepolia (chainSelector 10344971235874465080)
- Route/lane:       sepolia → base-sepolia (active)
- Token/amount:     1.0 LINK
- Payload:          0x... (32 bytes)
- Contracts:        Router 0x..., Receiver 0x...
- Method:           Router.ccipSend(destinationChainSelector, message)
- Expected effect:  Message delivered to receiver on Base Sepolia within ~10min;
                    1.0 LINK debited from sender on Sepolia, credited on Base Sepolia.

Do you want me to execute this?
```

For testnet sends, **require a second explicit confirmation** immediately before execution. Refuse all mainnet write actions in this skill version (read-only mainnet lookups are fine).

---

## Sender contract (canonical pattern)

```solidity
// SPDX-License-Identifier: MIT
// UNAUDITED — review before mainnet use.
pragma solidity ^0.8.20;

import { IRouterClient } from "@chainlink/contracts-ccip/contracts/interfaces/IRouterClient.sol";
import { Client }        from "@chainlink/contracts-ccip/contracts/libraries/Client.sol";
import { IERC20 }        from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract Sender {
    IRouterClient public immutable router;
    IERC20        public immutable feeToken; // e.g., LINK

    error InsufficientFee(uint256 fee, uint256 balance);

    constructor(address _router, address _feeToken) {
        router   = IRouterClient(_router);
        feeToken = IERC20(_feeToken);
    }

    function send(
        uint64  destChainSelector,
        address receiver,
        bytes memory data,
        Client.EVMTokenAmount[] memory tokenAmounts
    ) external returns (bytes32 messageId) {
        Client.EVM2AnyMessage memory message = Client.EVM2AnyMessage({
            receiver:     abi.encode(receiver),
            data:         data,
            tokenAmounts: tokenAmounts,
            extraArgs:    Client._argsToBytes(Client.EVMExtraArgsV2({
                              gasLimit:        200_000,
                              allowOutOfOrderExecution: true
                          })),
            feeToken:     address(feeToken)
        });

        uint256 fee = router.getFee(destChainSelector, message);
        uint256 bal = feeToken.balanceOf(address(this));
        if (bal < fee) revert InsufficientFee(fee, bal);

        feeToken.approve(address(router), fee);
        messageId = router.ccipSend(destChainSelector, message);
    }
}
```

### Notes

- `destChainSelector` is the **CCIP chain selector** (uint64), not the EVM chain ID. Look up at [CCIP directory](https://docs.chain.link/ccip/directory).
- `feeToken` can be `address(0)` for native gas token, or LINK / wrapped native for ERC-20 fees.
- `extraArgs` carries `gasLimit` for the destination `ccipReceive` call. Tune per destination receiver complexity.
- `allowOutOfOrderExecution: true` is required by recent CCIP versions for non-blocking lanes.

---

## Receiver contract (canonical pattern)

```solidity
// SPDX-License-Identifier: MIT
// UNAUDITED — review before mainnet use.
pragma solidity ^0.8.20;

import { CCIPReceiver } from "@chainlink/contracts-ccip/contracts/applications/CCIPReceiver.sol";
import { Client }       from "@chainlink/contracts-ccip/contracts/libraries/Client.sol";

contract Receiver is CCIPReceiver {
    error UnexpectedSourceChain(uint64 chainSelector);
    error UnexpectedSender(address sender);

    uint64  public immutable expectedSourceChainSelector;
    address public immutable expectedSender;

    constructor(address _router, uint64 _sourceSelector, address _sender)
        CCIPReceiver(_router)
    {
        expectedSourceChainSelector = _sourceSelector;
        expectedSender              = _sender;
    }

    function _ccipReceive(Client.Any2EVMMessage memory message) internal override {
        // Source-chain validation
        if (message.sourceChainSelector != expectedSourceChainSelector) {
            revert UnexpectedSourceChain(message.sourceChainSelector);
        }

        // Sender validation (sender is abi-encoded address)
        address actualSender = abi.decode(message.sender, (address));
        if (actualSender != expectedSender) revert UnexpectedSender(actualSender);

        // Process message.data and message.destTokenAmounts...
    }
}
```

### Defensive receiver checklist

- Validate `sourceChainSelector` against your expected source.
- Validate the abi-decoded `sender` against your expected counterparty.
- Use access control on any business-logic methods invoked by `_ccipReceive`.
- Don't let untrusted callers call `ccipReceive` directly — `CCIPReceiver` already enforces `onlyRouter`, but custom receivers must too.

---

## Fee estimation off-chain

Use `Router.getFee(...)` from a script before constructing the on-chain `ccipSend` to avoid surprise reverts:

```bash
# Source chain Router on Sepolia: 0x0BF3dE8c5D3e8A2B34D2BEeB17ABfCeBaf363A59
# Encode the EVM2AnyMessage struct (use ethers / viem; raw curl is impractical here)
# Then call getFee(destChainSelector, message)
```

For agent-driven workflows where the `@chainlink/mcp-server` MCP tool is available, prefer `ccip_sdk.getFee(...)` over hand-rolling the calldata.

---

## Programmable token transfers (CCT)

CCT (Cross-Chain Tokens) are tokens with first-class CCIP integration. Pool contracts on each chain handle the token's cross-chain semantics:

- **Burn-mint pools** — burn on source, mint on destination (canonical for newly-bridged tokens)
- **Lock-release pools** — lock on source, release on destination (canonical for existing tokens with fixed supply)
- **Rate-limited pools** — cap throughput per epoch to limit blast radius

CCT setup is a multi-step flow:

1. Deploy the token on each chain (or use existing).
2. Deploy a pool contract per chain (`BurnMintTokenPool` / `LockReleaseTokenPool`).
3. Register the token + pools with CCIP via the `TokenAdminRegistry`.
4. Configure rate limits per lane.
5. Test on testnet before enabling mainnet lanes.

For step-by-step CCT setup, refer to [CCT Setup docs](https://docs.chain.link/ccip/concepts/cross-chain-tokens). This skill's scope_in includes generating pool / setup contracts; mainnet enable / register actions still require the upstream `smartcontractkit/chainlink-agent-skills` repo for current production behavior.

---

## Monitoring + status

```text
PENDING:    message accepted on source, awaiting DON delivery
SUCCESS:    message delivered + ccipReceive succeeded
FAILED:     ccipReceive reverted (delivered, but the receiver failed)
```

Where to look:

- **CCIP Explorer** — https://ccip.chain.link/ (web UI; copy a tx hash or messageId)
- **`@chainlink/mcp-server`** — `ccip_sdk` MCP tool exposes `getMessageStatus`, `listLanes`, `listSupportedTokens`
- **On-chain events** — `OffRamp.ExecutionStateChanged(messageId, state)` on the destination chain

---

## Out of scope here

- **Data Streams** — Chainlink's sub-second pull-oracle product (separate from Data Feeds). Install upstream `smartcontractkit/chainlink-agent-skills` if you need it.
- **CRE / ACE / configurator / deployer** — Chainlink Runtime Environment skills.
- **Mainnet writes via this skill** — refused per safety guardrails. Refer users to the upstream skill for current mainnet write workflows; testnet writes are allowed with the approval protocol above.
- **Non-CCIP cross-chain bridges** — for users who want broader bridge coverage (27 bridges across 60+ chains), route to `lifi`.
