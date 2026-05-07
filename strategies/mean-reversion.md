---
name: mean-reversion
description: Use when writing a Bollinger-band mean-reversion strategy on Superior Trade — anything described as mean reversion, BB bands, oversold bounce, fade, range trade, ADX low, sigma extension. Note signals are very rare on 2.5σ over 100-bar windows; widen if the user wants more trades.
version: 0.1.0
updated: 2026-05-07
---

# Strategy: Mean Reversion · BTC Bands

## When to use

A user asks for "mean reversion", "fade extremes", "buy oversold", "BB bands", "Bollinger bounce", "low-volatility range trade", "ADX filter". Single-pair, hour-scale, hard timeout.

## Honest framing

The reference backtest fired only **5 trades over 4 months** (40% WR, −0.20%). Signals on 2.5σ Bollinger bands with a 100-bar window are very rare on 1h BTC. The strategy is structurally fine but the parameters are too strict to generate statistical confidence. **Widen the bands** (2.0σ) or **shorten the window** (50 bars) for more activity, OR accept it as a "wait for the once-a-month tail event" tool.

## Backtest reference

| Window | `BTC/USDC:USDC` 1h, 2026-01-01 → 2026-05-01 |
|---|---|
| Trades | 5 |
| Win rate | 40% |
| Wallet PnL | **−0.20%** |
| Backtest ID | `01kqypwcw105ebmgw04gejk998` |

## Reference implementation

```python
from freqtrade.strategy import IStrategy
from datetime import datetime
import pandas as pd
import talib.abstract as ta


class BtcMeanReversionStrategy(IStrategy):
    minimal_roi = {"0": 100.0}    # exit driven by reaching the band middle
    stoploss = -0.04
    trailing_stop = False
    timeframe = "1h"
    process_only_new_candles = True
    startup_candle_count = 200
    can_short = False

    def populate_indicators(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        bb = ta.BBANDS(dataframe, timeperiod=100, nbdevup=2.5, nbdevdn=2.5)
        dataframe["bb_upper"] = bb["upperband"]
        dataframe["bb_mid"] = bb["middleband"]
        dataframe["bb_lower"] = bb["lowerband"]
        dataframe["adx"] = ta.ADX(dataframe, timeperiod=14)
        return dataframe

    def populate_entry_trend(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        # Long when price closes below lower band AND ADX confirms ranging regime.
        dataframe.loc[
            (dataframe["close"] < dataframe["bb_lower"]) & (dataframe["adx"] < 30),
            "enter_long",
        ] = 1
        return dataframe

    def populate_exit_trend(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        # Exit when price reaches the band middle (mean).
        dataframe.loc[(dataframe["close"] >= dataframe["bb_mid"]), "exit_long"] = 1
        return dataframe

    def custom_exit(self, pair: str, trade, current_time: datetime,
                    current_rate: float, current_profit: float, **kwargs):
        # Trade thesis is mean reversion; if it hasn't reverted in 24h
        # the regime probably changed. Don't bag-hold.
        elapsed_h = (current_time - trade.open_date_utc).total_seconds() / 3600.0
        if elapsed_h >= 24:
            return "timeout_24h"
        return None
```

## Config requirements

```json
{
  "exchange": { "name": "hyperliquid", "pair_whitelist": ["BTC/USDC:USDC"] },
  "stake_currency": "USDC",
  "stake_amount": 100,
  "timeframe": "1h",
  "max_open_trades": 1,
  "stoploss": -0.04,
  "minimal_roi": { "0": 100.0 },
  "trading_mode": "futures",
  "margin_mode": "cross",
  "entry_pricing": { "price_side": "same" },
  "exit_pricing": { "price_side": "same" },
  "pairlists": [{ "method": "StaticPairList" }]
}
```

`startup_candle_count: 200` is correct — the 100-bar BB needs 100 bars of warmup, plus headroom.

## Tunable parameters

| Knob | Effect |
|---|---|
| `nbdevup/nbdevdn = 2.5` | Wider (`3.0`) → fewer, deeper signals. Tighter (`2.0`) → more activity, lower per-trade edge. |
| `timeperiod = 100` | Shorter (`50`) → more reactive bands, more trades. Longer (`200`) → smoother bands, even fewer signals. |
| `adx < 30` | Higher cap (`< 40`) → loosens regime filter, allows entries during weak trends. Lower (`< 20`) → only deep ranging. |
| `timeout_24h` | The mean-reversion thesis breaks if reversion takes longer than typical regime length. 24h is sane for 1h timeframe. |
| `bb_mid` exit | Some traders prefer `bb_upper` (full reversion) for higher per-trade gain at the cost of more giveback. |

## Variants worth testing

- **Volatility-filtered entries**: add `atr_pct = atr_14 / close` and gate entries on `atr_pct > 0.005` to avoid entries in dead-quiet markets where the bands are too tight.
- **2.0σ bands**: ~10× more signals; reduces win rate but materially improves Sharpe statistics on most pairs.
- **Pair this with `populate_exit_trend` partial closes** at `bb_mid` and full close at `bb_upper` (requires `adjust_trade_position`, see `strategies/grid-trading.md`).
- **Multi-pair scan**: ranging happens at different times on different pairs. Top 20 perp scan increases trade count without lowering signal quality.

## Common pitfalls

1. **Forgetting the regime filter (ADX).** Without `adx < 30`, the strategy enters during strong downtrends — which look like "below lower band" but are actually trending breakdowns, not reversion setups. The ADX filter is what makes mean reversion a regime-aware strategy.
2. **`startup_candle_count` too low for the BB period.** Default 30 with `timeperiod = 100` gives 70 bars of NaN bands at the start of every backtest — looks like the strategy did nothing for the first 3 days.
3. **Treating low trade count as "broken".** 5 trades / 4 months at 2.5σ is **expected**. Either widen bands or accept it.
4. **Combining with mean-reversion exits in a downtrend.** When the regime changes mid-trade, you can sit on a losing position for hours waiting for "reversion" that never comes. The 24h timeout is non-negotiable.

## Sources

- Internal audit — `docs/standard-strategies-audit.md`, backtest `01kqypwcw105ebmgw04gejk998`
- TA-Lib BBANDS reference — https://ta-lib.org/
