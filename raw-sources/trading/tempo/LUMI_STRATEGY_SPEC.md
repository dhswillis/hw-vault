# Lumi Traders Strategy Specification

Source: @LumiTraders (X/Twitter), March 2026
Claimed performance: 200 backtested + 40 forward-tested trades, ~65% win rate, 4 months data

## Setup Flow (in order)

### 1. HTF LEVEL
- Timeframes: M15 / M30 / H1 / H4
- Level types: FVG / OB / Liquidity (swing highs/lows)
- These are the structural levels that price must react from

### 2. M30/H1 Potential Sweep
- Reaction from HTF level must be confirmed by a sweep on M30 or H1
- Sweep = wick past the level + close back inside
- The sweep timeframe determines the LTF pairing (see step 3/4)

### 3. M3/M5 Order Block (LTF OB)
- If M30 sweep → look for M3 OB
- If H1 sweep → look for M5 OB
- OB = displacement candle in the reversal direction

### 4. M3 FVG / M5 FVG (LTF FVG)
- If M30 sweep → M3 OB + M3 FVG
- If H1 sweep → M5 OB + M5 FVG
- FVG can form BEFORE or AFTER the OB:
  - **FVG before OB**: Enter after OB is confirmed. Target still measured from FVG entry.
  - **FVG after OB**: Enter as soon as FVG forms. Can add to position if price retraces into FVG.

### 5. Target
- **2R from FVG** (NOT from personal entry, NOT from swing stop)
- The 2R is always measured from the FVG entry price
- If FVG formed before OB, actual trade may offer less than full 2R from your entry

### 6. Stop Loss
- Swing High / Swing Low
- The recent swing extreme before the setup

## Trading Hours
- 8:00 AM - 4:00 PM EST
- Strong time zones:
  - 8:00 - 9:00 AM EST
  - 10:00 - 11:00 AM EST
  - 1:30 - 2:30 PM EST

## Visual Example (from diagram)
1. H1 OB forms (HTF level established)
2. M30 SWEEP of the H1 OB level (wick past, close back)
3. M3 OB forms after the sweep (displacement candle)
4. M3 FVG forms (gap between bars confirming direction)
5. Entry at FVG edge, target = 2R measured from FVG, stop = swing extreme

## Key Differences from C# v14 Implementation
- Sweep on M30 AND H1 (not just 60-min)
- LTF OB/FVG timeframe depends on sweep TF (M3 for M30 sweep, M5 for H1 sweep)
- Target is 2R (not 3R)
- Target measured from FVG gap size (not swing stop distance)
- HTF levels include M15, M30, H4 (not just H1)
- FVG can form before or after OB (two entry modes)
- Hours: 8:00 AM - 4:00 PM EST
