# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains Pine Script indicators for TradingView. Pine Script is TradingView's domain-specific language for creating custom technical indicators and strategies for financial markets.

**Current Script:** Market Structure Mapper V2 - An indicator that identifies and tracks valid highs/lows, break of structure (BOS), and liquidity sweeps in market price action.

## File Format

- **Extension:** `.ps` files contain Pine Script code (version 5)
- **Language:** Pine Script v5 (denoted by `//@version=5` directive)
- **License:** Mozilla Public License 2.0

## Development Workflow

### Testing Scripts

Pine Script cannot be tested locally. To test changes:

1. Open TradingView in a browser
2. Navigate to Pine Editor (bottom panel)
3. Copy the entire script content
4. Paste into the editor
5. Click "Add to Chart" to visualize
6. Check for compilation errors in the console

### Deployment

Pine Scripts are deployed by:
1. Copying script content to TradingView's Pine Editor
2. Saving/publishing directly within TradingView
3. No build or compilation commands exist for this codebase

## Code Architecture

### Market Structure Mapper V2 Logic Flow

The indicator follows a state machine pattern to track market structure:

1. **Initialization Phase** (lines 280-298)
   - Waits for first bearish breakdown signal
   - Scans back up to 200 bars to find initial valid high
   - Sets `initialized = true` and begins tracking

2. **State Tracking** (lines 28-35)
   - `lookingForLow` / `lookingForHigh`: Boolean flags controlling which structure element to seek next
   - `currentValidHigh` / `currentValidLow`: Active structure levels being tracked
   - Alternates between seeking highs and lows based on market flow

3. **Structure Detection** (lines 300-363)
   - **Break of Structure (BOS):** Occurs when price closes beyond the current valid level
   - **New Valid High:** Identified after bearish breakdown when `lookingForHigh = true`
   - **New Valid Low:** Identified after bullish breakout when `lookingForLow = true`
   - Uses `findHighestBetween()` and `findLowestBetween()` to locate structure points

4. **Level Management** (lines 107-169)
   - Arrays store all historical levels with metadata (broken/sweeped status)
   - `addValidHigh()` / `addValidLow()` functions manage level creation
   - Previous solid lines deleted when new levels form to prevent clutter
   - Most recent high/low indices tracked separately for BOS detection

5. **Visual Updates** (lines 171-278)
   - `checkBrokenLevels()` runs every bar to update line styles
   - **Sweep:** Wick crosses level but close doesn't → line becomes dotted
   - **Break:** Close crosses level → line treatment depends on recency
     - Recent level (potential BOS): Line stops extending, turns gray, gets BOS label
     - Old level: Line deleted entirely

### Key Helper Functions

- `isBearishBreakdown()` (line 42): Detects bearish candle closing below previous low
- `isBullishBreakout()` (line 46): Detects bullish candle closing above previous high
- `findLowestBetween()` (line 50): Scans bar range for lowest low price
- `findHighestBetween()` (line 79): Scans bar range for highest high price

### Pine Script Constraints

- **Max lines:** 500 (set in indicator declaration)
- **Max labels:** 500 (set in indicator declaration)
- **Lookback limit:** 5000 bars (enforced in find functions at lines 67, 96)
- **Persistence:** Uses `var` keyword for variables that maintain state across bars

## Configuration Options

The script exposes user inputs (lines 7-13):

- **Display toggles:** Show/hide high/low labels and BOS labels
- **Styling:** Line width (1-5)
- **Colors:** Valid high, valid low, and BOS colors (customizable)

## Common Modifications

When modifying this script:

- **Adding new levels:** Follow the pattern of `highLevels`/`lowLevels` arrays with parallel tracking arrays
- **Changing detection logic:** Modify the helper functions or conditions in lines 300-363
- **Adjusting visuals:** Update line/label creation in `addValidHigh()`/`addValidLow()` or `checkBrokenLevels()`
- **Performance tuning:** Adjust lookback limits (currently 200 for init, 5000 for searches)

## Pine Script Language Notes

- **Bar execution model:** Script runs once per bar, sequentially from left to right on chart
- **No loops across bars:** Cannot iterate through future bars (only historical via `[index]` notation)
- **Array indices:** 0-based, but bar history uses `[1]` for previous bar, `[2]` for two bars ago
- **Type system:** Strictly typed (float, int, bool, string, color, line, label, etc.)
- **Scope:** Most variables must be declared at script scope, not inside conditions
