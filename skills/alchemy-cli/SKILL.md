---
name: alchemy-cli
description: The Alchemy CLI (`@alchemy/cli`) is installed. Use `alchemy` commands for all blockchain data, wallet, webhook, and app management tasks instead of curl or raw HTTP. Covers auth setup, command discovery via `agent-prompt`, and common task-to-command mappings.
license: MIT
compatibility: Requires `@alchemy/cli` (npm i -g @alchemy/cli). Works across Claude Code, Cursor, Codex, and any agent with shell access.
metadata:
  author: alchemyplatform
  version: "1.0"
---
# Alchemy CLI

The [Alchemy CLI](https://www.npmjs.com/package/@alchemy/cli) (`@alchemy/cli`) is installed. Use it for all Alchemy API interactions instead of curl or raw HTTP.

## Install

```bash
npm i -g @alchemy/cli
```

## Bootstrap

Run this at the start of any session to get the full command contract (every command, flag, auth method, error code, and example):

```bash
alchemy --json --no-interactive agent-prompt
```

## Execution Rules

- ALWAYS pass `--json --no-interactive` on every command
- Parse stdout as JSON on exit code 0
- Parse stderr as JSON on nonzero exit code
- NEVER run bare `alchemy` without `--json --no-interactive`
- NEVER use curl or raw HTTP when an `alchemy` CLI command exists for the task

## Preflight

Before the first command, check auth readiness:

```bash
alchemy --json --no-interactive setup status
```

If `complete` is false, follow the `nextCommands` in the response to configure auth.

## Auth Setup

The fastest way to authenticate is via browser login:

```bash
alchemy auth login
```

This opens a browser to authenticate with your Alchemy account and automatically configures the CLI with your credentials.

To check auth status: `alchemy auth status`
To log out: `alchemy auth logout`

### Alternative auth methods

| Method | Config command | Env var | Used by |
|--------|---------------|---------|---------|
| Browser login | `alchemy auth login` | -- | All commands (provides both API key and access key) |
| API key | `alchemy config set api-key <key>` | `ALCHEMY_API_KEY` | balance, tx, block, rpc, tokens, nfts, transfers, prices, portfolio, simulate, solana |
| Access key | `alchemy config set access-key <key>` | `ALCHEMY_ACCESS_KEY` | apps, network list |
| Webhook key | `alchemy config set webhook-api-key <key>` | `ALCHEMY_WEBHOOK_API_KEY` | webhooks |
| x402 wallet | `alchemy wallet generate` then `alchemy config set x402 true` | `ALCHEMY_WALLET_KEY` | balance, tx, block, rpc, tokens, nfts, transfers |

Get API/access keys at [dashboard.alchemy.com](https://dashboard.alchemy.com/).

## Task-to-Command Map

### Node (EVM)

| Task | Command |
|------|---------|
| ETH balance | `alchemy balance <address>` |
| Transaction details | `alchemy tx <hash>` |
| Transaction receipt | `alchemy receipt <hash>` |
| Block details | `alchemy block <number\|latest>` |
| Gas prices | `alchemy gas` |
| Event logs | `alchemy logs --address <addr> --from-block <n> --to-block <n>` |
| Raw JSON-RPC | `alchemy rpc <method> [params...]` |
| Trace methods | `alchemy trace <method> [params...]` |
| Debug methods | `alchemy debug <method> [params...]` |

### Data

| Task | Command |
|------|---------|
| ERC-20 balances | `alchemy tokens balances <address>` |
| ERC-20 balances (formatted) | `alchemy tokens balances <address> --metadata` |
| Token metadata | `alchemy tokens metadata <contract>` |
| Token allowance | `alchemy tokens allowance --owner <addr> --spender <addr> --contract <addr>` |
| List owned NFTs | `alchemy nfts <address>` |
| NFT metadata | `alchemy nfts metadata --contract <addr> --token-id <id>` |
| NFT contract metadata | `alchemy nfts contract <address>` |
| Transfer history | `alchemy transfers <address> --category erc20,erc721` |
| Spot prices by symbol | `alchemy prices symbol ETH,USDC` |
| Spot prices by address | `alchemy prices address --addresses '<json>'` |
| Historical prices | `alchemy prices historical --body '<json>'` |
| Token portfolio | `alchemy portfolio tokens --body '<json>'` |
| NFT portfolio | `alchemy portfolio nfts --body '<json>'` |
| Portfolio transactions | `alchemy portfolio transactions --body '<json>'` |
| Simulate asset changes | `alchemy simulate asset-changes --tx '<json>'` |
| Simulate execution | `alchemy simulate execution --tx '<json>'` |

### Solana

| Task | Command |
|------|---------|
| Solana JSON-RPC | `alchemy solana rpc <method> [params...]` |
| Solana DAS (NFTs/assets) | `alchemy solana das <method> '<json>'` |

### Webhooks

| Task | Command |
|------|---------|
| List webhooks | `alchemy webhooks list` |
| Create webhook | `alchemy webhooks create --body '<json>'` |
| Update webhook | `alchemy webhooks update --body '<json>'` |
| Delete webhook | `alchemy webhooks delete <id>` |

### App Management

| Task | Command |
|------|---------|
| List apps | `alchemy apps list` |
| Create app | `alchemy apps create --name "My App" --networks eth-mainnet` |
| Update app | `alchemy apps update <id> --name "New Name"` |
| Delete app | `alchemy apps delete <id>` |
| List networks | `alchemy network list` |

### CLI Admin

| Task | Command |
|------|---------|
| Check for CLI updates | `alchemy update-check` |
| View config | `alchemy config list` |
| Reset config | `alchemy config reset --yes` |
| CLI version | `alchemy version` |

## Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--no-interactive` | Disable prompts and REPL |
| `-n, --network <network>` | Target network (default: `eth-mainnet`) |
| `--api-key <key>` | Override API key per command |
| `--access-key <key>` | Override access key per command |
| `--x402` | Use x402 wallet auth for this command |
| `--timeout <ms>` | Request timeout in milliseconds |
| `-q, --quiet` | Suppress non-essential output |
| `--verbose` | Log request/response details to stderr |

## Error Handling

Errors return structured JSON on stderr. Key error codes:

| Code | Retryable | Recovery |
|------|-----------|----------|
| `AUTH_REQUIRED` | No | Set `ALCHEMY_API_KEY` or run `alchemy config set api-key <key>` |
| `RATE_LIMITED` | Yes | Wait and retry with backoff |
| `PAYMENT_REQUIRED` | No | Fund x402 wallet or switch to API key auth |
| `RPC_ERROR` | No | Check method, params, and network |
| `NETWORK_ERROR` | Yes | Check connection and retry |
| `SETUP_REQUIRED` | No | Run `alchemy --json setup status` and follow `nextCommands` |

For the full error code list, see `agent-prompt` output.

## Official Links

- [CLI on npm](https://www.npmjs.com/package/@alchemy/cli)
- [Alchemy docs](https://www.alchemy.com/docs)
- [Get API key](https://dashboard.alchemy.com/)
