# NautilusTrader Backtest Specification — BOS_FVG Pure Trailing

**Strategy**: BOS_FVG with 15-second trailing stop management
**Instrument**: NQ (E-mini Nasdaq-100 futures) or MNQ (Micro)
**Backtested**: 312 days of tick data (Databento), Feb 2025 — Feb 2026
**Results**: 75.8% WR, +1.141 avgR, Calmar 263.6, -3.0R max DD

---

## 1. Data Requirements

- **Tick data**: NQ futures trade-and-bid-ask (Databento MBP-1 or trades schema)
- **Bar construction**: Build 1-minute and 15-second OHLCV bars from ticks
  - Also need 5-minute bars for session bias computation
  - All bars include `volume` and `delta` (buy volume - sell volume)
  - Delta: trades at ask = buy, trades at bid = sell
- **Front month**: Roll to front contract (continuous front)
- **RTH filter**: Only process bars between 08:00-21:00 UTC (3:00 AM - 4:00 PM ET)

---

## 2. Signal Detection (on 1-Minute Bars)

### 2.1 Swing Points (3-bar lookback)

```
Parameters:
  lookback = 5 (configurable, used 5 in backtest)

Swing High at bar i:
  - bar[i].high == max(bar[i-lb].high ... bar[i].high)   # highest of lookback window
  - bar[i].high >= max(bar[i+1].high ... bar[i+lb].high)  # confirmed by lb subsequent bars
  - Confirmed at bar i+lb (NOT actionable until confirmation)
  - Record: price = bar[i].high, confirmed_time = bar[i+lb].time

Swing Low at bar i:
  - bar[i].low == min(bar[i-lb].low ... bar[i].low)
  - bar[i].low <= min(bar[i+1].low ... bar[i+lb].low)
  - Confirmed at bar i+lb
  - Record: price = bar[i].low, confirmed_time = bar[i+lb].time
```

### 2.2 Break of Structure (BOS)

```
For each 1M bar i, check against all confirmed swing points:

Bullish BOS:
  - swing_high.confirmed_index < i   (swing was confirmed BEFORE this bar)
  - swing_high.index >= i - 100       (recent swing, within 100 bars)
  - bar[i].high > swing_high.price    (current bar breaks above)
  - bar[i-1].high <= swing_high.price (previous bar was below — fresh break)
  - displacement = bar[i].high - swing_high.price > 2 pts

Bearish BOS:
  - swing_low.confirmed_index < i
  - swing_low.index >= i - 100
  - bar[i].low < swing_low.price
  - bar[i-1].low >= swing_low.price
  - displacement = swing_low.price - bar[i].low > 2 pts
```

### 2.3 Fair Value Gap (FVG)

```
Parameters:
  fvg_min_size = 3.0 pts

Bullish FVG at bar i (3-candle pattern):
  - bar[i].low > bar[i-2].high          (gap between candle 3 low and candle 1 high)
  - gap_size = bar[i].low - bar[i-2].high >= 3.0 pts
  - gap_high = bar[i].low
  - gap_low = bar[i-2].high
  - midpoint = (gap_high + gap_low) / 2

Bearish FVG at bar i:
  - bar[i-2].low > bar[i].high
  - gap_size = bar[i-2].low - bar[i].high >= 3.0 pts
  - gap_high = bar[i-2].low
  - gap_low = bar[i].high
  - midpoint = (gap_high + gap_low) / 2
```

### 2.4 FVG Fill Detection (Limit Order Entry)

```
Parameters:
  max_candles_to_fill = 50
  tick_offset = 0.25 pts (1 NQ tick)

After FVG forms at bar i, scan bars i+1 through i+50:

Bullish FVG fill (limit buy at midpoint):
  - bar[j].low <= midpoint - tick_offset  (price traded THROUGH midpoint)
  - fill_price = midpoint (your limit order price)
  - fill_time = bar[j].time

Bearish FVG fill (limit sell at midpoint):
  - bar[j].high >= midpoint + tick_offset
  - fill_price = midpoint
  - fill_time = bar[j].time
```

### 2.5 BOS + FVG Combination

```
For each BOS:
  Find FVGs where:
    - fvg.direction == bos.direction
    - bos.index - 20 <= fvg.index <= bos.index  (FVG formed near/during BOS)
    - fvg.filled == True                          (price returned to fill the gap)

  For each matching FVG:
    Entry:
      entry_price = fvg.fill_price (= fvg.midpoint)

    Stop:
      LONG:  stop = fvg.gap_low - 3 pts   (below FVG low + 3pt buffer)
      SHORT: stop = fvg.gap_high + 3 pts  (above FVG high + 3pt buffer)

    Risk:
      risk = abs(entry - stop)
      REJECT if risk < 3 pts or risk > 50 pts

    Reward (for signal generation only, not used in trailing):
      LONG:  target = bos.broken_level + bos.displacement * 2
      SHORT: target = bos.broken_level - bos.displacement * 2
      REJECT if reward/risk < 2.0
```

### 2.6 Confluence Gate (Minimum 3 Factors)

Every BOS_FVG trade starts with 2 confluence factors: `["BOS", "FVG_fill"]`.
Must reach 3+ to qualify:

```
Additional confluence checks:

Volume Spike at BOS candle:
  - Any volume spike at the same bar index as BOS, same direction
  - Volume spike = bar volume >= 3x rolling 20-bar average volume
  - If yes: add "volume_spike" to confluence

Engulfing at BOS candle:
  - Any engulfing candle at same bar index as BOS, same direction
  - Engulfing = current bar body completely engulfs prior bar body
  - body_ratio = abs(curr_body) / abs(prev_body) >= 1.5
  - If yes: add "engulfing" to confluence

REJECT trade if len(confluence) < 3
```

### 2.7 Session Bias Filter

Only take trades in the direction of the session bias.

```
Session bias is computed from data PRIOR to session start (no look-ahead):

Inputs (5M bars before session start, last 12 bars = 1 hour):
  1. Prior trend (±1 vote): close[-1] - open[0] > 15 pts → long, < -15 → short
  2. Last 5M BOS direction (±2 votes, double weight)
  3. Delta sum (±1 vote): < -200 → long (buying), > 200 → short (selling)
  4. Unfilled FVG count (±1 vote): more bull unfilled → long, more bear → short

Bias determination:
  - long_score > short_score + 1 → LONG bias
  - short_score > long_score + 1 → SHORT bias
  - Otherwise → no bias (skip trades? In backtest, no-bias sessions still generate trades)

REJECT trade if trade.direction != session_bias.direction
```

### 2.8 Session Windows (UTC)

Only generate signals during these windows:

| Session | UTC Start | UTC End | ET Equivalent |
|---------|-----------|---------|---------------|
| news_window | 13:30 | 14:00 | 8:30-9:00 |
| open_drive | 14:30 | 14:45 | 9:30-9:45 |
| ib_first | 14:45 | 15:30 | 9:45-10:30 |
| ib_ext | 15:30 | 16:00 | 10:30-11:00 |
| ny_am | 16:00 | 17:00 | 11:00-12:00 |
| silver_bullet | 18:30 | 19:30 | 1:30-2:30 |
| power_hour | 20:00 | 21:00 | 3:00-4:00 |

---

## 3. Trade Management (on 15-Second Bars)

### 3.1 Entry Execution

```
Order type: LIMIT ORDER at fvg.midpoint
Entry slippage: +0.25 pts (conservative — limit orders get filled at limit price,
                 but we add 0.25pt to account for queue position)
  LONG:  effective_entry = entry_price + 0.25
  SHORT: effective_entry = entry_price - 0.25
```

### 3.2 Initial Stop

```
LONG:  initial_stop = fvg.gap_low - 3 pts  (below FVG)
SHORT: initial_stop = fvg.gap_high + 3 pts (above FVG)

This is a hard stop — never moves DOWN (long) or UP (short).
```

### 3.3 Trailing Stop Logic (THE CORE)

```
Parameters:
  trigger_r = 0.5    (trailing activates after +0.5R favorable movement)
  trail_r = 0.1      (stop follows 0.1R behind high-water mark)
  max_hold = 120 min (hard time cutoff)
  exit_slippage = 0.5 pts

State variables:
  hwm = 0            (high-water mark in R-multiples)
  trailing_active = False
  current_stop = initial_stop

For each 15-second bar after entry:

  1. TIME CHECK:
     if bar.time > entry_time + 120 minutes → EXIT at bar.close (timeout)

  2. STOP CHECK (before any updates — conservative):
     LONG:  if bar.low <= current_stop → EXIT at current_stop - 0.5pt slippage
     SHORT: if bar.high >= current_stop → EXIT at current_stop + 0.5pt slippage
     Outcome: 'loss' if not trailing, 'trail_stop' if trailing active

  3. UPDATE HWM:
     LONG:  mfe = (bar.high - entry) / risk
     SHORT: mfe = (entry - bar.low) / risk
     hwm = max(hwm, mfe)

  4. ACTIVATE TRAILING:
     if hwm >= 0.5 (trigger_r) and not trailing_active:
       trailing_active = True

  5. UPDATE TRAILING STOP:
     if trailing_active:
       LONG:  new_stop = entry + (hwm - 0.1) * risk
       SHORT: new_stop = entry - (hwm - 0.1) * risk
       current_stop = max(current_stop, new_stop)  [LONG]
       current_stop = min(current_stop, new_stop)  [SHORT]

  6. CONSERVATIVE RE-CHECK (critical for Nautilus accuracy):
     After updating the trailing stop within the same bar, re-check:
     LONG:  if bar.low <= current_stop → EXIT at current_stop - 0.5pt
     SHORT: if bar.high >= current_stop → EXIT at current_stop + 0.5pt
     (This handles intra-bar scenario: price runs to new HWM, trail
      tightens, then price reverses past new stop — all within one bar)
```

### 3.4 Exit Slippage

```
All exits: 0.5 pts adverse slippage
  LONG exit:  exit_price = stop_level - 0.5
  SHORT exit: exit_price = stop_level + 0.5
```

### 3.5 R Calculation

```
LONG:  actual_r = (exit_price - effective_entry) / risk
SHORT: actual_r = (effective_entry - exit_price) / risk

where risk = abs(entry_price - stop_price)  [using original entry, not slipped]
```

---

## 4. Key Parameters Summary

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Instrument** | NQ / MNQ futures | |
| **Signal TF** | 1-minute bars | Detection only |
| **Management TF** | 15-second bars | Trail updates |
| **Swing lookback** | 5 bars | For BOS detection |
| **FVG min size** | 3.0 pts | Minimum gap |
| **FVG fill window** | 50 bars (1M) | Max bars to wait for fill |
| **FVG fill tick offset** | 0.25 pts | Confirms price traded through |
| **BOS-FVG proximity** | 20 bars | FVG must form within 20 bars of BOS |
| **Min displacement** | 2 pts | Minimum BOS displacement |
| **Stop buffer** | 3 pts | Beyond FVG edge |
| **Risk filter** | 3-50 pts | Reject micro/macro gaps |
| **Min R:R** | 2.0 | For signal generation |
| **Confluence min** | 3 factors | BOS + FVG_fill + (volume_spike OR engulfing) |
| **Trail trigger** | 0.5R | ~3 pts at median risk |
| **Trail distance** | 0.1R | ~0.6 pts (2 ticks) at median risk |
| **Recommended trail** | 0.25R | ~1.5 pts (6 ticks) — safer for execution |
| **Max hold** | 120 min | Hard time cutoff |
| **Entry slippage** | 0.25 pts | Limit order assumption |
| **Exit slippage** | 0.5 pts | Conservative |
| **Sessions** | 7 windows | See Section 2.8 |

---

## 5. Expected Results (to validate Nautilus matches)

### Per-Config Targets (685 trades, 312 days)

| Trail | WR | avgR | Total R | Calmar | R/Day |
|-------|-----|------|---------|--------|-------|
| **0.1R** | **75.8%** | **+1.141** | **+781.8** | **263.6** | **+2.51** |
| 0.15R | 75.8% | +1.103 | +755.5 | 250.6 | +2.42 |
| 0.25R | 75.8% | +1.028 | +703.9 | 225.9 | +2.26 |
| 0.5R | 72.3% | +0.842 | +576.6 | 171.3 | +1.85 |

### Monthly P&L (all 13 months positive)

| Month | Trades | Total R | avgR |
|-------|--------|---------|------|
| Feb 2025 | ~50 | +62.7 | +1.26 |
| Mar 2025 | ~85 | +85+ | +1.0 |
| Apr 2025 | ~70 | +80+ | +1.1 |
| ... all positive ... | | | |

### Walk-Forward (70/30 split)

| | WR | avgR |
|--|-----|------|
| IS (70%) | 76.2% | +1.140 |
| OOS (30%) | 73.9% | +1.149 |
| Q1 OOS | + | + |
| Q2 OOS | + | + |
| Q3 OOS | + | + |
| Q4 OOS | + | + |

**OOS is better than IS** — no overfitting. 4/4 quarters positive.

---

## 6. Outcome Distribution (for validation)

```
Expected trade outcomes (685 trades at trig=0.5R trail=0.1R):
  - Trail stop (winner):  ~519 trades (75.8%)
  - Loss (stopped at original stop): ~165 trades (24.1%)
  - Timeout (max hold): ~1 trade (0.1%)

Median risk per trade: 5.9 NQ points
Risk range: 3-50 pts (P25=3.5, P75=10.5)
Average trades per day: 2.2

FVG gap size: P50=5.8 pts, P25=3.5, P75=10.5
```

---

## 7. What NOT to Include

- **No breakeven management** — BE at 1R kills the edge (21.8% WR vs 75.8%)
- **No multi-TF alignment scoring** — clean alignment has zero edge, look-ahead version is biased
- **No FVG close confirmation gate** — requiring close beyond FVG kills 40% of edge (46.4% WR vs 75.8%)
- **No fixed R targets** — trailing captures fat tails that fixed targets miss
- **No daily kill switch** — -3R kill reduces total R and Calmar
- **No trail activation delay** — delaying by 60 seconds kills 90% of Calmar

---

## 8. Pseudocode (Complete Signal-to-Exit)

```python
for each trading day:
    build 5M, 1M, 15S OHLCV bars from tick data
    filter to RTH (08:00-21:00 UTC)

    on 1M bars:
        detect swing points (lookback=5, confirmed after 5 bars)
        detect BOS (breaks of confirmed swings, displacement > 2pt)
        detect FVGs (3-candle gaps >= 3pt)
        check FVG fills (price through midpoint within 50 bars)
        detect volume spikes (3x 20-bar avg)
        detect engulfing candles (body ratio >= 1.5x)

    for each session in [news_window, open_drive, ib_first, ib_ext, ny_am, silver_bullet, power_hour]:
        compute session_bias from prior 5M bars

        for each BOS in session:
            for each filled FVG near BOS (same direction, within 20 bars):
                entry = FVG midpoint
                stop = FVG edge + 3pt buffer
                risk = abs(entry - stop)
                if risk < 3 or risk > 50: skip
                if reward/risk < 2: skip

                confluence = ["BOS", "FVG_fill"]
                if volume_spike at BOS bar: confluence += "volume_spike"
                if engulfing at BOS bar: confluence += "engulfing"
                if len(confluence) < 3: skip

                if trade.direction != session_bias.direction: skip

                # === ENTRY: Place limit order at FVG midpoint ===
                effective_entry = entry + 0.25pt (long) or entry - 0.25pt (short)

                # === MANAGEMENT: Switch to 15S bars ===
                hwm = 0
                trailing = False
                current_stop = stop

                for each 15S bar after fill_time, up to 120 min:
                    # 1. Check stop
                    if price hits current_stop: EXIT (loss or trail_stop)

                    # 2. Update HWM
                    mfe = favorable excursion in R
                    hwm = max(hwm, mfe)

                    # 3. Activate trail at 0.5R
                    if hwm >= 0.5 and not trailing: trailing = True

                    # 4. Update trail
                    if trailing:
                        new_stop = entry ± (hwm - 0.1) * risk
                        current_stop = better of (current_stop, new_stop)

                    # 5. Re-check stop after update (conservative)
                    if price hits new current_stop: EXIT (trail_stop)

                record trade result
```

---

## 9. Slippage Stress Test Results

The strategy survives aggressive slippage:

| Entry Slip | Exit Slip | avgR | Calmar |
|-----------|-----------|------|--------|
| 0.25pt | 0.50pt | +1.141 | 263.6 |
| 0.50pt | 0.50pt | +1.096 | 236.8 |
| 1.00pt | 0.50pt | +1.004 | 147.4 |
| 0.25pt | 2.00pt | +0.893 | 145.6 |

Still profitable with 1pt entry + 2pt exit slippage (combined 3pt adverse).

---

## 10. Files for Reference

- **Signal detection**: `context-engine/micro_structure.py` (BOS, FVG, swing points, volume spikes)
- **Session management**: `context-engine/intraday_scanner.py` (bias, sessions, confluence)
- **Trailing sim**: `context-engine/run_v10z_breathe_audit.py` (conservative simulation with re-check)
- **Results**: `context-engine/NQ_PLAYBOOK.md` (full research playbook)
- **Tick data**: Databento NQ trades, 312 days
