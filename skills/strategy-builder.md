# Strategy Builder — Superior Trade

Sub-skill for writing trading strategy code and configuration.

**Load when:** User describes a trading idea and the agent needs to generate strategy code + config.

## Strategy Code Template

The `code` field must be valid Python with a strategy class. Class name must end with `Strategy` in PascalCase. Use `import talib.abstract as ta` for indicators.

```python
from freqtrade.strategy import IStrategy
import pandas as pd
import talib.abstract as ta


class MyCustomStrategy(IStrategy):
    minimal_roi = {"0": 0.10, "30": 0.05, "120": 0.02}
    stoploss = -0.10
    trailing_stop = False
    timeframe = '5m'
    process_only_new_candles = True
    startup_candle_count = 20

    def populate_indicators(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
        dataframe['sma_20'] = ta.SMA(dataframe, timeperiod=20)
        return dataframe

    def populate_entry_trend(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        dataframe.loc[
            (dataframe['rsi'] < 30) & (dataframe['close'] > dataframe['sma_20']),
            'enter_long'
        ] = 1
        return dataframe

    def populate_exit_trend(self, dataframe: pd.DataFrame, metadata: dict) -> pd.DataFrame:
        dataframe.loc[(dataframe['rsi'] > 70), 'exit_long'] = 1
        return dataframe
```

**Requirements:**
- Standard imports/inheritance (see template)
- `import talib.abstract as ta` for indicators
- Define `populate_indicators`, `populate_entry_trend`, `populate_exit_trend`
- Class name must end in `Strategy` (PascalCase)

## Multi-Output TA-Lib Functions (CRITICAL)

Some TA-Lib functions return **multiple columns**. Assigning directly to one column causes a runtime crash.

| Function | Returns |
|----------|---------|
| `ta.BBANDS` | `upperband`, `middleband`, `lowerband` |
| `ta.MACD` | `macd`, `macdsignal`, `macdhist` |
| `ta.STOCH` | `slowk`, `slowd` |
| `ta.STOCHF` / `ta.STOCHRSI` | `fastk`, `fastd` |
| `ta.AROON` | `aroondown`, `aroonup` |
| `ta.HT_PHASOR` | `inphase`, `quadrature` |
| `ta.MAMA` | `mama`, `fama` |
| `ta.MINMAXINDEX` | `minidx`, `maxidx` |

```python
# WRONG — runtime crash
dataframe["bb_upper"] = ta.BBANDS(dataframe, timeperiod=20)

# CORRECT
bb = ta.BBANDS(dataframe, timeperiod=20)
dataframe["bb_upper"] = bb["upperband"]
dataframe["bb_middle"] = bb["middleband"]
dataframe["bb_lower"] = bb["lowerband"]

macd = ta.MACD(dataframe)
dataframe["macd"] = macd["macd"]
dataframe["macd_signal"] = macd["macdsignal"]
dataframe["macd_hist"] = macd["macdhist"]

stoch = ta.STOCH(dataframe)
dataframe["slowk"] = stoch["slowk"]
dataframe["slowd"] = stoch["slowd"]
```

Single-output functions (RSI, SMA, EMA, ATR, ADX) return a Series and can be assigned directly.

## Pair Format Reference

**CCXT convention — format depends on trading mode:**

| Mode | Format | Example |
|------|--------|---------|
| Spot | `BASE/QUOTE` | `BTC/USDC` |
| Futures/Perp | `BASE/QUOTE:SETTLE` | `BTC/USDC:USDC` |
| HIP3 | `PROTOCOL-TICKER/QUOTE:SETTLE` | `XYZ-AAPL/USDC:USDC` (hyphen, NOT colon) |

> Wrong pair format is the #1 cause of "pair not tradable" errors.

### HIP3 — Tokenized Real-World Assets

HIP3 assets (stocks, commodities, indices) are perpetual futures.

> **CRITICAL: HIP3 uses a HYPHEN, not a colon. This is the #1 format mistake.**

```python
# WRONG — colon after protocol (will fail with "pair not tradable")
"XYZ:AAPL/USDC:USDC"   # ← WRONG
"XYZ:GOLD/USDC:USDC"   # ← WRONG

# CORRECT — hyphen after protocol
"XYZ-AAPL/USDC:USDC"   # ← CORRECT
"XYZ-GOLD/USDC:USDC"   # ← CORRECT
```

The pattern is `PROTOCOL-TICKER/QUOTE:SETTLE` — the separator between protocol and ticker is always **`-`** (hyphen).

| Protocol | Asset Types | Stake Currency | Examples |
|----------|-------------|----------------|----------|
| `XYZ-` | US/KR stocks, metals, currencies, indices | USDC | `XYZ-AAPL/USDC:USDC`, `XYZ-GOLD/USDC:USDC` |
| `CASH-` | Commodities, stocks | USDT0 | `CASH-GOLD/USDT0:USDT0` |
| `FLX-` | Commodities, stocks, crypto | USDC or USDH | `FLX-GOLD/USDH:USDH` |
| `KM-` | Stocks, indices, bonds | USDH | `KM-GOOGL/USDH:USDH` |
| `HYNA-` | Leveraged crypto | USDC or USDE | `HYNA-SOL/USDE:USDE` |

**XYZ tickers (USDC):** AAPL, AMZN, GOOGL, META, TSLA, NFLX, HOOD, PLTR, INTC, RIVN, COIN, SNDK, BABA, GOLD, SILVER, COPPER, PLATINUM, PALLADIUM, JPY, XYZ100, HYUNDAI, SKHX, SMSN

**Trading rules:** HIP3 are futures-only — always use `trading_mode: "futures"` and `margin_mode: "isolated"`. XYZ pairs use `stake_currency: "USDC"`. Stock-based assets may have reduced liquidity outside US market hours.

### Pair Discovery

Verify pairs exist before using them:
```
POST https://api.hyperliquid.xyz/info
{"type":"meta"}
```
Check the response `universe` array for the coin name.

**When presenting HIP3 pairs to users**, always use the CCXT format with a **hyphen**: `XYZ-GOLD/USDC:USDC`, `CASH-GOLD/USDT0:USDT0`, etc. The Hyperliquid API returns raw coin names (e.g. `XYZ/GOLD`) — you must convert to CCXT format with hyphen separator before showing to users or using in config.

## Config Reference

The config object is the trading bot configuration. Fields `api_server` and `dry_run` are controlled by the platform and must NOT be included.

### Common Config Fields

| Field | Type | Description |
|-------|------|-------------|
| `exchange.name` | string | Must be `"hyperliquid"` |
| `exchange.pair_whitelist` | string[] | Spot: `["BTC/USDC"]`, Futures: `["BTC/USDC:USDC"]` |
| `stake_currency` | string | `"USDC"` (or `"USDT0"`, `"USDH"`, `"USDE"` for HIP3) |
| `stake_amount` | number or `"unlimited"` | Amount per trade (see warnings below) |
| `timeframe` | string | `"1m"`, `"5m"`, `"15m"`, `"1h"`, `"4h"`, `"1d"` |
| `max_open_trades` | integer | Max concurrent trades (-1 for unlimited) |
| `stoploss` | number | Must be negative, e.g. `-0.10` for 10% |
| `minimal_roi` | object | Minutes-to-ROI map: `{ "0": 0.10, "30": 0.05 }` |
| `trading_mode` | string | `"spot"` or `"futures"` (omit for spot default) |
| `margin_mode` | string | `"cross"` or `"isolated"` (required for futures) |
| `trailing_stop` | boolean | Enable trailing stop-loss |
| `trailing_stop_positive` | number | Trailing activation profit (requires `trailing_stop: true`) |
| `pairlists` | array | `StaticPairList`, `VolumePairList`, etc. |
| `entry_pricing.price_side` | string | `"ask"`, `"bid"`, `"same"`, `"other"` |
| `exit_pricing.price_side` | string | `"ask"`, `"bid"`, `"same"`, `"other"` |

### Futures Config (recommended)

```json
{
  "exchange": { "name": "hyperliquid", "pair_whitelist": ["BTC/USDC:USDC"] },
  "stake_currency": "USDC", "stake_amount": 100, "timeframe": "5m",
  "max_open_trades": 3, "stoploss": -0.1,
  "trading_mode": "futures", "margin_mode": "cross",
  "pairlists": [{ "method": "StaticPairList" }]
}
```

### Spot Config

```json
{
  "exchange": { "name": "hyperliquid", "pair_whitelist": ["BTC/USDC"] },
  "stake_currency": "USDC", "stake_amount": 100, "timeframe": "5m",
  "max_open_trades": 3, "stoploss": -0.1,
  "pairlists": [{ "method": "StaticPairList" }]
}
```

**Spot limitations:** No stoploss on exchange (bot handles internally), no market orders (simulated via limit with up to 5% slippage).

### HIP3 Config Example

```json
{
  "exchange": { "name": "hyperliquid", "pair_whitelist": ["XYZ-AAPL/USDC:USDC"] },
  "stake_currency": "USDC", "stake_amount": 100, "timeframe": "15m",
  "max_open_trades": 3, "stoploss": -0.05,
  "trading_mode": "futures", "margin_mode": "isolated",
  "entry_pricing": { "price_side": "other" },
  "exit_pricing": { "price_side": "other" },
  "pairlists": [{ "method": "StaticPairList" }]
}
```

## DCA / Position Scaling

The engine enforces one open trade per pair. Use `adjust_trade_position()` for DCA:

```python
def adjust_trade_position(self, trade, current_time, current_rate,
                          current_profit, min_stake, max_stake,
                          current_entry_rate, current_exit_rate,
                          current_entry_profit, current_exit_profit, **kwargs):
    if should_dca(trade, current_time):
        return max(500, min_stake)  # add $500, respect exchange minimum
    return None
```

- Called every candle while a trade is open. Positive = DCA buy, negative = partial close, None = no action.
- **Hyperliquid minimum: $10 per order.** Engine inflates by stoploss reserve (up to 1.5x) — always use `min_stake`.
- `max_open_trades` limits total concurrent trades across all pairs, not entries per pair.

### `stake_amount: "unlimited"` Warning

`"unlimited"` bypasses minimum-order validation. The bot starts but **silently executes zero trades** if balance is insufficient — no error, just heartbeats. Always use explicit numeric `stake_amount` with small balances (<$50).

| Stoploss | Effective minimum |
|----------|-------------------|
| -0.5% | ~$10.55 |
| -5% | ~$11.05 |
| -10% | ~$11.67 |
| -30% | ~$15.00 |

## Exchange Notes

**Supported exchange:** Hyperliquid only.

| Exchange | Stake Currencies | Trading Modes |
|----------|-----------------|---------------|
| Hyperliquid | USDC (also USDT0, USDH, USDE via HIP3) | spot, futures |

**Futures:** Margin modes `"cross"` and `"isolated"`. Stoploss on exchange via `stop-loss-limit` orders.

**Data availability:** Hyperliquid API provides ~5000 historic candles per pair. Pre-downloaded data availability starts from ~November 2025. XYZ from ~November 2025, KM/CASH/FLX from ~February 2026. Timeframes: 1m, 3m, 5m, 15m, 30m, 1h (also 2h, 4h, 8h, 12h, 1d, 3d, 1w for some). Funding rate data at 1h.

### Unified vs Legacy Account Mode

Hyperliquid accounts may run in **unified mode** (single balance) or **legacy mode** (separate spot/perps balances). Do NOT assume which mode the user has.

- If perps shows $0 but spot shows funds, ask about unified mode before suggesting the user move funds themselves.
- In unified mode, spot USDC is automatically available as perps collateral.

## Orphan Position Handling

When a bot crashes, it may leave open positions that lock up margin. Strategy code pattern:
- In `bot_loop_start()`, check for positions not in the bot's trade database
- Close orphans with a limit order before entering fresh
- Use a flag (`_orphan_closed`) to run cleanup exactly once per lifecycle
