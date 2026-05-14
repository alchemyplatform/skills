# Canonical Contract Addresses on Monad

> **CRITICAL:** Always verify with a network explorer before interacting. Never hallucinate an address — wrong addresses can mean **loss of funds**.

| Network | Explorer | RPC (public) |
| --- | --- | --- |
| Monad Mainnet | `monadscan.com` | `https://rpc.monad.xyz` |
| Monad Testnet | `testnet.monadscan.com` | `https://testnet-rpc.monad.xyz` |

If the user hasn't specified mainnet vs testnet, **ask**. Do not provide a mainnet address when the user asked for a testnet address (or vice versa).

For Alchemy's authenticated Monad endpoints (rate-limited and reliable), use `alchemy-cli` or `alchemy-api`.

## Verifying an address has code

If Foundry is installed:

```bash
# Mainnet
cast code <SMART_CONTRACT_ADDRESS> --rpc-url https://rpc.monad.xyz

# Testnet
cast code <SMART_CONTRACT_ADDRESS> --rpc-url https://testnet-rpc.monad.xyz
```

Otherwise call `eth_getCode` directly via JSON-RPC.

## Canonical contracts (Monad mainnet)

| Name | Address |
| --- | --- |
| Wrapped MON (`WMON`) | `0x3bd359C1119dA7Da1D913D1C4D2B7c461115433A` |
| Create2Deployer | `0x13b0D85CcB8bf860b6b79AF3029fCA081AE9beF2` |
| CreateX | `0xba5Ed099633D3B313e4D5F7bdc1305d3c28ba5Ed` |
| ERC-2470 Singleton Factory | `0xce0042b868300000d44a59004da54a005ffdcf9f` |
| **ERC-4337 EntryPoint v0.6** | `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` |
| ERC-4337 SenderCreator v0.6 | `0x7fc98430eAEdbb6070B35B39D798725049088348` |
| **ERC-4337 EntryPoint v0.7** | `0x0000000071727De22E5E9d8BAf0edAc6f37da032` |
| ERC-4337 SenderCreator v0.7 | `0xEFC2c1444eBCC4Db75e7613d20C6a62fF67A167C` |
| ERC-6492 UniversalSigValidator | `0xdAcD51A54883eb67D95FAEb2BBfdC4a9a6BD2a3B` |
| Foundry Deterministic Deployer | `0x4e59b44847b379578588920ca78fbf26c0b4956c` |
| Multicall3 | `0xcA11bde05977b3631167028862bE2a173976CA11` |
| MultiSend | `0x998739BFdAAdde7C933B942a68053933098f9EDa` |
| MultiSendCallOnly | `0xA1dabEF33b3B82c7814B6D82A79e50F4AC44102B` |
| Permit2 | `0x000000000022d473030f116ddee9f6b43ac78ba3` |
| Safe | `0x69f4D1788e39c87893C980c06EdF4b7f686e2938` |
| SafeL2 | `0xfb1bffC9d739B8D520DaF37dF666da4C687191EA` |
| SafeSingletonFactory | `0x914d7Fec6aaC8cd542e72Bca78B30650d45643d7` |
| SimpleAccount | `0x68641DE71cfEa5a5d0D29712449Ee254bb1400C2` |
| Simple7702Account | `0xe6Cae83BdE06E4c305530e199D7217f42808555B` |
| Sub Zero VanityMarket | `0x000000000000b361194cfe6312EE3210d53C15AA` |
| Zoltu Deterministic Deployment Proxy | `0x7A0D94F55792C434d74a40883C6ed8545E406D12` |

## ERC-8004 (AI agent identity registry — same on Monad mainnet and testnet)

| Contract | Address |
| --- | --- |
| IdentityRegistry | `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432` |
| ReputationRegistry | `0x8004BAa17C55a88189AE136b182e5fdA19dE9b63` |

## Bridged stablecoins (Monad mainnet)

| Symbol | Name | Address |
| --- | --- | --- |
| AUSD | Agora USD | `0x00000000eFE302BEAA2b3e6e1b18d08D69a9012a` |
| USDC | USD Coin | `0x754704Bc059F8C67012fEd69BC8A327a5aafb603` |
| USDT0 | Tether USD | `0xe7cd86e13AC4309349F30B3435a9d337750fC82D` |
| USD1 | USD1 | `0x111111d2bf19e43C34263401e0CAd979eD1cdb61` |
| thBILL | Theo Short Duration UST Fund | `0xfDD22Ce6D1F66bc0Ec89b20BF16CcB6670F55A5a` |
| wsrUSD | Wrapped srUSD | `0x4809010926aec940b550D34a46A52739f996D75D` |
| yzUSD | Yuzu USD | `0x9dcB0D17eDDE04D27F387c89fECb78654C373858` |
| syzUSD | Staked Yuzu USD | `0x484be0540aD49f351eaa04eeB35dF0f937D4E73f` |

## Bridged ETH-related (Monad mainnet)

| Symbol | Name | Address |
| --- | --- | --- |
| WETH | Wrapped Ether | `0xEE8c0E9f1BFFb4Eb878d8f15f368A02a35481242` |
| ezETH | Renzo Restaked ETH | `0x2416092f143378750bb29b79eD961ab195CcEea5` |
| wstETH | Lido Wrapped Staked ETH | `0x10Aeaf63194db8d453d4D85a06E5eFE1dd0b5417` |
| weETH | Wrapped EtherFi ETH | `0xA3D68b74bF0528fdD07263c60d6488749044914b` |
| pufETH | pufETH | `0x37D6382B6889cCeF8d6871A8b60E667115eDDBcF` |

## Bridged BTC-related (Monad mainnet)

| Symbol | Name | Address |
| --- | --- | --- |
| cbBTC | Coinbase Wrapped BTC | `0xd18B7EC58Cdf4876f6AFebd3Ed1730e4Ce10414b` |
| WBTC | Wrapped Bitcoin | `0x0555E30da8f98308EdB960aa94C0Db47230d2B9c` |
| LBTC | Lombard Staked Bitcoin | `0xecAc9C5F704e954931349Da37F60E39f515c11c1` |
| BTC.b | BTC.b | `0xB0F70C0bD6FD87dbEb7C10dC692a2a6106817072` |
| SolvBTC | Solv BTC | `0xaE4EFbc7736f963982aACb17EFA37fCBAb924cB3` |
| xSolvBTC | xSolvBTC | `0xc99F5c922DAE05B6e2ff83463ce705eF7C91F077` |

## MON on other chains

| Name | Chain | Address |
| --- | --- | --- |
| WMON | Solana | `CrAr4RRJMBVwRsZtT62pEhfA9H5utymC2mVx8e7FreP2` |
| WMON | Ethereum | `0x6917037f8944201b2648198a89906edf863b9517` |

## Beyond the canonical set

For protocol-specific addresses (DEXs, lending markets, etc.), check the official Monad repositories — they are versioned and curated:

- **Protocols:** https://github.com/monad-crypto/protocols
  Organized as `mainnet/<protocol>.jsonc` and `testnet/<protocol>.jsonc`. Each file has an `addresses` object with named entries.
- **Token list:** https://github.com/monad-crypto/token-list
  Per-token folders under `mainnet/` with address JSON.

If an address isn't in the tables above or the upstream repos, **ask the user**. Do not hallucinate.

## Upstream reference

Full monskills `addresses/` skill: https://github.com/therealharpaljadeja/monskills/blob/main/addresses/SKILL.md
