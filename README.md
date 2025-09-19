# Break & Retest Swing EA

The Break & Retest Swing EA is a MetaTrader 5 expert advisor that automates a higher-timeframe breakout-and-retest methodology with prop-firm style risk controls. The strategy focuses on clean H4 breakouts of significant levels, waits for a retest with price action confirmation, and executes trades with tight risk management, robust logging, and optional higher timeframe filters.

## Key Features

- **Adaptive level engine** – builds support/resistance zones from configurable fractal swings and optional round numbers, filtered by touch count, spacing and ATR-sized zones.
- **Breakout & retest logic** – marks levels as "broken" only when closes breach the zone plus an ATR buffer and watches for retests within a configurable bar window.
- **Price-action confirmation** – optional pin bar and engulfing requirements on the retest candle, plus EMA/RSI filters, spread limits, session window, and optional economic calendar filter.
- **Trade execution & management** – position sizing by percent equity risk, configurable SL/TP (single or multi-target), breakeven shifts, ATR or swing trailing, limit re-entries at the zone mid, and per-symbol position caps.
- **Prop-firm risk guardrails** – daily loss and overall drawdown limits, daily risk budget, daily trade cap, and automatic trade disablement (with optional emergency flat) when breached.
- **Visual aids & logs** – optional zone drawing with labelled rectangles and detailed expert log messages for level creation, breakouts, filters, and risk decisions.

## File Layout

```
MQL5/
└── Experts/
    └── BreakRetestSwingEA/
        └── BreakRetestSwingEA.mq5
└── Presets/
    └── BreakRetestSwingEA_default.set
```

Copy the `BreakRetestSwingEA` folder into your terminal's `MQL5/Experts` directory (or sync the entire repository) and compile the EA inside MetaEditor. The default parameter set is provided under `MQL5/Presets` for quick loading inside the Strategy Tester or on live charts.

## Inputs Overview

| Group | Parameter | Description |
| --- | --- | --- |
| General | `Symbols`, `PrimaryTF`, `FilterTF`, `MagicNumber`, `CommentPrefix`, `SlippagePoints` | Multi-symbol scanning (comma separated) or leave blank for chart symbol only. Primary timeframe defaults to H4, with optional D1 filters. |
| Level Detection | `FractalDepth`, `MinTouches`, `MinBarsBetweenTouches`, `ATR_Period`, `LevelZoneATRmult`, `UseRoundNumbers`, `RoundIncrementPips`, `MinLevelSeparationPips` | Controls how zones are discovered and filtered. Zones are sized by ATR and deduplicated within the separation distance. |
| Break & Retest | `BreakBufferATRmult`, `MaxRetestBars`, `RetestToleranceATRmult`, `GapInvalidateATRmult` | Defines breakout confirmation buffer, retest window length, and how tolerant retests/gaps should be. |
| Confirmation | `RequirePinBar`, `PinWickFactor`, `ClosePercentile`, `RequireEngulfing`, `UseEMAFilter`, `EMA_Period`, `UseRSIFilter`, `RSI_Period`, `RSI_Upper`, `RSI_Lower` | Configure the confirmation candle characteristics and optional momentum filters. |
| Entry/SL/TP | `UseLimitAtMidZone`, `SL_BufferATRmult`, `SL_ATRmult`, `UseMaxOfZoneOrATR`, `TargetRR`, `UseMultiTP`, `TP1_RR`, `TP2_RR`, `TP3_Method`, `BE_TriggerRR`, `BE_LockPips`, `TrailMethod`, `ATR_TrailMult` | Define entry style, risk parameters, breakeven and trailing logic. Multi-TP mode automatically manages partial closes at R-multiples and optional structural targets. |
| Trade Management | `MaxPositionsPerSymbol`, `MaxTradesPerDay`, `AllowedHours`, `UseNewsFilter`, `NewsImpact`, `NewsBlockMinutesBefore/After` | Caps trade frequency, enforces session timing, and optionally blocks around high-impact economic events via the MT5 calendar feed. |
| Prop-Firm Controls | `RiskPercent`, `MaxDailyRiskPercent`, `DailyLossLimitPercent`, `MaxDrawdownPercent`, `CloseAllOnDD`, `UseCustomDayStart`, `CustomDayStartHour` | Implements risk sizing, cumulative risk exposure limits, daily equity loss ceilings, and overall drawdown locks. |
| Utilities | `BacktestMode`, `LogVerbosity`, `DrawObjects`, `MaxSpreadPoints` | Toggle verbose logging, chart annotations, and max allowable spread for entries. |

See the `.set` file for the default configuration aligned with the project brief (0.75% risk, 2R targets, pin-bar confirmation, EMA filter enabled, etc.).

## Operating Notes

1. **Initial setup**: Attach the EA on an H4 chart (or any chart if scanning multi-symbols). Load the preset or adjust inputs. Ensure the MT5 economic calendar is enabled if you plan to use the news filter.
2. **Daily reset**: The EA stores start-of-day equity (broker midnight or a custom hour) to enforce the daily loss cap. When the cap is hit, new entries are blocked for the remainder of the day.
3. **Drawdown protection**: The EA tracks peak equity for max drawdown protection. When `CloseAllOnDD` is true and the limit is hit, all EA-managed positions are flattened and trading pauses until equity recovers above the threshold.
4. **Retest lifecycle**: Levels meeting quality criteria are tracked as active zones. When a breakout candle closes beyond the ATR buffer, the EA starts a retest window. During that window, if price re-enters the zone and the confirmation candle passes all filters, orders are placed (market or limit).
5. **Trade management**: Risk per trade is auto-sized from equity and SL distance. Breakeven and trailing are monitored each tick. Multi-target mode closes half the remaining size at each configured R multiple and fully exits at the structural target if provided.
6. **Backtesting**: Enable `BacktestMode` to suppress non-essential logs if desired. Use MT5's Strategy Tester in "Every tick" or "Every tick based on real ticks" for best fidelity. Load the preset file for consistent parameters.

## Logging & Visuals

- Zones are drawn as dashed rectangles (red for resistance, blue for support) when `DrawObjects` is enabled. They are refreshed each new primary timeframe bar.
- Log verbosity 0/1/2 controls how much detail prints in the Experts tab (0 = critical only, 1 = standard workflow, 2 = verbose diagnostics).
- Risk checks report the reason whenever trading is blocked (spread, session window, risk caps, news proximity, etc.).

## News Filter Setup

When `UseNewsFilter` is true, the EA queries the built-in MT5 economic calendar and blocks entries inside the configurable minutes-before/after window for events at or above the chosen impact (`High` or `Medium`). Ensure `Tools → Options → Events` is enabled and the terminal is connected to receive calendar updates.

## Disclaimer

This project delivers the full MQL5 source code and default preset as requested. Always forward-test and validate the behaviour under your broker's conditions before trading real funds. Prop-firm rules vary; adjust the risk limits to match your plan.
