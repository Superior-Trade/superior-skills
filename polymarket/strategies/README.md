# Polymarket Strategy Archetypes

These templates are starting points for writing and adapting Polymarket strategies on Superior Trade with NautilusTrader. They are not promises of profit. Use them to narrow the search space, generate strategy code, and test ideas before trading.

## How to use this folder

1. Search markets with `POST /v3/markets/search` and pick an exact market slug.
2. Pick the closest archetype based on market shape and your thesis.
3. Generate NautilusTrader strategy code with `on_trade_tick`.
4. Backtest with `POST /v3/backtest` using custom `strategySource` and `strategyConfig`.
5. Review filled-trade behavior, PnL, duration, and whether assumptions held.
6. Tune parameters or reject the setup.
7. Discuss deployment only after backtest completion and explicit user confirmation.

## Core assumptions to keep in view

- Backtests are built from filled `TradeTick` data, not full order-book simulation.
- Exact market slugs from search results are required before writing a strategy.
- Filled-trade backtests can reduce guesswork, but they do not replace liquidity or queue-aware live execution.

## Archetypes

| Archetype | Use when | Best fit | Weak fit |
|---|---|---|---|
| [Probability Momentum](probability-momentum.md) | Price and fills accelerate in one direction | Crypto milestones, fast news markets | Thin markets, single-print moves |
| [Probability Mean Reversion](probability-mean-reversion.md) | Probability overextends and then loses momentum | Noisy politics/culture/sports markets | New information shocks that permanently reprices the market |
| [Deadline Drift](deadline-drift.md) | Time-to-resolution changes the fair setup | Before-date, expiry-driven markets | Markets with unclear resolution mechanics |
| [Related-Market Spread](related-market-spread.md) | Related markets imply inconsistent probabilities | BTC levels, election derivative markets | Markets with mismatched resolution rules |
| [Large-Fill Pressure](large-fill-pressure.md) | Repeated large fills indicate directional pressure | High-volume active markets | Wash-like or low-volume microstructure |
| [Catalyst Confirmation](catalyst-confirmation.md) | User has a catalyst thesis and asks for confirmation | CPI/Fed/election/court/ETF events | Pure chart-only requests |
