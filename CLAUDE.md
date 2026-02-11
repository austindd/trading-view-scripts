# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains Pine Script (v5) indicators and strategies for TradingView, focused on **market structure analysis and price action trading**. All scripts are authored by "scientificbest" and licensed under Mozilla Public License 2.0.

## File Format

- **Extension:** `.ps` files contain Pine Script code (version 5)
- **Language:** Pine Script v5 (denoted by `//@version=5` directive)
- **License:** Mozilla Public License 2.0

## Development Workflow

### Testing

Pine Script cannot be tested locally. To test changes:

1. Open TradingView in a browser
2. Navigate to Pine Editor (bottom panel)
3. Copy the entire script content and paste into the editor
4. Click "Add to Chart" to visualize
5. Check for compilation errors in the console

### Deployment

Scripts are deployed by copying content to TradingView's Pine Editor and saving/publishing within TradingView. No build or compilation commands exist.

## Scripts

### Market Structure Mapper Series

The core of this repo. Each version builds on the previous:

| File | Lines | Description |
|------|-------|-------------|
| `market-structure-mapper-v2.ps` | 367 | Foundational version. Identifies valid highs/lows, BOS, and liquidity sweeps using a state machine pattern with parallel arrays. |
| `market-structure-mapper-v3.ps` | 1,115 | Adds higher timeframe (HTF) overlay with dual-timeframe tracking (CTF + HTF). |
| `market-structure-mapper-v4.ps` | 1,141 | Refactors with custom types (`HighLevel`, `LowLevel`) replacing parallel arrays. Level classifications: strong, weak, broken, swept. |
| `market-structure-mapper-v5.ps` | 1,838 | Adds supply/demand zones (`Zone` type) with box visualization, break conditions (wick vs close), and ATR-based min height filter. |
| `market-structure-mapper-v6.ps` | 2,512 | Three-timeframe analysis (CTF + HTF + HTF2/trading range). S&D zones per timeframe. |
| `market-structure-mapper-v7.ps` | 3,029 | **Latest version.** Full three-timeframe with comprehensive settings, independent toggles per TF, peaks/valleys visualization, broken zone extension. |
| `__working_version_of_v7.ps` | 2,819 | Experimental variant of V7 with tweaked defaults (lime/maroon colors). |
| `market-structure-mapper-strategy-v1.ps` | 2,836 | **Strategy** (not indicator). Backtestable zone trading with directional bias filter, R:R config (1:1–5:1), max active trades, zone intersection stats table. |

### Supply & Demand Zones Series

Standalone S&D zone detection using z-score analysis to identify significant price moves:

| File | Lines | Description |
|------|-------|-------------|
| `supply-demand-zones-v4.ps` | 471 | Z-score based zone detection. Identifies "explosive" candles via statistical outliers, creates zones at move origins. Tracks touched/crossed status. Configurable significance threshold and move-to-zone ratio filter. |
| `supply-demand-zones-v8.ps` | 756 | **Multi-timeframe version.** Adds HTF zone detection with independent enable/colors. Maintains HTF candle history arrays for proper lookback. Debug labels option for troubleshooting. |

### Specialized Indicators

| File | Lines | Description |
|------|-------|-------------|
| `candlestick-swing-points.ps` | 120 | Plots horizontal lines from 3-candle swing points (middle candle high/low exceeds neighbors). Lines are solid until wicked (dotted) or closed through (stops extending). |
| `candle-continuation-theory.ps` | 201 | HTF candle structure analysis for continuation signals. Visualizes HTF candle boxes (N, N-1, N-2) with BOS detection across HTF candles. |
| `virgin-wick-theory.ps` | 97 | Identifies virgin wicks (uncrossed wick extremes from HTF candles). Array-based tracking with 2-candle expiry. Alerts on close-through. |
| `htf-engulfing-sweep.ps` | 206 | Detects HTF engulfing patterns with sweep ray generation. Rays extend 2 HTF candles then stop. |
| `engulfing-bar-play.ps` | 275 | Engulfing bar detection with configurable consecutive-candle filter (default: 3 prior candles in same direction). Ray extensions from pattern extremes. |
| `wickless-candles.ps` | 29 | Simplest script. Draws horizontal lines at wickless candle levels. No state tracking. |
| `wickless-candles-v2.ps` | 90 | Enhanced wickless detection with array-based level tracking, alert generation on formation/first cross/retest, and configurable tracking window (default: 20 bars). |

## Architecture Patterns

### State Machine (All MSM versions)

1. **Initialization:** Wait for first BOS signal, scan back for initial structure
2. **Structure Detection:** Alternate between seeking highs and lows based on market flow
3. **Level Management:** Track levels in arrays, update broken/swept status
4. **Visual Updates:** Restyle lines each bar (solid → dotted on sweep, stop extending on break)

### Key Concepts

- **Break of Structure (BOS):** Price closes beyond a current valid level
- **Sweep/Wick:** Wick crosses a level but close doesn't (level becomes dotted)
- **Valid High/Low:** Structure points identified after breakdowns/breakouts
- **Supply/Demand Zones (V5+):** Box regions around structure points that may act as future support/resistance
- **Candlestick Swing Point:** 3-candle pattern where middle candle's high/low exceeds both neighbors
- **Z-Score Zone Detection:** Statistical outlier detection to identify "explosive" moves; zones form at move origins

### Multi-Timeframe Pattern

All HTF-aware scripts use:
```pine
[htfHigh, htfLow, htfOpen, htfClose, htfTime] = request.security(syminfo.tickerid, htfTimeframe, [...])
newHTF = ta.change(htfTime) != 0  // Detect new HTF bar
```

### HTF History Arrays Pattern (S&D Zones V8)

For HTF calculations requiring lookback across multiple HTF candles, maintain history arrays:
```pine
var float[] htfHighHist = array.new<float>(50, na)
// ... other OHLC history arrays

if htfNewCandle
    array.unshift(htfHighHist, htfHigh[1])  // Push completed candle
    if array.size(htfHighHist) > 50
        array.pop(htfHighHist)
```
This allows iterating over previous HTF candles in detection loops, since `htfHigh[i]` only gives CTF bar history of sampled HTF values.

### Data Structure Evolution

- **V2–V3:** Parallel arrays (`highLevels`, `highBars`, `highBroken`, `highLines`, etc.)
- **V4+:** Custom types encapsulating level data:
  ```pine
  type HighLevel
      float price
      int barIndex
      line levelLine
      string levelType  // "strong", "weak", "broken", "swept"
  ```
- **V5+:** Zone type for supply/demand:
  ```pine
  type Zone
      float top
      float bottom
      int leftTime
      int rightTime
      bool broken
      bool touched
      int creationBar
  ```

### Shared Helper Functions (MSM series)

- `isBearishBreakdown()` — bearish candle closing below previous low
- `isBullishBreakout()` — bullish candle closing above previous high
- `findLowestBetween(start, end)` — scan bar range for lowest low
- `findHighestBetween(start, end)` — scan bar range for highest high

## Common Modifications

- **Adding new levels:** Follow the typed array pattern from V4+ (`HighLevel`/`LowLevel`)
- **Changing detection logic:** Modify helper functions or main detection conditions
- **Adjusting visuals:** Update line/label/box creation functions or `checkBrokenLevels()`
- **Adding a new timeframe:** Follow the HTF pattern from V6+ (separate state variables, colors, and detection logic per TF)
- **Adding alerts:** Follow `wickless-candles-v2.ps` or `virgin-wick-theory.ps` patterns for array-based alert tracking

## Pine Script Language Notes

- **Bar execution model:** Script runs once per bar, left to right. No loops across future bars.
- **History operator:** `[1]` = previous bar, `[2]` = two bars ago. Array indices are 0-based.
- **State persistence:** `var` keyword maintains values across bars. Without `var`, variables reset each bar.
- **Type system:** Strictly typed (float, int, bool, string, color, line, label, box, etc.)
- **Scope:** Most variables must be declared at script scope, not inside conditions.
- **Drawing limits:** Indicators set `max_lines_count`, `max_labels_count`, `max_boxes_count` in the `indicator()` call. Exceeding these silently removes oldest drawings.
- **Lookback limit:** Find functions enforce a 5000-bar max lookback.
- **`request.security()`:** Used for HTF data. `lookahead` parameter controls whether current or confirmed bar data is used.
