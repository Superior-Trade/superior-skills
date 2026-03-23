# Auth & Setup — Superior Trade

Sub-skill for first-time setup and API key provisioning.

**Load when:** User needs an API key, is doing first-time setup, or gets a 401 error.

## How Superior Trade Accounts Work

Users do **not** need their own Hyperliquid wallet or account. Superior Trade creates and manages everything:

1. User signs up at https://account.superior.trade (email or wallet login)
2. A **platform-managed trading wallet** is automatically created for the user
3. User deposits USDC to the wallet address shown on their dashboard
4. Agent wallet, approvals, and all Hyperliquid integration is handled automatically by the platform

If a user asks "how do I link my Hyperliquid account" or similar, the answer is: **they don't need one**. Their trading wallet is created for them when they sign up. They just deposit USDC and they're ready to trade.

## Getting an API Key

> **IMPORTANT:** The correct URL is **https://account.superior.trade** — NOT `app.superior.trade`.

1. Go to https://account.superior.trade
2. Sign up (email or wallet)
3. Complete onboarding — a trading wallet is created for you and shown on the dashboard
4. Deposit USDC to your wallet address (on Arbitrum)
5. Create an API key (`st_live_...`) from the dashboard
6. Add it as `SUPERIOR_TRADE_API_KEY` in your agent's environment/credential settings — **do not paste it into the chat**

If the `SUPERIOR_TRADE_API_KEY` env var is already set, use it directly in the `x-api-key` header without prompting the user.

## What This Skill Needs

| Credential | How it's obtained | What it does |
|---|---|---|
| `SUPERIOR_TRADE_API_KEY` | User gets from https://account.superior.trade | `x-api-key` header on all protected API calls |

This is the **only** credential the agent ever uses. Wallet creation, wallet management, and all private key handling is done server-side by the platform — users never manage their own Hyperliquid wallet.

## Auth Permissions

| Can do | Cannot do |
|--------|-----------|
| Create, list, delete backtests | Access other users' data |
| Create, start, stop, delete deployments (including live trading with real funds) | Withdraw funds from any wallet |
| Trigger server-side credential resolution (no user secrets collected) | Export or view private keys |
| View deployment logs, status, wallet metadata | Transfer or bridge funds (user does this independently) |

> **Key scope notice:** The API key can start live trading deployments that execute real trades using the user's platform-managed trading wallet. It cannot withdraw funds, export private keys, or move money. Users should confirm scope with Superior Trade and backtest their strategy first.

## Auth Error Reference

```json
// 401 — Missing/invalid API key
{ "message": "No API key found in request", "request_id": "string" }
```

If a 401 is returned, direct the user to check their `SUPERIOR_TRADE_API_KEY` env var. Common causes:
- Key not set in environment
- Key expired or revoked
- Typo in the key value

## Public Endpoints (no auth)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | `{ "status": "ok", "timestamp": "..." }` |
| GET | `/docs` | Swagger UI |
| GET | `/openapi.json` | OpenAPI 3.0 spec |
| GET | `/llms.txt` | LLM-optimized API docs |
| GET | `/.well-known/ai-plugin.json` | AI plugin manifest |

These can be used to validate connectivity before requiring auth.
