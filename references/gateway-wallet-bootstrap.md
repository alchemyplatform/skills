# Wallet Bootstrap

Use this rule when the agent does not have access to an Ethereum wallet (no `wallet.json` file exists).

## Step 1: Create a wallet

```typescript
import { generatePrivateKey, privateKeyToAccount } from "viem/accounts";

const privateKey = generatePrivateKey();
const account = privateKeyToAccount(privateKey);

const wallet = {
  address: account.address,
  privateKey: privateKey,
};

console.log(JSON.stringify(wallet, null, 2));
```

This outputs wallet JSON to stdout:

```json
{
  "address": "0xYourChecksummedAddress",
  "privateKey": "0x..."
}
```

Save the output to `wallet.json`:

```bash
npx tsx create-wallet.ts > wallet.json
```

## Step 2: Add wallet.json to .gitignore

Append `wallet.json` to `.gitignore` to prevent committing the private key:

```bash
echo "wallet.json" >> .gitignore
```

## Step 3: Fund the wallet with testnet USDC

1. Go to the [Circle USDC faucet](https://faucet.circle.com/)
2. Select **Base Sepolia**
3. Paste your wallet address
4. Request testnet USDC

The USDC will arrive at your address on Base Sepolia (`0x036CbD53842c5426634e7929541eC2318f3dCF7e`).

## Step 4: Load the wallet in code

```typescript
import { readFileSync } from "node:fs";
import { privateKeyToAccount } from "viem/accounts";

const wallet = JSON.parse(readFileSync("wallet.json", "utf-8")) as {
  address: string;
  privateKey: `0x${string}`;
};

const account = privateKeyToAccount(wallet.privateKey);
```

Use this `account` for SIWE token generation (see [gateway-authentication](gateway-authentication.md)) and payment signing (see [gateway-making-requests](gateway-making-requests.md)).
