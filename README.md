# Superior Skills

Trading strategies and intelligence tools for [Superior Trade](https://superior.trade) — natural-language strategy authoring, backtesting, and autonomous deployment on Hyperliquid.

Designed for OpenClaw users adding trading capabilities to their agent, and for traders who want validated templates rather than rolling their own.

[Website](https://superior.trade) · [Discord](https://discord.gg/aVZR8cCxcR) · [Twitter](https://x.com/SuperiorTrade_)

---

## Validated strategies (with backtest evidence)

These strategies have backtest evidence on real Hyperliquid data. Numbers are full-period (162 days, 2025-11-20 → 2026-05-01) unless noted.

| Strategy | Regime | Pairs tested | Trades | Win | Profit | Max DD | File |
|---|---|---|---|---|---|---|---|
| **Donchian Strong-Regime** | Strong directional trend | BTC | 6 | 100% | **+6.69%** | **0%** | [strategies/donchian-strong-regime.md](strategies/donchian-strong-regime.md) |
| **Bollinger Reverter 4h** | Range / chop (ADX<25) | BTC/ETH/SOL/DOGE | 84 | 65.5% | **+8.77%** | 18.5% | [strategies/bollinger-reverter-4h.md](strategies/bollinger-reverter-4h.md) |

Paired together as separate sub-accounts, the two strategies are **regime-complementary**: the Donchian gate fires zero trades during the chop windows where the Bollinger reverter thrives, and the Bollinger reverter mildly underperforms during the strong-trend windows the Donchian captures.

## Template strategies (starting points)

Reference templates for adapting to your own thesis. Backtest before deploying.

- [`strategies/dca-weekly.md`](strategies/dca-weekly.md) — dollar-cost averaging with scheduled buys
- [`strategies/grid-trading.md`](strategies/grid-trading.md) — profit-laddered position adjustment
- [`strategies/funding-rate-arbitrage.md`](strategies/funding-rate-arbitrage.md) — negative-funding capture (carry)
- [`strategies/funding-squeeze.md`](strategies/funding-squeeze.md) — funding-extreme squeeze ride
- [`strategies/basis-arb.md`](strategies/basis-arb.md) — spot-perp basis convergence
- [`strategies/breakout.md`](strategies/breakout.md) — Donchian breakout with trailing stop
- [`strategies/mean-reversion.md`](strategies/mean-reversion.md) — Bollinger band fade (4h validated; see file)
- [`strategies/scalping.md`](strategies/scalping.md) — RSI + volume-thrust template

Categories covered: **trend-following**, **mean-reversion**, **carry**, **arbitrage**, **scalping**.

## Reusable primitives

Building blocks that compose across strategies.

- [`optimizations/regime-overlay.md`](optimizations/regime-overlay.md) — Triple-confirmation regime gate (EMA separation + ADX + N-bar return). Turns fragile directional strategies into regime-robust ones. Aliases: **regime filter**, **trend gate**, **directional confirmation**.
- [`optimizations/dsl-exit-engine.md`](optimizations/dsl-exit-engine.md) — Three-phase exit primitive: ROI ladder, hard stop, ratcheting trailing stop. Aliases: **ratcheting trailing stop**, **two-phase exit**, **take-profit ladder**.
- [`optimizations/fees-optimizations.md`](optimizations/fees-optimizations.md) — Maker (ALO) vs taker (MARKET) order-type decisioning, builder fees, parameter sweeps. Aliases: **fee optimizer**, **ALO vs MARKET**, **maker pricing**.
- [`optimizations/backtesting.md`](optimizations/backtesting.md) — Window selection, walk-forward, parameter sweeps.

## Intelligence (opportunity scanner)

The `intelligence/` folder provides the platform's pair-ranking system. Aliases: **opportunity scanner**, **pair scanner**, **market screener**, **smart-money detector**, **four-stage funnel**.

- [`intelligence/buckets.md`](intelligence/buckets.md) — The four bucket framework: squeeze fuel, stealth accumulation, coiled spring, basis flipping.
- [`intelligence/api.md`](intelligence/api.md) — `/v2/intelligence/scan` endpoint and setup.
- [`intelligence/workflow.md`](intelligence/workflow.md) — Scan-to-deploy recipes.
- [`intelligence/glossary.md`](intelligence/glossary.md) — Terminology reference.

## Capability matrix

| Capability | Where it lives | Aliases |
|---|---|---|
| Strategy backtesting | `SKILL.md` → `/v2/backtesting` | walk-forward, parameter sweep |
| Live deployment | `SKILL.md` → `/v2/deployment` | autonomous execution, agentic trading |
| Opportunity scanner | `intelligence/` | pair scanner, market screener, ranking funnel |
| Fee optimizer | `optimizations/fees-optimizations.md` | ALO vs MARKET, maker vs taker, fee budgeting |
| Two-phase trailing stop | `optimizations/dsl-exit-engine.md` | ratcheting stop, DSL exit engine |
| Regime gate | `optimizations/regime-overlay.md` | trend filter, directional confirmation |
| Sub-account orchestration | `SKILL.md` → `/v2/portfolio/...` | multi-strategy isolation |
| Atomic exit-all | `SKILL.md` → `/v2/portfolio/hyperliquid/exit` | kill-switch, emergency exit |
| HIP3 RWA support | `SKILL.md` → HIP3 section | tokenized stocks, commodities, indices |
| Managed wallet | `SKILL.md` → Account Setup | no-key trading, custodial-style UX |

## Folder layout

- `strategies/` — Template trading strategies (trend, mean-reversion, carry, arb, scalp). Includes validated drop-in strategies and starting-point templates.
- `intelligence/` — Market scanning and pair-ranking tools across Hyperliquid altcoins. The opportunity scanner / market screener lives here.
- `optimizations/` — Reusable primitives: regime gate, DSL exit engine, fee optimizer, backtesting harness.
- `exchanges/` — Exchange-specific implementations (e.g., Aerodrome for spot AMM execution).
- `SKILL.md` — Full API reference and agent operating rules.

## Getting started

```
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

Install paths:
- **OpenClaw**: agent prompt → "Install superior-skills"
- **ClawHub**: see `.clawhubignore` for skill manifest
- **Manual**: clone this repo into your agent's skills folder

## Workflow

The platform emphasizes a structured progression: **draft → backtest → review → deploy**. No live deployment without explicit user confirmation. Every parameter is logged; every trade is tracked.

## Risk disclosure

Trading involves risk. Backtests do not guarantee future performance. The validated strategies above showed positive returns on a single 162-day window (2025-11-20 → 2026-05-01); a strategy that worked then may not work in a different regime. Users remain responsible for strategy choice, deployment decisions, exchange connectivity, and capital risk. Pair every deployment with the [`optimizations/regime-overlay.md`](optimizations/regime-overlay.md) gate or equivalent — strategies without regime confirmation are demonstrably fragile.

## Topics

`hyperliquid`, `trading-bot`, `ai-agents`, `openclaw`, `backtesting`, `fee-optimization`, `opportunity-scanner`, `trailing-stop`, `algorithmic-trading`, `regime-filter`, `mean-reversion`, `trend-following`, `funding-arbitrage`, `mcp`
