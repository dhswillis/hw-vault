> ### INVALIDATED 2026-04-10 — DO NOT CITE NUMBERS FROM THIS FILE
>
> Every performance number in this document was produced by **1-minute bar-level simulation**. When the same signals were re-run against tick data on 2026-04-10, the +0.354 avgR / +13.37 R/day / Calmar 151 headline collapsed to **+0.001 avgR / +0.01 R/day**. The edge was a bar-reconstruction artifact.
>
> **Canonical replacement: `BOS_FVG_FAILURE_CONSOLIDATED.md` (same directory).**
>
> This file is preserved for history only. It documents the strategy mechanics honestly and may be useful as a signal-definition reference, but **none of the R-numbers, WR-numbers, or Calmar-numbers below are trustworthy**. CLAUDE.md has been correct on this since March 2026. See `BOS_FVG_FAILURE_CONSOLIDATED.md` for the paired bar-vs-tick comparison, the failure mechanism, and the list of dependent claims that are now also invalidated.

---

# BOS_FVG Strategy — Complete Research Document

**Strategy**: Break of Structure + Fair Value Gap entry with pure trailing stop on NQ futures
**Data**: 312 trading days of Databento tick data (Feb 2025 – Feb 2026)
**Execution**: Conservative simulation with fill-bar pass-through check
**Last Updated**: March 8, 2026 (SUPERSEDED 2026-04-10)

---

## EXECUTIVE SUMMARY

### Single Timeframe (1min, 24H)
**+13.37 R/day | Calmar 151 (trade-level) | 47.7% WR | 100% WWR | 80.4% DWR | Sharpe 15.4**

| Metric | Value |
|--------|-------|
| Trades | 11,786 (37.8/day) |
| Win Rate | 47.7% |
| Avg R | +0.354 |
| Total R | +4,170 |
| R/day | +13.37 |
| Max DD (trade-by-trade) | 27.6R |
| Max DD (daily) | 14.2R |
| Calmar (trade-level) | 151 |
| Calmar (daily) | 294 |
| Daily WR | 80.4% |
| Weekly WR | 100% (54/54) |
| Monthly WR | 100% (13/13) |
| Worst single day | -14.2R |
| Median day | +11.0R |

### Best Multi-TF Portfolio (1min + 5min, 24H)
**+17.95 R/day | Calmar 402 (daily) | 86% DWR**

### Best Session Portfolio (1min, 5 sessions)
**+34.10 R/day | Calmar 327 (daily) | 98% DWR | Walk-forward validated**

---

## 1. STRATEGY MECHANICS

### Entry Flow

**Step 1: Swing Detection (3-bar confirmed)**
- Swing High: bar[i].high is highest in [i-3, i], confirmed by next 3 bars being lower
- Swing Low: mirror logic for lows
- Swing only usable after bar `i + 3` (confirmation delay prevents look-ahead)

**Step 2: Break of Structure (BOS)**
- BOS Long: bar[i].close > most recent confirmed swing high, AND bar[i-1].close was still below
- BOS Short: bar[i].close < most recent confirmed swing low, AND bar[i-1].close was still above
- Minimum displacement: 2.0 pts (close must punch through by ≥2pts)
- Only triggers on FIRST close through (not repeated breaks of same level)

**Step 3: Fair Value Gap (FVG) — 3-candle imbalance**
- Bullish FVG: bar[i].low > bar[i-2].high (gap between candle 1 and candle 3)
- Bearish FVG: bar[i-2].low > bar[i].high
- FVG must form during or within 15 bars after BOS, same direction
- No minimum FVG size filter (tested — edge survives at every size threshold)

**Step 4: Fill → Entry**
- Price retraces into FVG (low touches gap_high for longs, high touches gap_low for shorts)
- Fill checked starting bar AFTER FVG forms, expires after 50 bars
- Entry: limit at FVG edge ± 0.25pt slippage
- Stop: opposite edge of FVG
- Risk must be 1.5–50 pts
- Only 1 FVG fill per BOS (avoids clustering)

**Step 5: Exit (pure trailing stop)**
- Fill-bar pass-through: if stop hit on fill bar → immediate loss
- Trail trigger: 0.5R MFE → activate trail
- Trail buffer: 0.1R behind high-water mark
- Session kill: 20:00 UTC (or session-specific end time)
- Conservative ordering: check stop BEFORE updating trail on each bar

### Key Implementation Files
- **Python backtester**: `clean_backtest.py` (776 lines, single file)
- **MotiveWave study**: `BOSFVGStrategy.java`
- **Mining scripts**: `/tmp/mine_24h.py`, `/tmp/mine_tf_sessions.py`

---

## 2. CALMAR RATIO AUDIT

Calmar varies significantly by calculation method:

| Method | Max DD | Calmar | Notes |
|--------|--------|--------|-------|
| Trade-by-trade cumsum | 27.6R | **151** | Most conservative — captures intraday DD |
| Daily PnL (all days) | 14.2R | 294 | Standard method — masks intraday swings |
| Weekly PnL | 0.0R | ∞ | Every week positive — no weekly drawdown |
| Monthly PnL | 0.0R | ∞ | Every month positive |

**Recommended Calmar: 151 (trade-by-trade)** — this is the honest number.

The daily Calmar of 294 is technically correct but misleading: worst "daily DD" is a single day (-14.2R on Feb 26, 2025) that recovered immediately. Trade-by-trade reveals 27.6R of intraday drawdown that daily aggregation smooths away.

### Daily PnL Distribution
- Mean: +13.37R, Median: +11.02R, Std: 13.79R
- P5: -2.83R, P95: +37.52R
- Worst day: -14.2R, Best day: +60.8R
- Win days: 251, Loss days: 55, Flat: 6

### Annualized Sharpe: 15.38
This is extremely high. Real-world execution friction (latency, slippage, partial fills) will reduce this, but even at 50% degradation it's exceptional.

### Execution Risk: 0.1R Trail Buffer
The tight trail (0.1R = ~0.6pts on median 5.9pt risk = 2-3 ticks NQ) is the source of both the high win rate and the Calmar concern. At 0.25R trail buffer, Calmar drops to ~225 (daily) but is more executable.

---

## 3. TIMEFRAME ANALYSIS

### Per-Timeframe (24H, baseline config)

| TF | Trades/day | WR | avgR | R/day | Calmar (daily) |
|---|---|---|---|---|---|
| **1min** | 37.8 | 47.7% | +0.354 | +13.38 | 295 |
| **3min** | 15.8 | 43.4% | +0.379 | +6.00 | 106 |
| **5min** | 10.1 | 42.0% | +0.445 | +4.48 | 58 |
| **15min** | 3.3 | 35.7% | +0.402 | +1.34 | 20 |

Pattern: higher TF = fewer trades, higher avgR, lower total R/day. Edge present at ALL timeframes.

### Per-Session (1min)

| Session (UTC) | Trades/day | WR | avgR | R/day | Calmar |
|---|---|---|---|---|---|
| Asia (0-8) | 22.6 | 53.5% | +0.386 | +8.74 | 196 |
| London (8-13) | 18.3 | 50.5% | +0.488 | +8.91 | 144 |
| NY AM (13-16) | 13.8 | 42.2% | +0.513 | +7.09 | 74 |
| NY Lunch (16-18) | 8.5 | 44.7% | +0.464 | +3.95 | 70 |
| NY PM (18-20:55) | 11.3 | 45.6% | +0.480 | +5.41 | 90 |
| **24H Combined** | **74.5** | **48.5%** | **+0.458** | **+34.10** | **327** |

Asia has highest WR (53.5%), NY AM has highest avgR (+0.513).

### Per-Session x Per-Timeframe Breakdown

| Session | 1min | 3min | 5min |
|---|---|---|---|
| **Asia** | 22.6/d, 53.5% WR, +0.386, +8.74 R/d, Cal 196 | 9.2/d, 47.3% WR, +0.410, +3.77 R/d, Cal 51 | 5.9/d, 45.7% WR, +0.526, +3.12 R/d, Cal 55 |
| **London** | 18.3/d, 50.5% WR, +0.488, +8.91 R/d, Cal 144 | 7.8/d, 46.5% WR, +0.525, +4.09 R/d, Cal 73 | 5.2/d, 43.7% WR, +0.554, +2.87 R/d, Cal 53 |
| **NY AM** | 13.8/d, 42.2% WR, +0.513, +7.09 R/d, Cal 74 | 6.0/d, 33.4% WR, +0.562, +3.39 R/d, Cal 32 | 3.9/d, 32.5% WR, +0.598, +2.33 R/d, Cal 21 |
| **NY Lunch** | 8.5/d, 44.7% WR, +0.464, +3.95 R/d, Cal 70 | 3.5/d, 42.9% WR, +0.476, +1.69 R/d, Cal 16 | 2.2/d, 40.8% WR, +0.450, +0.99 R/d, Cal 16 |
| **NY PM** | 11.3/d, 45.6% WR, +0.480, +5.41 R/d, Cal 90 | 4.4/d, 39.5% WR, +0.456, +2.00 R/d, Cal 41 | 2.9/d, 36.5% WR, +0.493, +1.43 R/d, Cal 13 |
| **24H** | 74.5/d, 48.5% WR, +0.458, +34.10 R/d, Cal 327 | 30.9/d, 42.8% WR, +0.483, +14.93 R/d, Cal 188 | 20.1/d, 40.8% WR, +0.534, +10.74 R/d, Cal 117 |

---

## 4. MULTI-TIMEFRAME CORRELATION

### Daily PnL Correlation Matrix

|  | 1min | 3min | 5min | 15min |
|---|---|---|---|---|
| 1min | 1.000 | 0.107 | 0.183 | 0.002 |
| 3min | 0.107 | 1.000 | 0.132 | 0.041 |
| 5min | 0.183 | 0.132 | 1.000 | 0.035 |
| 15min | 0.002 | 0.041 | 0.035 | 1.000 |

**Essentially uncorrelated** — highest pair is 1m/5m at 0.183. 15min is zero with everything.

### Trade Overlap (same fill minute)

| Pair | Shared | % of TF1 | % of TF2 |
|---|---|---|---|
| 1m vs 3m | 488 | 4.2% | 9.9% |
| 1m vs 5m | 231 | 2.0% | 7.4% |
| 1m vs 15m | 64 | 0.5% | 6.2% |
| 3m vs 5m | 259 | 5.3% | 8.3% |

~90-98% of trades are unique per timeframe.

### Combined Portfolios (unique trades only)

| Combo | Trades/day | avgR | R/day | Calmar | DWR |
|---|---|---|---|---|---|
| 1m only | 37.8 | +0.354 | +13.38 | 295 | — |
| **1m+5m** | **47.1** | **+0.381** | **+17.95** | **402** | **86%** |
| 1m+3m | 52.0 | +0.371 | +19.29 | 371 | 84% |
| 1m+3m+5m | 60.6 | +0.390 | +23.66 | 335 | 85% |
| 1m+3m+5m+15m | 63.4 | +0.389 | +24.66 | 324 | 86% |

**1m+5m is the sweet spot** — Cal 402 (highest), adds +4.57 R/day.

---

## 5. WALK-FORWARD VALIDATION

### 24H Combined (1min)
| Period | Trades/day | avgR | R/day | Calmar |
|---|---|---|---|---|
| H1 In-Sample | 75.1 | +0.441 | +33.13 | 159 |
| **H2 Out-of-Sample** | **73.9** | **+0.475** | **+35.07** | **501** |

OOS is BETTER than IS — no overfitting. avgR improves from +0.441 to +0.475.

### Per-Timeframe Walk-Forward
| TF | H1 IS avgR | H2 OOS avgR | Pass? |
|---|---|---|---|
| 1min | +0.441 | +0.475 | YES |
| 3min | +0.493 | +0.472 | YES |
| 5min | +0.555 | +0.512 | YES |

All timeframes walk-forward validated.

---

## 6. PARAMETER SENSITIVITY

### FVG Minimum Size Sweep
Edge survives at every threshold — avgR stable across all sizes:

| Min Size | Trades | WR | avgR | R/day | Calmar |
|---|---|---|---|---|---|
| 0 (no filter) | 4,429 | 50.7% | +0.417 | +7.14 | 62 |
| 1.0 | 4,269 | 50.6% | +0.413 | +6.82 | 63 |
| 2.0 | 3,935 | 50.5% | +0.399 | +6.07 | 52 |
| 5.0 | 2,452 | 50.2% | +0.382 | +3.63 | 36 |
| 10.0 | 1,092 | 47.9% | +0.365 | +1.54 | 16 |

### Fill-Bar Pass-Through Fix Impact
Without check: +0.895 avgR (inflated — small FVGs get free looks)
With check: +0.417 avgR (honest — still very profitable)

### Trail Buffer Sensitivity (from V10z)
| Trail Buffer | avgR | Calmar |
|---|---|---|
| 0.1R (tight) | +1.141 | 264 |
| 0.25R (recommended) | +0.895 | 226 |
| 0.5R (wide) | +0.670 | 118 |

0.25R recommended for live execution safety (1.5 pts = 6 ticks).

---

## 7. CONFLUENCE FILTER ANALYSIS (Leppyrd)

All Leppyrd-style filters tested — none improve BOS_FVG:

| Filter | Trades | WR | avgR | totR |
|---|---|---|---|---|
| **Baseline (no filter)** | 4,429 | 50.7% | +0.417 | +1,846 |
| Zone-aligned only | 2,148 | 50.4% | +0.404 | +867 |
| Aligned + ADR<75% cont | 1,786 | 50.9% | +0.421 | +752 |
| Aligned reversals only | 897 | 51.3% | +0.432 | +388 |

Premium/discount zones, ADR exhaustion, day-of-week, continuation vs reversal — none improve the edge. The signal is structural and self-contained.

---

## 8. BUGS FOUND AND FIXED

### Bad Tick Data (March 2026 Audit)
- 601,050 bad ticks across 306/312 days
- Prices ~200-500 (likely MNQ trades in NQ feed)
- Fix: `df = df[(df['price'] >= 10000) & (df['price'] <= 30000)]`
- **Impact on BOS_FVG: NONE** — context engine symbol filter already excluded bad ticks
- **Impact on doji break: FATAL** — strategy completely dead with clean data

### Fill-Bar Pass-Through (March 2026)
- Sim didn't check stop on fill bar when price goes through entire FVG
- Small FVGs (<2pt) had inflated +2.650 avgR from "free looks"
- Fix: check if fill bar low/high violates stop → immediate loss
- avgR dropped from +0.895 to +0.417, Calmar from 237 to 62

### B/E Phantom Fills (March 2026 — doji break only)
- B/E activated on TIME only, not checking if price reached the level
- Fixed with HWM tracking: require `hwm >= be_stop` before activation
- Only affected doji break strategy, not BOS_FVG

---

## 9. DEAD ENDS (DO NOT RE-TEST)

- **Leppyrd confluence filters**: Premium/discount, ADR, zone alignment — none improve BOS_FVG
- **B/E stops**: Kill the edge at every trigger level (confirmed across 11 configs)
- **Inverse trades**: Net negative (-0.220 avgR)
- **Multi-TF alignment**: All 5 clean methods dead (V10p) — entire edge was look-ahead bias
- **Time stops**: Zero difference to WR or avgR
- **Partial profits without alignment**: Don't improve anything
- **All non-alignment structural filters**: Cluster, body ratio, volume, displacement, VWAP, FVG presence — all dead

---

## 10. LOOK-AHEAD BIAS AUDIT

| Component | Audit | Clean? |
|---|---|---|
| Swing detection | 3-bar confirmation delay, only uses bars with confirmed_idx < current | YES |
| BOS | Checks close vs confirmed swings, requires prior bar below/above | YES |
| FVG | 3-candle pattern, no future data | YES |
| Fill check | Starts bar AFTER FVG forms | YES |
| Trail stop | Updates sequentially, worst-case ordering | YES |
| Fill-bar stop | Conservative: checks stop on fill bar | YES |
| Bad tick filter | Price range filter, no data dependency | YES |

---

## 11. DATA & INFRASTRUCTURE

### Tick Data
- **Source**: Databento (NQ front-month trades)
- **Location**: `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/`
- **Coverage**: 312 files, Feb 2025 – Feb 2026

### Running Scripts
```bash
cd /tmp && /Users/harrisonwillis/Documents/trading-system/.venv/bin/python <script.py>
```
Must run from /tmp to avoid `operator/` module shadowing.

### Key Scripts
| Script | Purpose |
|---|---|
| `clean_backtest.py` | Primary backtester (776 lines) |
| `/tmp/mine_24h.py` | 24H session-specific mining |
| `/tmp/mine_tf_sessions.py` | Per-session per-TF stats |
| `/tmp/tf_correlation.py` | Multi-TF correlation analysis |
| `/tmp/calmar_audit.py` | Calmar calculation audit |
| `/tmp/fvg_size_sweep.py` | FVG min size parameter sweep |
| `/tmp/leppyrd_filters.py` | Leppyrd confluence filters |
