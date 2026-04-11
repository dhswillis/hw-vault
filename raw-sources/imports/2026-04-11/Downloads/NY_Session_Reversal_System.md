# NY Session Reversal System — Engulfing & Pin Bar, 1R Target

## Session Window
9:30 AM – 12:00 PM ET (highest displacement, cleanest setups)

## Instruments
ES / NQ futures

## Entry Timeframe
Dynamic per Tempo framework (15s–5m based on vol/tick regime)

---

## Pre-Conditions (Filters Before Any Entry)

1. **Identify Draw on Liquidity (DOL)** — Before NY open, mark session high/low from Asia+London, 15m fractals, and unmitigated 5m/15m FVGs. These are where price is heading.

2. **Wait for a liquidity sweep** — Price must take out one of those DOL levels (session H/L, 15m fractal high/low, or trade into a 5m/15m FVG fill zone).

3. **Displacement must follow** — After the sweep, you need a sharp move in the opposite direction creating a new FVG on your entry TF. This confirms the sweep was a trap, not continuation.

---

## Entry Logic

### Trigger A — Engulfing Candle

- After displacement off the sweep level, wait for a pullback into the newly created FVG
- Engulfing candle forms inside or at the edge of that FVG
- Engulfing body must fully consume prior candle's body
- **Stop market** placed at the engulfing candle's close direction:
  - Buy stop above engulfing high (longs)
  - Sell stop below engulfing low (shorts)
- **Stop loss** = opposite side of the engulfing wick
- **TP** = 1R (mirror the stop distance)

### Trigger B — Pin Bar

- Same setup — sweep → displacement → pullback into FVG
- Pin bar forms with wick rejecting off the FVG zone
- Wick must be ≥ 2x the body length
- Wick must point toward the swept liquidity (into the FVG)
- **Stop market** placed beyond the pin bar's close:
  - Buy stop above pin bar high (longs)
  - Sell stop below pin bar low (shorts)
- **Stop loss** = beyond the pin bar wick tip
- **TP** = 1R

---

## Risk Rules

- Max trades per session: 3
- Max daily loss: 2R
- Risk per trade: Fixed $ or % of account
- 1R target: Equal to stop distance
- Time cutoff: No new entries after 11:30 AM ET
- Kill switch: Stop trading after 2 consecutive losses

---

## Why 1R Works Here

You're not catching a trend — you're trading the snap-back off a liquidity sweep. The displacement already happened, confirming the reversal. You enter on the pullback into the FVG with a tight stop behind a candle showing clear rejection.

- Only need ~50% win rate to break even (minus commissions)
- With these filters, realistic win rate: 55–65%
- 6–8 trades/day at 1R with 60% WR = ~2–3R net daily

---

## Automation Logic

### 1. Sweep Detection
Programmatically flag when price crosses above/below pre-marked DOL levels then reverses within X candles.

### 2. FVG Creation (Three-Candle Logic)
- Bearish FVG: candle 1 high < candle 3 low
- Bullish FVG: candle 1 low > candle 3 high

### 3. Candle Pattern Recognition

**Engulfing:**
- current_body_high > prior_body_high
- AND current_body_low < prior_body_low
- AND opposite direction from prior candle

**Pin Bar:**
- wick_length / body_length >= 2
- AND wick_direction == toward_swept_level

### 4. Order Placement
- Stop market at candle extreme
- SL at opposite extreme
- TP = 1R (calculated from entry to SL distance)

---

## Key Note
The hardest part to automate is DOL identification and sweep confirmation. Everything downstream (FVG detection, candle pattern recognition, order placement) is clean binary logic.
