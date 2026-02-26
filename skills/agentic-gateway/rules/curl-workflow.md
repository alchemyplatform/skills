# Curl Workflow

A lightweight way to call any Alchemy gateway endpoint using curl and the `@alchemy/x402` CLI, without setting up a full npm project. The gateway supports JSON-RPC, NFT, Portfolio, and Prices APIs — all accessible with the same SIWE auth and payment flow.

## When to Use

- Answering quick blockchain questions (latest block, ETH balance, token balance, NFT ownership, token prices, portfolio data)
- Making a few API calls from the command line or a bash script
- No existing npm project and you don't want to set one up

For SDK-based workflows with automatic payment handling, see [making-requests](making-requests.md) instead.

## Step 0: Ensure Wallet Exists

Follow [wallet-bootstrap](wallet-bootstrap.md) before proceeding. Do NOT generate or import a wallet from this file — the wallet-bootstrap rule contains a mandatory user prompt that must be followed.

## Step 1: Generate a SIWE Token

```bash
npx @alchemy/x402 sign-siwe --private-key ./wallet-key.txt > siwe-token.txt
```

For subsequent requests, read from the cached file:

```bash
TOKEN=$(cat siwe-token.txt)
```

> **Important:** SIWE tokens expire after 1 hour by default. Use `--expires-after` to customize (e.g. `--expires-after 2h`). If you get a 401 `MESSAGE_EXPIRED` error, regenerate the token (see Step 4). Always add `siwe-token.txt` to `.gitignore`.

## Step 2: Make API Calls with curl

All gateway endpoints share the same base URL (`https://x402.alchemy.com`) and auth pattern. See [reference](reference.md) for the full list of supported endpoints, chain network slugs, and API methods.

---

### Node JSON-RPC (`/:chainNetwork/v2`)

#### Get the Latest Block Number

```bash
TOKEN=$(cat siwe-token.txt)

curl -s -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN" \
  -d '{"id":1,"jsonrpc":"2.0","method":"eth_blockNumber"}'
```

#### Get ETH Balance for an Address

```bash
TOKEN=$(cat siwe-token.txt)

curl -s -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN" \
  -d '{"id":1,"jsonrpc":"2.0","method":"eth_getBalance","params":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","latest"]}'
```

#### Read a Contract (e.g. USDC `balanceOf`)

The `eth_call` method lets you call read-only contract functions. For ERC-20 `balanceOf`, the data is the function selector `0x70a08231` followed by the address padded to 32 bytes:

```bash
TOKEN=$(cat siwe-token.txt)

# USDC balanceOf(0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045) on Ethereum Mainnet
# USDC contract: 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48
curl -s -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN" \
  -d '{"id":1,"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","data":"0x70a08231000000000000000000000000d8dA6BF26964aF9D7eEd9e03E53415D37aA96045"},"latest"]}'
```

---

### NFT API (`/:chainNetwork/nft/v3/*`)

#### Get NFTs Owned by an Address

```bash
TOKEN=$(cat siwe-token.txt)

curl -s -G "https://x402.alchemy.com/eth-mainnet/nft/v3/getNFTsForOwner" \
  --data-urlencode "owner=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045" \
  --data-urlencode "withMetadata=true" \
  --data-urlencode "pageSize=10" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN"
```

---

### Prices API (`/prices/v1/tokens/*`)

#### Get Token Prices by Symbol

```bash
TOKEN=$(cat siwe-token.txt)

curl -s -G "https://x402.alchemy.com/prices/v1/tokens/by-symbol" \
  --data-urlencode "symbols=ETH" \
  --data-urlencode "symbols=BTC" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN"
```

---

### Portfolio API (`/data/v1/assets/*`)

#### Get Token Balances Across Chains

```bash
TOKEN=$(cat siwe-token.txt)

curl -s -X POST "https://x402.alchemy.com/data/v1/assets/tokens/by-address" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN" \
  -d '{"addresses":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"],"withMetadata":true}'
```

## Step 3: Handle 402 Payment Required

If curl returns HTTP 402, the gateway requires a one-time USDC payment for this SIWE token. Extract the `PAYMENT-REQUIRED` header and use the CLI to create a payment:

```bash
TOKEN=$(cat siwe-token.txt)

# Save response headers and capture HTTP status code
HTTP_CODE=$(curl -s -o response.json -D headers.txt -w "%{http_code}" -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: SIWE $TOKEN" \
  -d '{"id":1,"jsonrpc":"2.0","method":"eth_blockNumber"}')

if [ "$HTTP_CODE" = "402" ]; then
  # Extract the PAYMENT-REQUIRED header value
  PAYMENT_REQUIRED=$(grep -i 'payment-required:' headers.txt | sed 's/^[^:]*: //' | tr -d '\r')

  # Generate payment signature using the CLI
  PAYMENT_SIG=$(npx @alchemy/x402 pay --private-key ./wallet-key.txt --payment-required "$PAYMENT_REQUIRED")

  # Retry with payment
  curl -s -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -H "Authorization: SIWE $TOKEN" \
    -H "Payment-Signature: $PAYMENT_SIG" \
    -d '{"id":1,"jsonrpc":"2.0","method":"eth_blockNumber"}'
else
  cat response.json
fi
```

For more details on the payment flow, see [payment](payment.md).

**Note:** After a successful payment, subsequent requests using the same SIWE token will return 200 without requiring payment again.

## Step 4: Handle 401 MESSAGE_EXPIRED

If curl returns HTTP 401 with `"code":"MESSAGE_EXPIRED"`, the SIWE token has expired. Regenerate it:

```bash
npx @alchemy/x402 sign-siwe --private-key ./wallet-key.txt > siwe-token.txt
# Retry the request with the new token
```

For other 401 error codes, see [authentication](authentication.md) for the full list of auth error codes.
