# SIWE Authentication

Every request to the gateway must include an `Authorization` header with a SIWE token. The token proves wallet ownership without transmitting the private key.

## Token Format

```
Authorization: SIWE <base64(siwe_message)>.<signature>
```

## CLI: Generate a SIWE Token

For ad-hoc requests and curl workflows, use the `@alchemy/x402` CLI. Pass the path to a file containing the private key:

```bash
npx @alchemy/x402 sign-siwe --private-key ./wallet-key.txt
```

Options:

| Flag | Default | Description |
|------|---------|-------------|
| `--private-key <key>` | (required) | Path to a key file (recommended), hex private key, or raw hex |
| `--expires-after <duration>` | `1h` | Token lifetime (e.g. `30m`, `2h`, `7d`) |

> **Security:** Always pass a file path rather than a raw key to avoid exposing the private key in shell history and process listings.

The command prints the encoded SIWE token to stdout. Pipe it to a file to avoid terminal exposure:

```bash
npx @alchemy/x402 sign-siwe --private-key ./wallet-key.txt > siwe-token.txt
TOKEN=$(cat siwe-token.txt)

curl -s -X POST "https://x402.alchemy.com/eth-mainnet/v2" \
  -H "Authorization: SIWE $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"id":1,"jsonrpc":"2.0","method":"eth_blockNumber"}'
```

## Library: Generate a SIWE Token in Code

For applications, use the `signSiwe` function from `@alchemy/x402`. Read the private key from an environment variable — never hardcode it:

```bash
npm install @alchemy/x402
```

```typescript
import { signSiwe } from "@alchemy/x402";

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const siweToken = await signSiwe({
  privateKey,
  expiresAfter: "1h", // optional, default "1h"
});

// Use in requests
const headers = { Authorization: `SIWE ${siweToken}` };
```

## SIWE Message Fields

The generated token contains a SIWE message with these fields:

| Field | Value | Notes |
|-------|-------|-------|
| `domain` | `x402.alchemy.com` | Must match the gateway's configured domain |
| `address` | Your wallet address | Checksummed Ethereum address |
| `statement` | `Sign in to Alchemy Gateway` | Human-readable intent |
| `uri` | `https://x402.alchemy.com` | Must use `https://` + domain |
| `version` | `1` | SIWE spec version |
| `chainId` | `8453` | Base Mainnet chain ID |
| `nonce` | Random alphanumeric string | At least 8 characters |
| `expirationTime` | ISO 8601 timestamp | Default: 1 hour from now |

## Auth Error Codes

When authentication fails, the gateway returns HTTP 401 with a JSON body:

```json
{
  "error": "Unauthorized",
  "message": "<description>",
  "code": "<error_code>"
}
```

| Code | Cause | How to fix |
|------|-------|------------|
| `MISSING_AUTH` | No `Authorization` header provided | Add the `Authorization: SIWE <token>` header to your request |
| `INVALID_AUTH_FORMAT` | Token is not in `base64.signature` format or Base64 decoding failed | Ensure the token is `base64(message).signature` with exactly one `.` separator |
| `INVALID_SIWE` | SIWE message could not be parsed | Regenerate the token using the CLI or `signSiwe()` |
| `INVALID_SIGNATURE` | Signature does not match the message signer | Ensure the correct private key is being used |
| `INVALID_DOMAIN` | SIWE message domain does not match `x402.alchemy.com` | Use the `@alchemy/x402` CLI or library — they set the correct domain automatically |
| `MESSAGE_EXPIRED` | SIWE message `expirationTime` has passed or `notBefore` is in the future | Generate a new SIWE token — the current one has expired |

## Handling Auth Errors

If you receive a 401 response, parse the `code` field and take the appropriate action:

- **Regenerable errors** (`MESSAGE_EXPIRED`): Generate a fresh SIWE token and retry the request.
- **Configuration errors** (`INVALID_DOMAIN`, `MISSING_AUTH`, `INVALID_AUTH_FORMAT`): Fix the token generation code — these will not resolve by retrying.
- **Signing errors** (`INVALID_SIGNATURE`, `INVALID_SIWE`): The token is malformed. Regenerate using the CLI or `signSiwe()`.

```typescript
if (response.status === 401) {
  const body = await response.json();

  if (body.code === "MESSAGE_EXPIRED") {
    // Token expired — regenerate and retry
    const newToken = await signSiwe({ privateKey });
    // retry request with new token...
  } else {
    // Configuration or signing error — do not retry, fix the code
    throw new Error(`Auth failed (${body.code}): ${body.message}`);
  }
}
```
