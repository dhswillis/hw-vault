# NQ Futures Comprehensive Mining Report (V2 — Corrected)

## Critical Corrections from V1

V2 fixes three major issues that inflated V1 results:

1. **Look-ahead bias fixed**: Simulation now starts at `idx+1` (next bar after entry), not `idx`. V1 was checking the entry bar's high/low against targets, but since we enter at the close, the high/low already happened.
2. **Slippage added**: 0.4pt on all market entries (entry price adjusted by 0.4pt in trade direction).
3. **Minimum risk floor**: 1.5pt minimum risk to prevent tiny stops inflating R multiples. V1 had no floor, and 53% of body_stop trades had <3pt risk producing artificial 10R+ winners.

## Dataset

- **34,323 trades** across **258 trading days** (Feb 2025 - Feb 2026)
- **Bar-by-bar R-target simulation** — stop/target checked candle by candle starting from bar after entry
- **7 timeframe MTF alignment** — 1M, 3M, 5M, 10M, 15M, 1H, 4H (trend = 3-bar HH/HL pattern)
- **7 signal types** — BOS_FVG, sweep_wick_sweep, sweep_close_through, sweep_fvg_combo, fvg_exit (body_stop + fvg_stop), swing_failure
- **0.4pt slippage** on all entries; 1.5pt min risk; 25pt max risk

## Signal Type Rankings (V2 Corrected)

### 1. BOS_FVG — The Core Signal (CONFIRMED)

- n=11,348 | BE+Trail WR 63.5% | +15,368R total | R/T +1.35
- Walk-forward: IS 63.6% → OOS 63.4% (**STABLE**)
- Static targets: negative through 1.5R, profitable at 2.0R+
- **Best signal in the dataset by total R and walk-forward stability**
- Fat-tail distribution: 29.5% of trades are 2R+ runners producing +17,461R; remaining 70.5% combined for -2,094R

### 2. FVG Exit FVG Stop — Solid with Trail

- n=3,767 | BE+Trail WR 67.2% | +2,219R total | R/T +0.59
- Walk-forward: IS 67.8% → OOS 66.6% (**STABLE**)
- All static targets negative — must use BE+Trail
- 30.3% of trades end as near-zero breakevens

### 3. FVG Exit Body Stop — Modest After Corrections

- n=1,892 | BE+Trail WR 60.5% | +502R total | R/T +0.27
- Walk-forward: IS 63.5% → OOS 57.3% (slight degradation)
- V1 showed +15,623R and R/T +3.68 — **this was massively inflated by tiny-risk trades**
- With proper min risk floor, returns are modest but still positive

### 4. Sweep Wick Sweep — Best at Long R Targets

- n=8,390 | BE+Trail WR 47.5% | Negative unfiltered
- Optimal: al≥2 co≤2 @ 5.0R target: n=3,045, WR 26.4%, +1,126R, R/T +0.37
- Also profitable at R=1.0 with al≥2: n=4,075, WR 55.1%, +340R
- Walk-forward stable at both exits

### 5. Sweep Close-Through — High WR at Fixed Target

- n=4,905 | BE+Trail WR 47.5% | Negative unfiltered
- Optimal: al≥2 @ 5.0R: n=1,216, WR 35.4%, +883R, R/T +0.73
- Also strong at R=1.0 with al≥2: n=1,216, WR 71.2%, +503R
- Walk-forward: 70.3% → 72.0% (**STABLE**)

### 6. Swing Failure — Modest Edge

- n=3,480 | BE+Trail WR 50.4% | -658R unfiltered
- Optimal: al≥2 @ 5.0R: n=1,087, WR 24.9%, +342R, R/T +0.31
- Walk-forward: 25.8% → 24.1% (stable)

### 7. Sweep FVG Combo — Small but Viable

- n=541 | BE+Trail WR 59.1% | +222R, R/T +0.41
- Walk-forward: 57.9% → 60.4% (stable)
- Low sample size — monitor carefully

## MTF Alignment — Still the Most Important Filter

MTF stacking maintains perfect monotonic improvement in V2:

| TFs Aligned (7TF) | n | BE+Trail WR | R/T | Total R |
|---|---|---|---|---|
| 0 | 7,628 | 40.8% | -0.09 | -694 |
| 1 | 7,504 | 48.2% | +0.26 | +1,949 |
| 2 | 6,143 | 54.5% | +0.68 | +4,194 |
| 3 | 5,159 | 61.2% | +1.18 | +6,102 |
| 4 | 4,028 | 68.1% | +1.80 | +7,251 |
| 5 | 2,757 | 74.0% | +2.38 | +6,550 |
| 6 | 1,104 | 81.0% | +3.24 | +3,577 |

## V10i ALIGNMENT BUG — CRITICAL FINDING

### The Bug

V10i's `get_alignment()` function checks `close > open` on each TF's "last bar" using `tf_c.index <= candles_1m.index[entry_idx]`. This selects the CURRENTLY OPEN higher-TF bar. For example, a 5M bar timestamped 10:00 covers 10:00-10:05, but if entry is at 10:02 this bar is selected despite not being closed yet. The `close` field contains the FUTURE close price.

### Before vs After Fix

| Strategy | Original n | Original WR | Fixed n | Fixed WR | Fixed R |
|---|---|---|---|---|---|
| FA ATR 1.5x + al≥5 + co≤0 | 2,035 | 91.1% | 340 | 37.6% | -110R |

- 0/13 green months after fix. The **entire edge was look-ahead bias**.
- Entry trigger count unchanged (16,583 both versions) — only alignment labels changed.
- With original alignment, unclosed bars that are mid-candle get their future close price checked, inflating alignment counts.

### Our V2 Alignment Is Clean

Our full_mine_v2 uses 3-bar HH/HL trend pattern on completed bars, not single-candle close>open on current bar. **No look-ahead bias.**

## 100% WR Investigation

BOS_FVG + body_confirms + co≤0 @ BE+Trail: 100% WR on 309 trades.

**Verdict: REAL but explained by BE mechanism.**

- All 309 trades reach 1R (min max_r_reached = 1.11R, avg = 7.01R)
- BE always triggers, so worst case is breakeven (near-zero win)
- At fixed R=1.0, same subset shows 91.9% WR — legitimately excellent
- Only 12 trades (3.9%) end as near-breakeven wins (0 < r < 0.1)

## Trade Correlation Analysis

### Cross-Strategy Daily P&L Correlation

| | BOS_FVG | fvg_fvg | fvg_body | wick_sweep | close_thru | swing_fail | sweep_combo |
|---|---|---|---|---|---|---|---|
| BOS_FVG | 1.000 | 0.324 | -0.028 | 0.129 | -0.070 | -0.028 | 0.109 |
| fvg_exit_fvg | 0.324 | 1.000 | 0.350 | -0.044 | -0.019 | 0.035 | 0.157 |
| fvg_exit_body | -0.028 | 0.350 | 1.000 | 0.052 | 0.062 | 0.071 | -0.073 |
| sweep_wick | 0.129 | -0.044 | 0.052 | 1.000 | 0.158 | 0.085 | -0.063 |
| sweep_close | -0.070 | -0.019 | 0.062 | 0.158 | 1.000 | 0.161 | 0.090 |
| swing_fail | -0.028 | 0.035 | 0.071 | 0.085 | 0.161 | 1.000 | 0.041 |
| sweep_combo | 0.109 | 0.157 | -0.073 | -0.063 | 0.090 | 0.041 | 1.000 |

**Key finding:** FVG family moderately correlated internally (0.32-0.35). Sweep family near-zero correlation with FVG family. Average pairwise portfolio correlation: **0.134** — genuine diversification.

### Intra-Day Clustering

| Signal | Trades/Day | Outcome Corr | Independence |
|---|---|---|---|
| BOS_FVG | 44.2 | 57.3% | MODERATE |
| fvg_exit_fvg | 14.7 | 53.7% | MODERATE |
| fvg_exit_body | 7.4 | 49.8% | GOOD |
| sweep_wick | 32.5 | 51.4% | GOOD |
| sweep_close | 19.0 | 51.2% | GOOD |
| swing_failure | 13.5 | 51.2% | GOOD |
| sweep_fvg_combo | 3.0 | 77.5% | LOW (small n) |

## Recommended Portfolio (V2 Final)

### Combined 7-Strategy Portfolio (All Exits)

| Signal | Filters | Exit | n | WR | Total R | R/Trade | WF |
|---|---|---|---|---|---|---|---|
| BOS_FVG | unfiltered | be_trail | 11,348 | 63.5% | +15,368 | +1.354 | 63.6%→63.4% |
| fvg_exit_fvg_stop | unfiltered | be_trail | 3,767 | 67.2% | +2,219 | +0.589 | 67.8%→66.6% |
| fvg_exit_body_stop | unfiltered | be_trail | 1,892 | 60.5% | +502 | +0.265 | 63.5%→57.3% |
| sweep_wick_sweep | al≥2 co≤2 | 5.0R | 3,045 | 26.4% | +1,126 | +0.370 | 27.6%→25.2% |
| sweep_close_through | al≥2 | 5.0R | 1,216 | 35.4% | +883 | +0.726 | 34.9%→35.9% |
| swing_failure | al≥2 | 5.0R | 1,087 | 24.9% | +342 | +0.315 | 25.8%→24.1% |
| sweep_fvg_combo | unfiltered | be_trail | 541 | 59.1% | +222 | +0.411 | 57.9%→60.4% |

### Combined Portfolio Stats

- **Total trades: 22,896**
- **Total R: +20,662**
- **Green days: 211/258 (81.8%)**
- **Green months: 13/13 (100%)**
- **Daily avg: +80.1R**
- **Max cumulative DD: -48.6R**
- **Calmar: 425**
- **Avg pairwise correlation: 0.134**

### Two Distinct Books

1. **FVG Trend-Following Book** (BOS_FVG + fvg_exit variants, BE+Trail): High WR, captures runners. Moderately correlated internally (0.3-0.6).
2. **Sweep Mean-Reversion Book** (wick/close/swing, 5.0R fixed targets): Low WR, big payoffs. Nearly uncorrelated with FVG book (~0.0).

## Strategy Tier Summary

- **533 Snipers** (WR ≥ 70%, n ≥ 50, WF pass) — mostly BOS_FVG + body_confirms variants
- **307 Sharpshooters** (WR ≥ 60%, n ≥ 200, WF pass) — BOS_FVG with moderate filtering
- **30 Gunners** (WR ≥ 50%, n ≥ 500, WF pass) — broader signal types

## BE+Trail Mechanism — Why It Matters

BE+Trail is a trend-following exit: move stop to breakeven at 1R, then trail. This converts losing trades into breakevens (near-zero wins), then lets winners run.

| Signal | BE WR | R=1.0 WR | WR Gap | Near-Zero % | BE avgR | R=1.0 avgR |
|---|---|---|---|---|---|---|
| BOS_FVG | 63.5% | 39.7% | +23.8% | 19.4% | +1.354 | -0.260 |
| fvg_exit_fvg | 67.2% | 38.0% | +29.2% | 30.3% | +0.589 | -0.290 |
| fvg_exit_body | 60.5% | 50.3% | +10.2% | 34.8% | +0.265 | -0.027 |
| sweep_wick | 47.5% | 45.6% | +1.9% | 33.0% | -0.313 | -0.111 |

BOS_FVG's edge is entirely in the trail: 29.5% of trades are 2R+ runners generating +17,461R. The other 70.5% combined for -2,094R. Classic fat-tail strategy.

## Raw Data

- V2 Excel workbook: `NQ_Mining_Results_V2.xlsx` (8 sheets — summary, rankings, portfolio, correlation, monthly, BE analysis, V10i bug, clustering)
- V2 trade data: `full_mine_v2_batches/batch_0-5.json` (34,323 trades)
- V10i replication: `replicate_v10i_batches/` and `replicate_fixed_batches/`
- Scripts: `build_portfolio.py`, `correlation_analysis.py`, `stress_test_v10i.py`
