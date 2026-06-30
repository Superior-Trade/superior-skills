# Superior Skills

Trading strategies and intelligence tools for [Superior Trade](https://superior.trade) — natural-language strategy authoring, backtesting, and autonomous deployment on Hyperliquid.

Designed for OpenClaw users adding trading capabilities to their agent, and for traders who want validated templates rather than rolling their own.

[Website](https://superior.trade) · [Discord](https://discord.gg/aVZR8cCxcR) · [Twitter](https://x.com/SuperiorTrade_)

---

## Validated strategies (with backtest evidence)

These strategies have backtest evidence on real Hyperliquid data. Numbers are full-period (162 days, 2025-11-20 → 2026-05-01) unless noted.

| Strategy | Regime | Pairs tested | Trades | Win | Profit | Max DD | File |
|---|---|---|---|---|---|---|---|
| **Donchian Strong-Regime** | Strong directional trend | BTC | 6 | 100% | **+6.69%** | **0%** | [`donchian-strong-regime`](skills/donchian-strong-regime/SKILL.md) |
| **Bollinger Reverter 4h** | Range / chop (ADX<25) | BTC/ETH/SOL/DOGE | 84 | 65.5% | **+8.77%** | 18.5% | [`bollinger-reverter-4h`](skills/bollinger-reverter-4h/SKILL.md) |

Paired together as separate sub-accounts, the two strategies are **regime-complementary**: the Donchian gate fires zero trades during the chop windows where the Bollinger reverter thrives, and the Bollinger reverter mildly underperforms during the strong-trend windows the Donchian captures.

## Template strategies (starting points)

Reference templates for adapting to your own thesis. Backtest before deploying.

- [`dca-weekly`](skills/dca-weekly/SKILL.md) — dollar-cost averaging with scheduled buys
- [`grid-trading`](skills/grid-trading/SKILL.md) — profit-laddered position adjustment
- [`funding-rate-arbitrage`](skills/funding-rate-arbitrage/SKILL.md) — negative-funding capture (carry)
- [`funding-squeeze`](skills/funding-squeeze/SKILL.md) — funding-extreme squeeze ride
- [`basis-arb`](skills/basis-arb/SKILL.md) — spot-perp basis convergence
- [`breakout`](skills/breakout/SKILL.md) — Donchian breakout with trailing stop
- [`mean-reversion`](skills/mean-reversion/SKILL.md) — Bollinger band fade (4h validated; see file)
- [`scalping`](skills/scalping/SKILL.md) — RSI + volume-thrust template

Categories covered: **trend-following**, **mean-reversion**, **carry**, **arbitrage**, **scalping**.

## Reusable primitives

Building blocks that compose across strategies.

- [`trade-thesis`](skills/trade-thesis/SKILL.md) — Structured pre-trade thesis builder: bull/bear cases, invalidation criteria, and sizing rationale before any live deployment of a new strategy idea. Aliases: **pre-trade analysis**, **conviction check**, **trade plan**, **bull/bear case**.
- [`regime-overlay`](skills/regime-overlay/SKILL.md) — Triple-confirmation regime gate (EMA separation + ADX + N-bar return). Turns fragile directional strategies into regime-robust ones. Aliases: **regime filter**, **trend gate**, **directional confirmation**.
- [`dsl-exit-engine`](skills/dsl-exit-engine/SKILL.md) — Three-phase exit primitive: ROI ladder, hard stop, ratcheting trailing stop. Aliases: **ratcheting trailing stop**, **two-phase exit**, **take-profit ladder**.
- [`fees-optimizations`](skills/fees-optimizations/SKILL.md) — Maker (ALO) vs taker (MARKET) order-type decisioning, builder fees, parameter sweeps. Aliases: **fee optimizer**, **ALO vs MARKET**, **maker pricing**.
- [`backtesting`](skills/backtesting/SKILL.md) — Window selection, walk-forward, parameter sweeps.

## Intelligence (opportunity scanner)

The `skills/intelligence/` folder provides the platform's pair-ranking system. Aliases: **opportunity scanner**, **pair scanner**, **market screener**, **smart-money detector**, **four-stage funnel**.

- [`references/buckets.md`](skills/intelligence/references/buckets.md) — The four bucket framework: squeeze fuel, stealth accumulation, coiled spring, basis flipping.
- [`references/api.md`](skills/intelligence/references/api.md) — `/v2/intelligence/scan` endpoint and setup.
- [`references/workflow.md`](skills/intelligence/references/workflow.md) — Scan-to-deploy recipes.
- [`references/glossary.md`](skills/intelligence/references/glossary.md) — Terminology reference.

## Capability matrix

| Capability | Where it lives | Aliases |
|---|---|---|
| API key onboarding | `SKILL.md` → `/auth/sign-in/magic-link` | auth setup, email API key, x-api-key |
| Hyperliquid account bootstrap | `skills/superior-trade-hyperliquid/SKILL.md` → `/v3/account/{address}/hyperliquid` + `/v3/account/{address}/status/hyperliquid` | setup readiness, agent wallet, builder fee |
| Strategy backtesting | `skills/superior-trade-hyperliquid/SKILL.md` → `/v2/backtesting` | walk-forward, parameter sweep |
| Live deployment | `skills/superior-trade-hyperliquid/SKILL.md` → `/v2/deployment` | autonomous execution, agentic trading |
| Opportunity scanner | `skills/intelligence/` | pair scanner, market screener, ranking funnel |
| Fee optimizer | `skills/fees-optimizations/SKILL.md` | ALO vs MARKET, maker vs taker, fee budgeting |
| Two-phase trailing stop | `skills/dsl-exit-engine/SKILL.md` | ratcheting stop, DSL exit engine |
| Pre-trade thesis | `skills/trade-thesis/SKILL.md` | trade plan, conviction check, bull/bear case |
| Regime gate | `skills/regime-overlay/SKILL.md` | trend filter, directional confirmation |
| Sub-account orchestration | `skills/superior-trade-hyperliquid/SKILL.md` → `/v2/portfolio/...` | multi-strategy isolation |
| Hyperliquid deposit | `skills/superior-trade-hyperliquid/SKILL.md` → `/v2/portfolio/hyperliquid/deposit` | Arbitrum USDC deposit, fund trading |
| Atomic exit-all | `skills/superior-trade-hyperliquid/SKILL.md` → `/v2/portfolio/hyperliquid/exit` | kill-switch, emergency exit |
| HIP3 RWA support | `skills/superior-trade-hyperliquid/SKILL.md` → HIP3 section | tokenized stocks, commodities, indices |
| Polymarket strategy archetypes | one skill each: `skills/probability-momentum/`, `…/probability-mean-reversion/`, `…/deadline-drift/`, `…/related-market-spread/`, `…/large-fill-pressure/`, `…/catalyst-confirmation/` | prediction-market archetypes |
| Managed wallet | `skills/superior-trade-hyperliquid/SKILL.md` → Account Setup | no-key trading, custodial-style UX |

## Folder layout

Every skill is a self-contained directory under `skills/`, following the [Agent Skills](https://agentskills.io) standard (`skills/<name>/SKILL.md`).

- `SKILL.md` (repo root) — auth/onboarding: request an API key by email and use the `x-api-key` header. Also served at `superior.trade/SKILL.md`.
- `skills/superior-trade-hyperliquid/` — Hyperliquid/Freqtrade API reference and agent operating rules; the core trading skill the strategies build on.
- `skills/<strategy>/` — one skill per trading strategy (trend, mean-reversion, carry, arb, scalp) — validated drop-ins (`donchian-strong-regime`, `bollinger-reverter-4h`) and starting-point templates.
- `skills/<primitive>/` — reusable primitives: `regime-overlay`, `dsl-exit-engine`, `fees-optimizations`, `backtesting`, `trade-thesis`.
- `skills/aerodrome/` — Aerodrome spot-AMM execution.
- `skills/intelligence/` — opportunity scanner / pair-ranking; detail in `references/`.
- `skills/polymarket/` — prediction-market trading entry skill (`/v3` API); each archetype (`probability-momentum`, `probability-mean-reversion`, `deadline-drift`, `related-market-spread`, `large-fill-pressure`, `catalyst-confirmation`) is its own skill.

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

Installing the repo pulls the whole library; grab a single skill with `--skill <name>` (npx) or
by naming it (`gh skill install Superior-Trade/superior-skills <name>`).

Hyperliquid skills need a **`SUPERIOR_TRADE_API_KEY`**; Polymarket skills use
**`SUPERIOR_TRADE_PM_API_KEY`**. The root `SKILL.md` (`superior-trade-auth`) walks a new user
through getting a key by email.

## Workflow

The platform emphasizes a structured progression: **draft → backtest → review → deploy**. No live deployment without explicit user confirmation. Every parameter is logged; every trade is tracked.

## Risk disclosure

Trading involves risk. Backtests do not guarantee future performance. The validated strategies above showed positive returns on a single 162-day window (2025-11-20 → 2026-05-01); a strategy that worked then may not work in a different regime. Users remain responsible for strategy choice, deployment decisions, exchange connectivity, and capital risk. Pair every deployment with the [`regime-overlay`](skills/regime-overlay/SKILL.md) gate or equivalent — strategies without regime confirmation are demonstrably fragile.

## Topics

`hyperliquid`, `trading-bot`, `ai-agents`, `openclaw`, `backtesting`, `fee-optimization`, `opportunity-scanner`, `trailing-stop`, `algorithmic-trading`, `regime-filter`, `mean-reversion`, `trend-following`, `funding-arbitrage`, `mcp`
