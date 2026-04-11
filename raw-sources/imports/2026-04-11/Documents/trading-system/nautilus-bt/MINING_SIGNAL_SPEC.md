# BOS_FVG Signal Spec — Exact Match to Mining Framework

This is the EXACT signal logic that produced 11,348 trades, 63.5% WR, +15,368R across 258 days.
NautilusTrader must match this 1:1 to validate.

## Signal Detection

### 1. Break of Structure (BOS)
- Timeframe: 1-minute bars
- Swing detection: 3-bar HH/HL pattern on COMPLETED bars only
  - Swing High: bar[1].high > bar[0].high AND bar[1].high > bar[2].high
  - Swing Low: bar[1].low < bar[0].low AND bar[1].low < bar[2].low
- BOS Long: price closes above the most recent swing high
- BOS Short: price closes below the most recent swing low
- Track the BOS bar index for timing

### 2. Fair Value Gap (FVG)
- 3-candle imbalance pattern on 1-minute bars
- Bullish FVG: bar[0].low > bar[2].high (gap between bar 0 low and bar 2 high)
- Bearish FVG: bar[0].high < bar[2].low (gap between bar 0 high and bar 2 low)
- FVG must form DURING or AFTER the BOS displacement (within 15 bars of BOS)
- FVG direction must match BOS direction
- NO minimum FVG size requirement in mining
- NO stale FVG pruning in mining (FVG stays valid until filled)

### 3. FVG Fill (Entry Trigger)
- Price must retrace INTO the FVG zone
- Entry at the FVG fill price (when price touches the FVG zone)
- NOT at midpoint — at the edge of the FVG where price first enters
- This is a theoretical fill (mining assumes you get filled at touch)

### 4. Stop Loss
- Stop at the opposite edge of the FVG
- NO buffer (no extra points added)
- Risk = distance from entry to stop

### 5. Risk Filter
- Risk must be between 3 and 30 points
- If risk < 3pt or > 30pt, skip the trade

## Filters

### 6. Session Filter
- Trade must occur during valid session windows
- Valid sessions: RTH (Regular Trading Hours) primarily
- Mining did NOT apply strict session sub-windows (no news_window, ib_first, etc.)

### 7. Alignment Score
- Mining used an "alignment score" (al) based on multi-timeframe confluence
- NOT a confluence gate with volume/engulfing/etc.
- The alignment score counts how many higher-timeframe factors agree
- For the base BOS_FVG signal, NO minimum alignment score was required
- Sniper tiers filter on alignment AFTER the fact (al>=3 for Gunner, al>=5 for Sharpshooter)

### 8. Confirmation Override (co)
- co = 0 means standard confirmation
- co <= 0 for Sharpshooter tier
- This was a FILTER applied to results, not part of signal detection

## Exit: BE+Trail

### 9. Breakeven + Trail (THIS IS CRITICAL)
This is where the entire edge lives. Mining used BE+Trail, NOT 15S trailing.

- **Breakeven trigger**: When unrealized profit reaches 1R, move stop to entry + 0.5pt
- **Trail activation**: When unrealized profit reaches 2R, begin trailing
- **Trail formula**: stop = entry + (max_r - 1.0) × risk_pts
  - Example: entry=100, risk=5pt, max excursion=3R (115)
  - Trail stop = 100 + (3.0 - 1.0) × 5 = 110 (locked in 2R)
- **No max hold time** — trade runs until stopped out by trail
- **No time-based exit** — no 120 minute limit

### How BE+Trail creates the edge:
- 29.5% of trades are 2R+ runners → these produce +17,461R total
- 70.5% are losers + breakevens → these produce -2,094R total
- Net: +15,368R
- The runners MORE than pay for everything else
- Average near-zero breakeven rate: ~19.4% of trades

## What Mining Did NOT Have

These things are in the NautilusTrader version but were NOT in mining:

1. ❌ Confluence gate (volume_confirmed, volume_spike, engulfing, strong_bias)
2. ❌ FVG midpoint entry (mining entered at FVG edge/fill price)
3. ❌ Stop buffer (±2pt) — mining used raw FVG edge
4. ❌ 15S trailing stop (0.5R trigger, 0.1R trail) — mining used BE+Trail
5. ❌ Max hold 120 minutes — mining had no time limit
6. ❌ Min FVG size requirement
7. ❌ Stale FVG pruning (50 bars)
8. ❌ Entry/exit slippage of 0.25pt/0.5pt (mining applied different slippage model)

## Validation Targets

If NautilusTrader matches this spec exactly, it should produce approximately:
- ~44 trades per day (11,348 / 258 days)
- ~63.5% win rate
- ~+1.35R per trade
- ~+59.6R per day
- Total: ~+15,368R across 258 days

Differences from realistic execution (bar-close fills vs theoretical fills)
will create some gap, but the gap should be <20%, not >50%.
