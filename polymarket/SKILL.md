---
name: Polymarket Prediction Market Trading
version: 1.0.0
updated: 2026-06-17
description: "Discover, backtest, and deploy prediction market trading strategies on Polymarket through Superior Trade's managed cloud."
homepage: https://superior.trade
source: https://github.com/Superior-Trade
primaryEnv: SUPERIOR_TRADE_PM_API_KEY
auth:
  type: api_key
  env: SUPERIOR_TRADE_PM_API_KEY
  header: "Authorization: Bearer"
  scope: "Read-write the user's own strategies, backtests, and deployments. Can start live trading deployments that execute real trades on Polymarket with the user's platform-managed wallet. Cannot export private keys or access other users' data. Withdrawals are not yet available via the API."
env:
  - name: SUPERIOR_TRADE_PM_API_KEY
    description: "Superior Trade prediction market API key (Bearer token). Obtained via POST /v3/account/onboard — no signup form required. Can create/manage strategies, backtests, and deployments including live trading. Cannot export private keys or access other users' data."
    required: true
    type: api_key
externalEndpoints:
  - url: https://api.superior.trade/v3
    purpose: "All market discovery, strategy validation, backtesting, deployment, and account operations"
---

# Polymarket Prediction Market Trading

Trade prediction markets on Polymarket through Superior Trade. Discover markets, write NautilusTrader strategies, backtest against historical trade data, and deploy live — all through one API.

**Base URL:** `https://api.superior.trade/v3`
**Auth:** `Authorization: Bearer <api_key>` header on all protected endpoints
**Docs:** `GET /v3/docs` (interactive reference), `GET /v3/openapi.json` (OpenAPI spec)

## Setup

### Getting an API Key (Zero-Friction Onboarding)

No manual signup or wallet setup is required. One public call provisions everything:

```
POST /v3/account/onboard
{ "label": "my-agent" }            // label optional, defaults to "agent"
```

**Response:**

```json
{
  "api_key": "sk_...",
  "user_id": "user_abc123",
  "wallet_id": "wal_abc123",
  "deposit_address": "0x1234...abcd",
  "builder_code": "0x9132...",
  "gasless": true,
  "message": "Save your api_key. Send USDC to deposit_address on Polygon to fund your account."
}
```

This auto-provisions:

1. **Wallet** — created via Polymarket builder relayer (gasless, no web UI needed)
2. **Polymarket allowances** — contract approvals set automatically (gasless)
3. **CLOB API credentials** — derived from the wallet signature when the first deployment starts

**Save the `api_key` immediately** — store it as `SUPERIOR_TRADE_PM_API_KEY` in the agent's environment/credential settings. If `SUPERIOR_TRADE_PM_API_KEY` is already set, use it directly in the `Authorization: Bearer` header without re-onboarding.

### Funding (The Only Manual Step)

The user must send **USDC on Polygon** to their `deposit_address`. The agent cannot move or bridge funds — the user handles this independently.

- Accepted assets: `pUSD`, `USDC`, `USDC.e` (on Polygon)
- No gas needed on the user's side — the platform relayer is gasless
- Get the deposit address any time via `GET /v3/account` or `POST /v3/account/deposit/polymarket`
- Check balances via `GET /v3/account/wallets` (returns per-wallet `usdc` and `pol` balances)

> **Withdrawals are not yet available via the API.** `POST /v3/account/withdraw/polymarket` exists but returns a pending stub — do not promise users that API withdrawals work. If a user asks to withdraw, say the feature is not yet live.

### Multi-Wallet Support

Each user can create up to **3 wallets** (hard limit). Each deployment uses one wallet, giving isolated balances and positions per strategy:

```
POST /v3/account/wallets { "label": "btc-carry" }    → wal_abc + deposit address
POST /v3/account/wallets { "label": "fed-spread" }   → wal_def + deposit address
GET  /v3/account/wallets                             → list all wallets with balances
POST /v3/deployment { "strategy_id": "...", "wallet_id": "wal_abc" }
```

If `wallet_id` is omitted when creating a deployment, the default wallet is used.

## Safety

### Security & Permissions

This skill requires exactly **one credential**: a Bearer token. The only secret the agent uses is `SUPERIOR_TRADE_PM_API_KEY`.

**Security rules (non-negotiable):**

1. **NEVER** ask users for private keys, seed phrases, or wallet credentials
2. **NEVER** log, store, or display private keys or seed phrases
3. **NEVER** fabricate wallet balances, API responses, market prices, or trade results
4. **NEVER** start a live deployment without explicit user confirmation
5. **NEVER** promise API withdrawals — the withdraw endpoint is not yet live
6. **Prefer user-friendly language** over internal technical names when speaking conversationally. Say "strategy" or "the bot" instead of internal class names or infrastructure details. If the user asks about the underlying technology, answer honestly (the platform uses NautilusTrader for strategy execution against Polymarket's CLOB).

| Can do                                                              | Cannot do                          |
| ------------------------------------------------------------------- | ---------------------------------- |
| Create, list, delete strategies                                     | Access other users' data           |
| Create and run backtests                                            | Export or view private keys        |
| Create, start, stop, exit deployments (live trading with real funds) | Withdraw funds (not yet available) |
| View positions, P&L, wallet balances                                | Transfer or bridge funds           |

### Polymarket-Specific Risks (tell the user when relevant)

- **Prices are probabilities** — every outcome trades between 0.001 and 0.999 pUSD. A "cheap" 0.03 outcome is not a bargain; it is a ~3% market-implied probability.
- **Resolution risk** — positions held to resolution settle at exactly 0 or 1. A strategy can be up and still settle at zero if the outcome resolves against it.
- **Liquidity risk** — many markets have thin order books. Market orders can fill far from the displayed price. Prefer markets with meaningful `volume_24h` and `liquidity`.
- **Market end dates** — markets stop trading at `end_date`. Never deploy a strategy on a market that resolves before the strategy has time to work.
- **Rate limits** — Polymarket enforces ~30 order submissions/minute and ~100 data requests/minute per user. Strategies that cancel/replace on every tick can hit these fast.

### Live Deployment Confirmation

Before any **live deployment start**, the agent MUST present this summary and wait for an explicit affirmative response:

```
Deployment Summary:
• Strategy: [strategy_class] ([strategy_id])
• Venue: Polymarket
• Market(s): [market question(s)]
• Wallet: [wallet_id / label] — balance: [usdc] USDC
• Trade size: [from strategy config] pUSD
• Market end date: [end_date]

⚠️ This will trade with REAL funds. Proceed? (yes/no)
```

Do NOT start a live deployment without an explicit affirmative response.

## Agent Operating Rules

- **Verification-first:** Every factual claim about balance, market price, position, or deployment status MUST be backed by an API call in the current turn. NEVER assume → report → verify later.
- **Anti-hallucination:** If you can't call the API, say "I haven't checked yet." Every number must come from a real response.
- **Backtest before deploy:** Always run a backtest and review results with the user before the first live deployment of any strategy.
- **Conversational:** Make API calls directly and present results conversationally. Show raw payloads only on request.
- **Proactive:** Ask for missing info conversationally, one concern at a time.

### Repeated Failures

If the same task fails 3+ times (e.g. strategy validation keeps failing, backtest keeps erroring), stop and:

1. Summarize what was tried and what failed
2. Suggest a simpler approach or different parameters
3. If the issue appears to be model capability (complex multi-instrument logic), suggest switching to a more capable model for strategy generation

## Workflow

```
1. Onboard          →  POST /v3/account/onboard (once) — wallet + key auto-provisioned
2. Fund wallet      →  User sends USDC on Polygon (only manual step)
3. Discover markets →  POST /v3/markets/search — find candidate market slugs matching the user's interest; pass exact Polymarket event URLs directly when the user provides one
4. Write strategy   →  Author NautilusTrader Python strategy code from the closest archetype
5. Backtest         →  POST /v3/backtest with `strategyId`, `strategySource`, and `strategyConfig`
6. Persist          →  POST /v3/strategy only when saving for deployment/reuse
7. Review results   →  Analyze performance; iterate or proceed
8. Deploy           →  POST /v3/deployment → confirm with user → start
9. Monitor          →  Positions, P&L, deployment status
10. Exit            →  POST /v3/deployment/{id}/exit when done
```

Steps 1–2 happen once. Steps 3–10 are fully automated by the agent (with user confirmation before step 8's start).

### Strategy Archetypes

When a user asks for a Polymarket strategy, pick an archetype first and then generate strategy code from it. This keeps backtest assumptions explicit and reduces silent drift.

- [Probability Momentum](strategies/probability-momentum.md) — momentum, breakout, fast reaction in active markets
- [Probability Mean Reversion](strategies/probability-mean-reversion.md) — overreaction fade and range-like behavior
- [Deadline Drift](strategies/deadline-drift.md) — time-to-resolution behavior, especially before-date markets
- [Related-Market Spread](strategies/related-market-spread.md) — relative-value checks across linked markets
- [Large-Fill Pressure](strategies/large-fill-pressure.md) — repeated oversized fills with directional follow-through
- [Catalyst Confirmation](strategies/catalyst-confirmation.md) — event thesis with market confirmation first

Operational rules:

1. Pick the closest archetype before coding.
2. Treat these as starting points, not validated strategies.
3. Generate a custom NautilusTrader strategy from the selected archetype and pass it directly to `POST /v3/backtest` via `strategySource` with matching `strategyConfig`.
4. Before any backtest request, confirm exact market slugs from search candidates.
5. Use filled-data assumptions (`TradeTick`) in backtests; do not promise queue/maker behavior without matching evidence.

Filled-data rule: Polymarket strategy logic should be driven by historical `TradeTick` replay in backtesting. If logic is quote-only, flag it as likely non-tradable in current backtest mode.

### Pre-Deployment Checklist (MANDATORY)

Before `PUT /v3/deployment/{id}/status` → `{"action":"start"}`:

1. **Strategy is valid** — `GET /v3/strategy/{id}` → `status: "valid"`. Invalid strategies are rejected at deployment creation with `422`.
2. **Backtest reviewed** — at least one completed backtest for this strategy, results shown to the user.
3. **Wallet funded** — `GET /v3/account/wallets` → confirm the deployment's wallet has enough `usdc` for the strategy's trade size. The platform does not pre-check balance; an unfunded wallet leads to rejected orders at the venue.
4. **Market selected** — `POST /v3/markets/search` → confirm the exact `slug` to use. If the user gives a Polymarket event URL, pass that URL as the search query so child markets can be expanded. If search returns multiple plausible candidates, show the candidate questions/slugs and ask the user to choose. For backtests, require `backtestSupported: true`, `coverageStatus: "available"`, and a requested timerange inside the candidate `coverage`.
5. **User confirmation** — show the deployment summary and get an explicit "yes".

Do NOT skip any step or assume it passed without the API call.

## API Reference

### Account

#### POST `/v3/account/onboard` — Onboard (public)

Creates a user, API key, and default wallet in one call. See [Setup](#setup) for request/response.

Idempotency note: calling onboard again creates a **new** user and key. To add keys or wallets to an existing account, use the authenticated endpoints below.

#### GET `/v3/account` — Account Overview

```json
{
  "user_id": "user_abc123",
  "wallets": [
    {
      "wallet_id": "wal_abc123",
      "label": "default",
      "deposit_address": "0x1234...abcd",
      "approvals_set": true,
      "clob_credentials": true
    }
  ],
  "active_deployments": 2,
  "builder_code": "0x9132...",
  "gasless": true
}
```

Note: this endpoint does **not** return balances. Use `GET /v3/account/wallets` for balances.

#### POST `/v3/account/wallets` — Create Wallet

```json
// Request
{ "label": "btc-carry" }

// Response
{
  "wallet_id": "wal_def456",
  "label": "btc-carry",
  "deposit_address": "0x5678...efgh",
  "builder_code": "0x9132...",
  "gasless": true,
  "message": "New wallet created. Send USDC to deposit_address on Polygon."
}

// Error (400) — wallet limit reached
{ "error": "Maximum 3 wallets per user" }
```

#### GET `/v3/account/wallets` — List Wallets with Balances

```json
{
  "wallets": [
    {
      "wallet_id": "wal_abc123",
      "label": "default",
      "deposit_address": "0x1234...abcd",
      "balances": { "usdc": "150.25", "pol": "0" },
      "approvals_set": true,
      "clob_credentials": true,
      "created_at": "2026-06-01T10:00:00Z"
    }
  ]
}
```

#### GET `/v3/account/wallets/{id}` — Wallet Detail

Same fields as the list entry, plus `signature_type`.

#### POST `/v3/account/deposit/polymarket` — Get Deposit Instructions

Takes no amount — returns where to send funds. Deposits credit automatically once the transfer lands.

```json
// Response
{
  "deposit_address": "0x1234...abcd",
  "chain": "polygon",
  "accepted_assets": ["pUSD", "USDC", "USDC.e"],
  "gasless": true,
  "message": "Send pUSD or USDC to deposit_address on Polygon. No gas needed."
}
```

There is no deposit status endpoint — verify arrival by polling `GET /v3/account/wallets` for the updated balance.

#### POST `/v3/account/withdraw/polymarket` — Withdraw (NOT YET LIVE)

Accepts `{ "amount": 50.0, "to_address": "0x..." }` but currently returns a pending stub:

```json
{ "status": "pending", "amount_usdc": 50, "message": "Withdrawal via relayer — implementation pending." }
```

Do not present this as a working withdrawal.

#### POST `/v3/account/setup-polymarket` — Re-run Provisioning

Ensures the default wallet exists with approvals and CLOB credentials. Useful if onboarding was interrupted.

```json
// Response
{
  "status": "completed",
  "wallet_id": "wal_abc123",
  "deposit_wallet": "0x1234...abcd",
  "approvals_set": true,
  "clob_credentials": true,
  "builder_code": "0x9132..."
}
```

> `POST /v3/account/keys`, `GET /v3/account/keys`, and `POST /v3/account/claim` are platform-internal endpoints used by the Superior Trade web terminal (Privy account linking). Agents should not need them.

### Markets

#### POST `/v3/markets/search` — Search Polymarket Markets

Use this before any Polymarket backtest when the user describes a market in natural language ("BTC 120k before July", "Trump Greenland before 2027", "Fed 50 bps cut") or provides a Polymarket event URL such as `https://polymarket.com/event/world-cup-winner`. The endpoint returns candidates, not a single guaranteed resolution.

```json
// Request
{
  "query": "https://polymarket.com/event/world-cup-winner",
  "limit": 10
}
```

**Fields:** `query` is required. It can be natural language, an exact market slug, or a Polymarket event URL. `limit` is optional, defaults to 10, and must be between 1 and 50.

**Response:**

```json
{
  "status": "candidates_found",
  "candidates": [
    {
      "slug": "will-bitcoin-hit-120000-before-july-1",
      "question": "Will Bitcoin hit $120,000 before July 1?",
      "tradeCount": 12345,
      "liquidity": 240000.5,
      "volume": 1800000.25,
      "volume24h": 95000.1,
      "coverage": {
        "start": "2026-05-01",
        "end": "2026-06-17"
      },
      "dataMode": "fills_only",
      "coverageStatus": "available",
      "backtestSupported": true,
      "deploymentSupported": false,
      "matchedKeywords": ["bitcoin", "120000", "july"]
    }
  ]
}
```

`status: "not_found"` with an empty `candidates` array means no matching market was found. Do not invent a slug.

Search rules:

- If the user provides a Polymarket event URL, pass the full URL as `query`; do not manually strip it or guess the child market.
- Event URLs can expand into child markets. Example: `https://polymarket.com/event/world-cup-winner` can return country-specific markets such as `will-portugal-win-the-2026-fifa-world-cup-912`.
- If the user says "Ronaldo World Cup winner", search first; do not claim no market exists just because there is no exact Ronaldo-title market. The relevant candidate may be a Portugal World Cup market.
- Pass the candidate `slug` exactly as returned into `marketSlugs[]` for `POST /v3/backtest`.
- If more than one candidate is plausible, ask the user to choose by question/slug before backtesting or deploying.
- For filled-data backtests, prefer candidates with `coverageStatus: "available"` and `backtestSupported: true`.
- If `coverageStatus` is `"metadata_only"`, the market is known but historical filled data is not hydrated yet; explain that it cannot be backtested until data coverage is available.
- `deploymentSupported` is intentionally separate from `backtestSupported`; do not assume a market is deployable just because filled-data backtesting is available.
- Current market search does **not** return order book depth, outcome token IDs, current prices, or prebuilt `instrument_id`s.

#### Instrument ID Format

Each outcome (Yes/No) is a separate tradeable instrument:

```
{condition_id}-{token_id}.POLYMARKET
```

Do not derive or guess `condition_id` / `token_id` values from search candidates. Market search is for finding the exact `slug` and data coverage. If a strategy requires explicit instruments, use values from a validated market-detail/orderbook source when that API is available, or ask the user for the exact instrument.

### Strategy

#### POST `/v3/strategy` — Create + Validate

```json
// Request
{
  "code": "from nautilus_trader.trading import Strategy\n\nclass MyStrategyConfig...",
  "config": {
    "instrument_id": "0xdd22472e...-21742633....POLYMARKET",
    "trade_size": 10.0
  },
  "venue": "polymarket"
}
```

`code` is required. `config` is the constructor kwargs for the strategy's config class. `venue` defaults to `"polymarket"`.

```json
// Response (200) — valid
{
  "id": "strat_abc123",
  "status": "valid",
  "strategy_class": "MyStrategy",
  "config_class": "MyStrategyConfig"
}

// Response (422) — invalid (record is still created, with the errors stored)
{
  "id": "strat_abc123",
  "status": "invalid",
  "errors": [
    { "type": "syntax", "message": "Line 15: unexpected indent" },
    { "type": "import", "message": "Blocked import: 'os' at line 2 (not in allowlist)" },
    { "type": "sandbox", "message": "Blocked name: 'eval' at line 30" }
  ]
}
```

**Validation is allowlist-based.** Only these imports are permitted:

- `nautilus_trader.*` (config, model, trading, etc.)
- Standard library: `decimal`, `math`, `json`, `datetime`, `collections`, `dataclasses`, `enum`, `functools`, `itertools`, `typing`, `abc`, `numbers`, `operator`, `statistics`, `time`
- `numpy`, `pandas`

Additionally blocked anywhere in the code: `eval`, `exec`, `open`, `getattr`/`setattr`, `globals`, `__import__`, dunder attribute access (`__class__`, `__globals__`, ...), `chr()`, and suspicious path strings. The code must contain exactly one class inheriting from `Strategy`.

#### GET `/v3/strategy` — List My Strategies

```json
{
  "strategies": [
    {
      "id": "strat_abc123",
      "status": "valid",
      "strategy_class": "Carry",
      "config_class": "CarryConfig",
      "venue": "polymarket",
      "created_at": "2026-06-02T10:00:00Z"
    }
  ]
}
```

#### GET `/v3/strategy/{id}` — Strategy Detail

Returns `id`, `status`, `strategy_class`, `config_class`, `venue`, `config`, `errors`, `created_at`.

#### DELETE `/v3/strategy/{id}`

Response: `{ "deleted": true }`. Returns `403` if the strategy belongs to another user.

### Backtest

#### POST `/v3/backtest` — Create and Run

```json
// Request
{
  "tenantId": "tenant_a",
  "backtestId": "bt_xyz789",
  "strategyId": "strat_abc123",
  "strategySource": "from nautilus_trader.config import StrategyConfig\nfrom nautilus_trader.model.data import TradeTick\n...",
  "strategyConfig": {
    "order_size": 10,
    "lookback_ticks": 20
  },
  "marketSlugs": ["will-donald-trump-win-the-2024-us-presidential-election"],
  "startingBalance": 1000,
  "timerange": { "start": "2024-10-01", "end": "2024-11-06" },
  "venueProfile": "polymarket-london"
}

// Response — planned/queued
{
  "tenantId": "tenant_a",
  "backtestId": "bt_xyz789",
  "strategyId": "strat_abc123",
  "status": "queued",
  "resultUri": "gs://.../backtest-results/polymarket/tenant=tenant_a/date=2026-06-17/bt_xyz789.json"
}
```

- `tenantId`, `backtestId`, `strategyId`, `marketSlugs`, `timerange`, and `venueProfile` are required.
- For generated custom strategies, use `strategyId` as the strategy/run identifier and include `strategySource` + `strategyConfig`. If `strategySource` is omitted, the API tries the known template matching `strategyId`.
- If the custom strategy config declares `instrument_id`, the runner injects the primary market outcome instrument when `strategyConfig.instrument_id` is omitted. Only provide explicit instrument IDs when the user or a market-detail source gives exact values.
- Use exact `slug` values returned by `POST /v3/markets/search` as `marketSlugs[]`.
- `startingBalance` defaults to 1000 pUSD.
- `timerange` must fit inside the candidate coverage returned by market search. If data coverage is missing, the API returns `409` with a blocked reason instead of pretending the backtest ran.
- The engine loads historical **trade ticks** for each market slug. Filled-data backtests cannot prove order-book queue priority, spread capture, or exact partial-fill realism.

> **Backtests replay historical trade ticks.** Strategies that only subscribe to quote ticks (`subscribe_quote_ticks`) may receive no callbacks in a backtest. For backtestable strategies, drive logic from `on_trade_tick` (or subscribe to both and let live trading benefit from quotes).

#### GET `/v3/backtest/{id}/status` — Poll Status

Poll until `status` is `completed` or `failed`. Typical run takes 1–30 seconds depending on data size.

**Statuses:** `pending` → `running` → `completed` | `failed`

```json
{
  "id": "bt_xyz789",
  "status": "completed",
  "resultUrl": "gs://.../backtest-results/polymarket/tenant=tenant_a/date=2026-06-17/bt_xyz789.json",
  "k8sJobName": "polymarket-backtest-bt-xyz789"
}
```

#### GET `/v3/backtest/{id}` — Persisted Record

Returns the persisted backtest record and current status.

#### GET `/v3/backtest/{id}/result` — Uploaded Result

Returns `202` while the result is not ready, then `200` when the runner has uploaded JSON to GCS.

```json
// Response (completed)
{
  "backtest_id": "bt_xyz789",
  "status": "completed",
  "resultUri": "gs://.../backtest-results/polymarket/tenant=tenant_a/date=2026-06-17/bt_xyz789.json",
  "result": {
    "starting_balance": 1000,
    "ending_balance": 1087.5,
    "total_pnl": 87.5,
    "total_return_pct": 8.75,
    "total_orders": 24,
    "total_fills": 24,
    "dataset_provenance": {
      "dataset_version": "20260617T033000Z-backtest-ready-trade-ticks-compatible",
      "source": "persisted_dataset",
      "total_ticks": 63626
    },
    "trades": [
      {
        "instrument": "0xdd22472e...-21742633....POLYMARKET",
        "side": "BUY",
        "quantity": 19.23,
        "price": 0.52,
        "timestamp": "2024-10-15T14:22:00Z"
      }
    ],
    "positions": [
      {
        "instrument": "0xdd22472e...-21742633....POLYMARKET",
        "side": "LONG",
        "entry_price": 0.52,
        "exit_price": 1.00,
        "pnl": 48.0
      }
    ]
  },
  "message": null
}

// Response (failed)
{ "backtest_id": "bt_xyz789", "status": "failed", "message": "No data loaded for slugs: invalid-market-slug" }
```

Balances are in pUSD. The `trades` array is capped at the first 50 fills. Win rate, Sharpe, and drawdown are not computed server-side — derive them from `trades`/`positions` if the user asks.

#### GET `/v3/backtest/{id}/logs` — Runtime Logs

Use this when status is `failed`, stuck, or the user asks what happened. Logs come from GCP Logging for the Kubernetes Job.

#### Result Interpretation

Before suggesting deployment, always run a backtest first. If the backtest produced **zero fills** over a period that should have generated signals, do not offer deployment — the strategy, market slug, or instrument_id likely has an issue. If PnL is **negative**, note the timerange may be unsuitable but don't dismiss the strategy outright. If PnL is **positive**, present results without overpromising — strong backtest fit on a single resolved market is weak evidence. Stay neutral and let the user decide.

### Deployment

#### POST `/v3/deployment` — Create

```json
// Request — wallet_id optional, defaults to the user's default wallet
{ "strategy_id": "strat_abc123", "wallet_id": "wal_abc123" }

// Response
{ "id": "dep_abc123", "wallet_id": "wal_abc123", "status": "created" }
```

Errors: `400` missing strategy_id or no wallet, `404` strategy/wallet not found, `422` strategy not valid.

#### GET `/v3/deployment` — List My Deployments

```json
{
  "deployments": [
    { "id": "dep_abc123", "strategy_id": "strat_abc123", "wallet_id": "wal_abc123", "status": "running", "created_at": "..." }
  ]
}
```

#### PUT `/v3/deployment/{id}/status` — Start or Stop

```json
// Request
{ "action": "start" }   // or "stop"

// Response (start)
{ "id": "dep_abc123", "status": "running", "method": "process", "pid": 12345, "podName": null }

// Response (stop)
{ "id": "dep_abc123", "status": "stopped" }
```

- Starting auto-generates CLOB credentials for the deployment's wallet if needed.
- `409` if already running; `500` with `{ "error": "..." }` if the engine fails to start (deployment is marked `failed` with the error stored).

**Statuses:** `created` → `running` → `stopped` | `failed`

#### POST `/v3/deployment/{id}/exit` — Emergency Exit

Stops the deployment and closes positions immediately.

```json
{ "id": "dep_abc123", "status": "stopped", "message": "Emergency exit — all positions closed" }
```

Before calling this, show the user the open positions (`GET /v3/deployment/{id}/positions`) and get confirmation — exits fill at market price and are irreversible.

#### GET `/v3/deployment/{id}` — Detail

```json
{
  "id": "dep_abc123",
  "strategy_id": "strat_abc123",
  "status": "running",
  "positions": [ ... ],
  "pnl": { ... },
  "error": null,
  "created_at": "..."
}
```

The server reconciles status on read: if a deployment is marked `running` but its process has died, this call flips it to `stopped`. Always trust the status from the latest GET, not a cached value.

#### GET `/v3/deployment/{id}/positions` — Positions

```json
{
  "positions": [
    {
      "instrument_id": "0xdd22472e...-21742633....POLYMARKET",
      "side": "LONG",
      "quantity": 100.0,
      "entry_price": 0.45,
      "current_price": 0.52,
      "unrealized_pnl": 7.0
    }
  ]
}
```

#### GET `/v3/deployment/{id}/pnl` — P&L

```json
{
  "realized_pnl": 45.20,
  "unrealized_pnl": 7.0,
  "total_pnl": 52.20,
  "total_orders": 12,
  "total_fills": 12
}
```

Returns zeros for all fields if the deployment hasn't reported yet.

### Shared API Notes

- **Auth errors (401):** `{ "error": "Unauthorized", "message": "Include 'Authorization: Bearer <api_key>' header. Get a key via POST /v3/account/onboard." }`
- **Not found (404):** `{ "error": "Strategy not found" }` (resource name varies)
- All timestamps are **UTC (ISO8601)**. Convert to the user's local timezone when presenting times conversationally.

## Strategy Authoring (NautilusTrader)

Write a NautilusTrader `Strategy` subclass in Python. The strategy receives real-time market data and submits orders. Remember the import allowlist (see [Strategy validation](#post-v3strategy--create--validate)) — `nautilus_trader`, common stdlib, `numpy`, `pandas` only.

### Strategy Structure

```python
from nautilus_trader.config import StrategyConfig
from nautilus_trader.model.data import QuoteTick, TradeTick, Bar, BarType
from nautilus_trader.model.enums import OrderSide, TimeInForce
from nautilus_trader.model.events import OrderFilled, PositionOpened, PositionClosed
from nautilus_trader.model.identifiers import InstrumentId
from nautilus_trader.trading import Strategy


class MyStrategyConfig(StrategyConfig, frozen=True):
    instrument_id: str          # Polymarket instrument ID
    # Add strategy-specific parameters here
    trade_size: float = 10.0    # pUSD per trade


class MyStrategy(Strategy):

    def __init__(self, config: MyStrategyConfig):
        super().__init__(config)
        self.instrument_id = InstrumentId.from_str(config.instrument_id)

    def on_start(self):
        """Called when strategy starts. Subscribe to data here."""
        self.subscribe_trade_ticks(self.instrument_id)
        # also available: self.subscribe_quote_ticks(...) — live only, see note below
        # or: self.subscribe_bars(BarType.from_str("..."))

    def on_quote_tick(self, tick: QuoteTick):
        """Called on every bid/ask update (live trading)."""
        bid = float(tick.bid_price)
        ask = float(tick.ask_price)

    def on_trade_tick(self, tick: TradeTick):
        """Called on every trade execution in the market."""
        price = float(tick.price)
        size = float(tick.size)

    def on_bar(self, bar: Bar):
        """Called on every bar close (if subscribed to bars)."""
        close = float(bar.close)

    def on_order_filled(self, event: OrderFilled):
        """Called when your order is filled."""
        pass

    def on_position_opened(self, event: PositionOpened):
        pass

    def on_position_closed(self, event: PositionClosed):
        pass

    def on_order_rejected(self, event):
        """Called when an order is rejected by venue or risk engine."""
        self.log.warning(f"Order rejected: {event.reason}")

    def on_stop(self):
        """Called when strategy stops. Clean up here."""
        self.cancel_all_orders(self.instrument_id)
```

> **Backtest compatibility:** backtests replay historical **trade ticks**. A strategy whose logic lives entirely in `on_quote_tick` will do nothing in a backtest. Drive backtestable logic from `on_trade_tick`, or subscribe to both.

### Submitting Orders

**Market order (BUY — quote quantity in pUSD):**

```python
instrument = self.cache.instrument(self.instrument_id)
order = self.order_factory.market(
    instrument_id=self.instrument_id,
    order_side=OrderSide.BUY,
    quantity=instrument.make_qty(10.0),   # 10 pUSD
    time_in_force=TimeInForce.IOC,
    quote_quantity=True,                  # BUY uses pUSD amount
)
self.submit_order(order)
```

**Market order (SELL — base quantity in shares):**

```python
order = self.order_factory.market(
    instrument_id=self.instrument_id,
    order_side=OrderSide.SELL,
    quantity=instrument.make_qty(25.0),   # 25 shares
    time_in_force=TimeInForce.IOC,
)
self.submit_order(order)
```

**Limit order (GTC):**

```python
order = self.order_factory.limit(
    instrument_id=self.instrument_id,
    order_side=OrderSide.BUY,
    quantity=instrument.make_qty(10.0),
    price=instrument.make_price(0.45),
    time_in_force=TimeInForce.GTC,
    post_only=True,                       # maker only
)
self.submit_order(order)
```

### Key Rules

- **Market BUY**: use `quote_quantity=True` — quantity is the pUSD amount to spend
- **Market SELL**: use base quantity — quantity is the number of shares to sell
- **Limit orders (both sides)**: always base quantity (shares), never `quote_quantity`
- Market orders require `TimeInForce.IOC` (maps to Polymarket FAK)
- Limit orders support `GTC`, `GTD`, `FOK`, `IOC`
- Prices are 0.001 to 0.999 (probability, in pUSD)
- Use `self.cache.instrument(id)` to get the instrument for `make_qty()` and `make_price()`
- `Position.quantity` is always positive (unsigned size)
- Handle `on_order_rejected(self, event)` — orders can be rejected by the venue or risk engine
- Minimum order size: 5 pUSD (market BUY) or 5 shares (limit/sell)
- Polymarket rate limits: 30 order submissions/minute, 100 data requests/minute per user

### Converting Dollars to Shares for Limit Orders

Limit orders use share quantities, not pUSD:

```python
shares = dollar_amount / price
# e.g., $50 at price 0.25 = 200 shares
```

### Multi-Instrument Strategies

A single strategy can subscribe to and trade multiple instruments:

```python
class MultiOutcomeConfig(StrategyConfig, frozen=True):
    instrument_ids: list[str]
    trade_size: float = 10.0

class MultiOutcome(Strategy):
    def __init__(self, config: MultiOutcomeConfig):
        super().__init__(config)
        self.ids = [InstrumentId.from_str(i) for i in config.instrument_ids]

    def on_start(self):
        for iid in self.ids:
            self.subscribe_trade_ticks(iid)

    def on_trade_tick(self, tick: TradeTick):
        # tick.instrument_id tells you which instrument this tick is for
        if tick.instrument_id == self.ids[0]:
            pass  # handle first outcome
        elif tick.instrument_id == self.ids[1]:
            pass  # handle second outcome
```

For multi-outcome markets (e.g. Fed rate: hold / 25bp cut / 50bp cut), pass all outcome instrument_ids and trade across them in one strategy. Each deployment runs one strategy instance — no need for separate deployments per outcome.

### Available Data

- `self.cache.quote_tick(instrument_id)` — latest bid/ask
- `self.cache.instrument(instrument_id)` — instrument details
- `self.cache.positions(instrument_id=id)` — open positions for an instrument
- `self.clock.utc_now()` — current UTC timestamp
- `self.portfolio` — account balances and positions

## Example Strategies

### Carry / Yield Harvesting

Buy high-probability outcomes near resolution. The "T-Bill" of prediction markets. Resolution risk applies: a 0.95 outcome that resolves NO loses everything staked.

```python
from nautilus_trader.config import StrategyConfig
from nautilus_trader.model.data import TradeTick
from nautilus_trader.model.enums import OrderSide, TimeInForce
from nautilus_trader.model.identifiers import InstrumentId
from nautilus_trader.trading import Strategy


class CarryConfig(StrategyConfig, frozen=True):
    instrument_id: str
    min_probability: float = 0.90
    max_probability: float = 0.99
    trade_size_pusd: float = 50.0


class Carry(Strategy):
    def __init__(self, config: CarryConfig):
        super().__init__(config)
        self.instrument_id = InstrumentId.from_str(config.instrument_id)
        self._has_position = False

    def on_start(self):
        self.subscribe_trade_ticks(self.instrument_id)

    def on_trade_tick(self, tick: TradeTick):
        if self._has_position:
            return
        price = float(tick.price)
        if self.config.min_probability <= price <= self.config.max_probability:
            instrument = self.cache.instrument(self.instrument_id)
            order = self.order_factory.market(
                instrument_id=self.instrument_id,
                order_side=OrderSide.BUY,
                quantity=instrument.make_qty(self.config.trade_size_pusd),
                time_in_force=TimeInForce.IOC,
                quote_quantity=True,
            )
            self.submit_order(order)

    def on_position_opened(self, event):
        self._has_position = True

    def on_position_closed(self, event):
        self._has_position = False

    def on_stop(self):
        self.cancel_all_orders(self.instrument_id)
```

### Momentum / News Fade

Buy when probability drops sharply (overreaction), sell when it recovers.

```python
from nautilus_trader.config import StrategyConfig
from nautilus_trader.model.data import TradeTick
from nautilus_trader.model.enums import OrderSide, TimeInForce
from nautilus_trader.model.identifiers import InstrumentId
from nautilus_trader.trading import Strategy


class MomentumFadeConfig(StrategyConfig, frozen=True):
    instrument_id: str
    lookback_ticks: int = 20
    drop_threshold: float = -0.05
    recovery_threshold: float = 0.02
    trade_size_pusd: float = 25.0


class MomentumFade(Strategy):
    def __init__(self, config: MomentumFadeConfig):
        super().__init__(config)
        self.instrument_id = InstrumentId.from_str(config.instrument_id)
        self._prices = []
        self._has_position = False

    def on_start(self):
        self.subscribe_trade_ticks(self.instrument_id)

    def on_trade_tick(self, tick: TradeTick):
        price = float(tick.price)
        self._prices.append(price)
        if len(self._prices) > self.config.lookback_ticks:
            self._prices.pop(0)
        if len(self._prices) < self.config.lookback_ticks:
            return

        recent_return = (price - self._prices[0]) / self._prices[0]
        instrument = self.cache.instrument(self.instrument_id)

        if not self._has_position and recent_return < self.config.drop_threshold:
            order = self.order_factory.market(
                instrument_id=self.instrument_id,
                order_side=OrderSide.BUY,
                quantity=instrument.make_qty(self.config.trade_size_pusd),
                time_in_force=TimeInForce.IOC,
                quote_quantity=True,
            )
            self.submit_order(order)

        elif self._has_position and recent_return > self.config.recovery_threshold:
            positions = self.cache.positions(instrument_id=self.instrument_id)
            for pos in positions:
                if pos.is_open:
                    order = self.order_factory.market(
                        instrument_id=self.instrument_id,
                        order_side=OrderSide.SELL,
                        quantity=instrument.make_qty(float(pos.quantity)),
                        time_in_force=TimeInForce.IOC,
                    )
                    self.submit_order(order)

    def on_position_opened(self, event):
        self._has_position = True

    def on_position_closed(self, event):
        self._has_position = False

    def on_stop(self):
        self.cancel_all_orders(self.instrument_id)
```

### Spread Capture / Market Making

Live-only reference. Do **not** use this as a filled-data backtest template: it needs quote/order-book state, queue assumptions, and spread realism that current Polymarket backtests do not replay. Watch the rate limit: cancelling and re-quoting on every tick can exceed 30 orders/minute on active markets.

```python
from nautilus_trader.config import StrategyConfig
from nautilus_trader.model.data import QuoteTick
from nautilus_trader.model.enums import OrderSide, TimeInForce
from nautilus_trader.model.events import OrderFilled
from nautilus_trader.model.identifiers import InstrumentId
from nautilus_trader.trading import Strategy


class SpreadCaptureConfig(StrategyConfig, frozen=True):
    instrument_id: str
    spread_width: float = 0.02
    order_size: float = 10.0
    max_inventory: float = 100.0


class SpreadCapture(Strategy):
    def __init__(self, config: SpreadCaptureConfig):
        super().__init__(config)
        self.instrument_id = InstrumentId.from_str(config.instrument_id)
        self._net_qty = 0.0

    def on_start(self):
        self.subscribe_quote_ticks(self.instrument_id)

    def on_quote_tick(self, tick: QuoteTick):
        self.cancel_all_orders(self.instrument_id)
        if abs(self._net_qty) >= self.config.max_inventory:
            return

        bid = float(tick.bid_price)
        ask = float(tick.ask_price)
        mid = (bid + ask) / 2
        half = self.config.spread_width / 2
        our_bid = mid - half
        our_ask = mid + half

        if our_bid <= 0 or our_ask >= 1:
            return

        instrument = self.cache.instrument(self.instrument_id)

        bid_order = self.order_factory.limit(
            instrument_id=self.instrument_id,
            order_side=OrderSide.BUY,
            quantity=instrument.make_qty(self.config.order_size),
            price=instrument.make_price(our_bid),
            time_in_force=TimeInForce.GTC,
            post_only=True,
        )
        self.submit_order(bid_order)

        ask_order = self.order_factory.limit(
            instrument_id=self.instrument_id,
            order_side=OrderSide.SELL,
            quantity=instrument.make_qty(self.config.order_size),
            price=instrument.make_price(our_ask),
            time_in_force=TimeInForce.GTC,
            post_only=True,
        )
        self.submit_order(ask_order)

    def on_order_filled(self, event: OrderFilled):
        qty = float(event.last_qty)
        if event.order_side == OrderSide.BUY:
            self._net_qty += qty
        else:
            self._net_qty -= qty

    def on_stop(self):
        self.cancel_all_orders(self.instrument_id)
```

## Troubleshooting

### Strategy Validation Failures

- **`type: "syntax"`** — Python syntax error; the message includes the line number.
- **`type: "import"`** — an import outside the allowlist. Rewrite using only `nautilus_trader`, the permitted stdlib modules, `numpy`, or `pandas`.
- **`type: "sandbox"`** — blocked construct (`eval`, `open`, dunder access, `chr()`, etc.). Remove it; there is no override.
- **`type: "structure"`** — no class inheriting from `Strategy` was found. Exactly one is required.

### Backtest Failures

- `status: "failed"` with `message: "No data loaded for slugs: ..."` — the market slug is wrong or has no hydrated fill history. Re-check the exact `slug` from `POST /v3/markets/search` and require `coverageStatus: "available"`.
- `message: "Process exited with code N"` — the strategy crashed at runtime. Common causes: config kwargs that don't match the config class fields, or referencing an instrument the engine didn't load.
- **Zero fills** — the strategy's conditions never triggered, or it only subscribes to quote ticks (backtests replay trade ticks only).

### Deployment Issues

- **Start returns 500** — the engine failed to launch; the deployment is marked `failed` and the message is stored in the `error` field of `GET /v3/deployment/{id}`.
- **Status flips from running to stopped on read** — the server reconciles status against the actual process on every GET. If this happens unexpectedly, the strategy likely crashed; check the deployment's `error` field.
- **Orders rejected at the venue** — usually an unfunded wallet (check `GET /v3/account/wallets`), an order below the 5 pUSD / 5 share minimum, or a market that has resolved or closed.
- **Rate limits** — if order submissions exceed ~30/minute, the venue throttles. Reduce re-quote frequency (e.g. only re-quote when the mid moves more than a threshold) rather than rapid stop/start cycles.

### Diagnosing Zero-Trade Deployments

Check in order:

1. **Wallet balance** — `GET /v3/account/wallets` for the deployment's wallet
2. **Deployment status** — `GET /v3/deployment/{id}` actually `running` with no `error`
3. **Market still active** — not resolved, `end_date` in the future
4. **Strategy conditions** — are entry thresholds reachable at current prices? (e.g. a carry strategy with `min_probability: 0.90` does nothing while the market trades at 0.60)
5. **Order minimums** — trade size at least 5 pUSD (market BUY) / 5 shares (limit/sell)
