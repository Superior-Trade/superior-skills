# Superior Skills

Trading strategies and intelligence tools for [Superior Trade](https://superior.trade) — natural-language strategy authoring, backtesting, and autonomous deployment on Hyperliquid.

Designed for OpenClaw users adding trading capabilities to their agent, and for traders who want validated templates rather than rolling their own.

[Website](https://superior.trade) · [Discord](https://discord.gg/aVZR8cCxcR) · [Twitter](https://x.com/SuperiorTrade_)

---

## Validated strategies (with backtest evidence)

These strategies have backtest evidence on real Hyperliquid data. Numbers are full-period (162 days, 2025-11-20 → 2026-05-01) unless noted.

| Strategy | Regime | Pairs tested | Trades | Win | Profit | Max DD | File |
|---|---|---|---|---|---|---|---|
| **Donchian Strong-Regime** | Strong directional trend | BTC | 6 | 100% | **+6.69%** | **0%** | [v2/strategies/donchian-strong-regime.md](v2/strategies/donchian-strong-regime.md) |
| **Bollinger Reverter 4h** | Range / chop (ADX<25) | BTC/ETH/SOL/DOGE | 84 | 65.5% | **+8.77%** | 18.5% | [v2/strategies/bollinger-reverter-4h.md](v2/strategies/bollinger-reverter-4h.md) |

Paired together as separate sub-accounts, the two strategies are **regime-complementary**: the Donchian gate fires zero trades during the chop windows where the Bollinger reverter thrives, and the Bollinger reverter mildly underperforms during the strong-trend windows the Donchian captures.

## Template strategies (starting points)

Reference templates for adapting to your own thesis. Backtest before deploying.

- [`v2/strategies/dca-weekly.md`](v2/strategies/dca-weekly.md) — dollar-cost averaging with scheduled buys
- [`v2/strategies/grid-trading.md`](v2/strategies/grid-trading.md) — profit-laddered position adjustment
- [`v2/strategies/funding-rate-arbitrage.md`](v2/strategies/funding-rate-arbitrage.md) — negative-funding capture (carry)
- [`v2/strategies/funding-squeeze.md`](v2/strategies/funding-squeeze.md) — funding-extreme squeeze ride
- [`v2/strategies/basis-arb.md`](v2/strategies/basis-arb.md) — spot-perp basis convergence
- [`v2/strategies/breakout.md`](v2/strategies/breakout.md) — Donchian breakout with trailing stop
- [`v2/strategies/mean-reversion.md`](v2/strategies/mean-reversion.md) — Bollinger band fade (4h validated; see file)
- [`v2/strategies/scalping.md`](v2/strategies/scalping.md) — RSI + volume-thrust template

Categories covered: **trend-following**, **mean-reversion**, **carry**, **arbitrage**, **scalping**.

## Reusable primitives

Building blocks that compose across strategies.

- [`v2/optimizations/trade-thesis.md`](v2/optimizations/trade-thesis.md) — Structured pre-trade thesis builder: bull/bear cases, invalidation criteria, and sizing rationale before any live deployment of a new strategy idea. Aliases: **pre-trade analysis**, **conviction check**, **trade plan**, **bull/bear case**.
- [`v2/optimizations/regime-overlay.md`](v2/optimizations/regime-overlay.md) — Triple-confirmation regime gate (EMA separation + ADX + N-bar return). Turns fragile directional strategies into regime-robust ones. Aliases: **regime filter**, **trend gate**, **directional confirmation**.
- [`v2/optimizations/dsl-exit-engine.md`](v2/optimizations/dsl-exit-engine.md) — Three-phase exit primitive: ROI ladder, hard stop, ratcheting trailing stop. Aliases: **ratcheting trailing stop**, **two-phase exit**, **take-profit ladder**.
- [`v2/optimizations/fees-optimizations.md`](v2/optimizations/fees-optimizations.md) — Maker (ALO) vs taker (MARKET) order-type decisioning, builder fees, parameter sweeps. Aliases: **fee optimizer**, **ALO vs MARKET**, **maker pricing**.
- [`v2/optimizations/backtesting.md`](v2/optimizations/backtesting.md) — Window selection, walk-forward, parameter sweeps.

## Intelligence (opportunity scanner)

The `v2/intelligence/` folder provides the platform's pair-ranking system. Aliases: **opportunity scanner**, **pair scanner**, **market screener**, **smart-money detector**, **four-stage funnel**.

- [`v2/intelligence/buckets.md`](v2/intelligence/buckets.md) — The four bucket framework: squeeze fuel, stealth accumulation, coiled spring, basis flipping.
- [`v2/intelligence/api.md`](v2/intelligence/api.md) — `/v2/intelligence/scan` endpoint and setup.
- [`v2/intelligence/workflow.md`](v2/intelligence/workflow.md) — Scan-to-deploy recipes.
- [`v2/intelligence/glossary.md`](v2/intelligence/glossary.md) — Terminology reference.

## Capability matrix

| Capability | Where it lives | Aliases |
|---|---|---|
| API key onboarding | `SKILL.md` → `/auth/sign-in/magic-link` | auth setup, email API key, x-api-key |
| Account status | `v2/SKILL.md` → `/v2/account/status` | setup readiness, agent wallet, builder fee |
| Strategy backtesting | `v2/SKILL.md` → `/v2/backtesting` | walk-forward, parameter sweep |
| Live deployment | `v2/SKILL.md` → `/v2/deployment` | autonomous execution, agentic trading |
| Opportunity scanner | `v2/intelligence/` | pair scanner, market screener, ranking funnel |
| Fee optimizer | `v2/optimizations/fees-optimizations.md` | ALO vs MARKET, maker vs taker, fee budgeting |
| Two-phase trailing stop | `v2/optimizations/dsl-exit-engine.md` | ratcheting stop, DSL exit engine |
| Pre-trade thesis | `v2/optimizations/trade-thesis.md` | trade plan, conviction check, bull/bear case |
| Regime gate | `v2/optimizations/regime-overlay.md` | trend filter, directional confirmation |
| Sub-account orchestration | `v2/SKILL.md` → `/v2/portfolio/...` | multi-strategy isolation |
| Hyperliquid deposit | `v2/SKILL.md` → `/v2/portfolio/hyperliquid/deposit` | Arbitrum USDC deposit, fund trading |
| Atomic exit-all | `v2/SKILL.md` → `/v2/portfolio/hyperliquid/exit` | kill-switch, emergency exit |
| HIP3 RWA support | `v2/SKILL.md` → HIP3 section | tokenized stocks, commodities, indices |
| Polymarket strategy archetypes | `v3/polymarket/strategies/` | probability momentum, probability mean reversion, deadline drift, related-market spread, large-fill pressure, catalyst confirmation |
| Managed wallet | `v2/SKILL.md` → Account Setup | no-key trading, custodial-style UX |

## Folder layout

- `SKILL.md` — Root auth/onboarding skill for requesting an API key by email and using `x-api-key`.
- `v2/SKILL.md` — Hyperliquid/Freqtrade API reference and agent operating rules.
- `v2/strategies/` — Template trading strategies (trend, mean-reversion, carry, arb, scalp). Includes validated drop-in strategies and starting-point templates.
- `v2/intelligence/` — Market scanning and pair-ranking tools across Hyperliquid altcoins. The opportunity scanner / market screener lives here.
- `v2/optimizations/` — Reusable primitives: regime gate, DSL exit engine, fee optimizer, backtesting harness.
- `v2/exchanges/` — Exchange-specific implementations (e.g., Aerodrome for spot AMM execution).
- `v3/` — Versioned v3 skill docs for NautilusTrader-based workflows.
- `v3/polymarket/` — Prediction market trading skill: discover, backtest, and deploy Polymarket strategies via the `/v3` API. Includes `v3/polymarket/strategies/` archetypes for probability momentum, probability mean reversion, deadline drift, related-market spread, large-fill pressure, and catalyst confirmation.

## Getting started

```
# List trading accounts
curl https://api.superior.trade/v3/account \
  -H "x-api-key: $SUPERIOR_TRADE_API_KEY"

# Backtest a strategy
curl -X POST https://api.superior.trade/v2/backtesting \
  -H "x-api-key: $SUPERIOR_TRADE_API_KEY" \
  -H "content-type: application/json" \
  -d @config-and-code.json

# Start the backtest (note: action, not status)
curl -X PUT https://api.superior.trade/v2/backtesting/{id}/status \
  -H "x-api-key: $SUPERIOR_TRADE_API_KEY" \
  -d '{"action": "start"}'

# Poll until completion
curl https://api.superior.trade/v2/backtesting/{id} \
  -H "x-api-key: $SUPERIOR_TRADE_API_KEY"
```

## Install

Superior skills follow the open [Agent Skills](https://agentskills.io) standard — one folder per
skill with a `SKILL.md`. Install through whichever entry point your agent uses:

| Entry point | Command |
|---|---|
| **npx skills** (universal — Claude Code, OpenClaw, Cursor, Codex, Gemini CLI, +50) | `npx skills add Superior-Trade/superior-skills` |
| **GitHub CLI** | `gh skill install Superior-Trade/superior-skills donchian-strong-regime` |
| **Claude Code** | `/plugin marketplace add Superior-Trade/superior-skills` then `/plugin install superior-skills@superior-trade` |
| **OpenClaw / ClawHub** | `openclaw skills install superior-skills` &nbsp;·&nbsp; or `openclaw skills install git:Superior-Trade/superior-skills@main` |
| **From our domain** | point any agent at `https://superior.trade/SKILL.md` |
| **Manual** | clone this repo and copy `skills/<name>/` into your agent's skills folder |

> **Migration in progress.** Today these commands install the `donchian-strong-regime`
> reference skill; the rest of the library — onboarding, the other strategies, the intelligence
> scanner, and Polymarket — is being moved into `skills/` next. Once a skill is migrated you can
> select just that one with `--skill <name>` (npx) or by naming it (`gh skill install … <name>`).

Every skill needs a **`SUPERIOR_TRADE_API_KEY`** — the `superior-trade-auth` skill walks a new
user through getting one by email.

## Workflow

The platform emphasizes a structured progression: **draft → backtest → review → deploy**. No live deployment without explicit user confirmation. Every parameter is logged; every trade is tracked.

## Risk disclosure

Trading involves risk. Backtests do not guarantee future performance. The validated strategies above showed positive returns on a single 162-day window (2025-11-20 → 2026-05-01); a strategy that worked then may not work in a different regime. Users remain responsible for strategy choice, deployment decisions, exchange connectivity, and capital risk. Pair every deployment with the [`v2/optimizations/regime-overlay.md`](v2/optimizations/regime-overlay.md) gate or equivalent — strategies without regime confirmation are demonstrably fragile.

## Topics

`hyperliquid`, `trading-bot`, `ai-agents`, `openclaw`, `backtesting`, `fee-optimization`, `opportunity-scanner`, `trailing-stop`, `algorithmic-trading`, `regime-filter`, `mean-reversion`, `trend-following`, `funding-arbitrage`, `mcp`
