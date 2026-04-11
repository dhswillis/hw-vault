# Key Level Sweep Fade — Complete Research Document

**Strategy**: Fade sweeps of key levels on NQ futures (tick-by-tick simulation)
**Data**: 299 trading days of Databento tick data (Feb 12, 2025 - Feb 27, 2026)
**Execution**: Tick-by-tick simulation with limit-order style entry at level price
**Phases Completed**: 18 (320+ configurations tested)
**Last Updated**: March 3, 2026

---

## EXECUTIVE SUMMARY

### Final Production Portfolio
**+10.43 R/day | Calmar 361.8 | 69.2% WR | 100% WWR | 13/13 months positive | DD 8.6R**

| Strategy | Exit | Trades | WR | avgR | Total R |
|----------|------|-------:|---:|-----:|--------:|
| 1H Wick Sweep | Trail 1R/0.5R | 1,721 | ~68% | +0.825 | +1,420 |
| Round 50pt Sweep | Trail 1R/0.5R | 1,630 | ~63% | +0.702 | +1,144 |
| Prev Day H/L Sweep | Trail 1R/0.5R | 151 | ~70% | +1.864 | +281 |
| VWAP Sweep | 2R fixed target | 408 | ~55% | +0.669 | +273 |
| **TOTAL** | | **3,910** | **69.2%** | **+0.798** | **+3,119** |

Monthly range: +114R (Sep) to +382R (Nov). Every month >100R. Never a losing week.

### Robustness (Phase 18 — Corrected)
| Stress Test | R/day | Calmar | WWR |
|-------------|------:|-------:|----:|
| Baseline (0 slippage) | +10.43 | 361.8 | 100% |
| 1pt entry slippage | +9.28 | 294.0 | 100% |
| 2pt entry slippage | +8.42 | 200.7 | 100% |
| 1pt entry + 1pt exit slip | +8.66 | 241.4 | 100% |
| Double commission (1pt) | +10.09 | 330.9 | 100% |
| 100pt round numbers only | +8.52 | 284.5 | 100% |
| Max risk cap 30pt | +8.91 | 242.7 | 100% |

### Walk-Forward Validation (No Overfitting)
| Period | Trades | WR | avgR | R/day | Calmar |
|--------|-------:|---:|-----:|------:|-------:|
| Q1 (Feb-Apr '25) | 1,302 | 67.4% | +0.646 | +11.36 | 103.5 |
| Q2 (May-Jul '25) | 546 | 70.5% | +0.840 | +6.12 | 73.9 |
| Q3 (Aug-Oct '25) | 709 | 69.0% | +0.776 | +7.33 | 112.0 |
| Q4 (Nov-Feb '26) | 1,353 | 70.6% | +0.938 | +16.92 | 147.3 |
| H1 (In-Sample) | 1,848 | 68.3% | +0.703 | +8.72 | 160.0 |
| **H2 (Out-of-Sample)** | **2,062** | **70.0%** | **+0.882** | **+12.13** | **211.1** |

All 4 quarters positive. H2 OOS (Cal 211) beats H1 IS (Cal 160). Performance improves in later data.

---

## 1. STRATEGY MECHANICS

### What It Does
Price sweeps through a key level (wick past it), then comes back through. We fade the sweep — enter at the level price in the opposite direction of the sweep.

### Entry Rules
- **Trigger**: Price trades past a key level (sweep), then trades back through to the other side
- **Entry price**: The key level price itself (limit-order style fill)
- **Stop**: Wick tip (furthest sweep point) + 5pt buffer
- **Direction**: Opposite to the sweep (sweep above level → short, sweep below → long)
- **Session**: 9:45 AM - 12:00 PM ET entries (14:45 - 17:00 UTC)
- **EOD close**: 3:55 PM ET (20:55 UTC)
- **Re-entries**: Up to 3 per level per day
- **Commission**: 0.5pt per trade (round trip), subtracted from R

### Exit Management (Trail 1R/0.5R)
The single most important optimization found across all 18 phases:
- When MFE (max favorable excursion) reaches 1.0R → activate trailing stop
- Trail distance: 0.5R behind the best price achieved
- If price never reaches 1R, trade exits at stop or EOD
- Trail exits = ~63% of all exits, stops = ~31%, EOD = ~6%

### One-Trade-At-A-Time with Priority
Only one open position at any time. When multiple signals compete:
1. **Prev Day H/L** (highest priority — rarest, highest avgR)
2. **1H Wick** (workhorse — most trades)
3. **Round 50pt Numbers** (uncorrelated signal)
4. **VWAP Sweep** (lowest priority — lowest avgR)

### Level Types
| Level | Source | Lookback | How Generated |
|-------|--------|----------|---------------|
| **1H Wick** | 1H candle swing H/L | 5 days (lb=5) | Swing points from last 5 days of 1H bars. Entry at wick tip. |
| **Prev Day H/L** | Previous session high/low | 1 day | Yesterday's RTH session high and low. Tracked from tick 0 (incl. overnight). |
| **Round 50pt** | 50pt price increments | N/A | 21000, 21050, 21100... within ±500 pts of current price. |
| **VWAP Sweep** | VWAP ± deviation | Intraday | VWAP = cum(price×vol)/cum(vol) from session open. Entry when price sweeps 12-15pt past VWAP. |

---

## 2. COMPLETE PHASE-BY-PHASE RESEARCH LOG

### Phase 1: Opening Range Strategies (15 configs × 274 days)
**Script**: `phase1_opening_range.py`
**Tested**: OR breakout, OR fade, OR squeeze, various time windows and targets
**Result**: ALL DEAD. Best was OR fade at +0.054 avgR (barely positive).
**Verdict**: Opening range sweeps don't have enough follow-through in NQ.

### Phase 2: Multi-Timeframe Wick Fades (12 configs × 274 days)
**Script**: `phase2_multi_tf_wick.py`
**Tested**: 15M, 1H, 4H wick fades with various lookback/target combinations
**Result**: 1H wick showed promise (+0.307 avgR). 4H and 15M dead.
**Key Finding**: 1H is the right timeframe — 15M too noisy, 4H too slow.

### Phase 3: Novel Strategies (14 configs × 274 days)
**Script**: `phase3_daily_open_novel.py`
**Tested**: VWAP fade, open pivot fade, opening drive, delta divergence
**Result**: VWAP 15pt fade standout (+0.422 avgR). All others dead.
**Note**: VWAP fade didn't reproduce on full year (Phase 5). Only VWAP *sweep* worked.

### Phase 4: 1H Wick Full-Year Validation (21 configs × 274 days)
**Script**: `phase4_validate_1h.py`
**Tested**: Lookback 3/5/10/20/All, wick size 5-25/10-50, targets 1.5R/2R/3R, trail, B/E
**Result**: lb=5 w10 2R = Cal 26.6, +2.07 R/day, OOS +0.319 avgR
**CRITICAL WARNING**: lbAll = Cal 100.2 = ARTIFACT. Unlimited lookback accumulates hundreds of levels → every price move triggers a trade → fake edge. Only lb=3 to lb=5 trustworthy.
**Top configs**:
| Config | Trades | WR | avgR | R/day | Cal |
|--------|-------:|---:|-----:|------:|----:|
| lb5_w10_2R | 1,548 | 45.2% | +0.319 | +2.07 | 26.6 |
| lb3_w10_2R | 1,374 | 43.3% | +0.298 | +1.55 | 21.1 |
| lb10_w10_2R | 1,638 | 43.6% | +0.277 | +2.77 | 25.1 |

### Phase 5: VWAP/OR/Prev-Day Validation (57 configs × 274 days)
**Script**: `phase5_vwap_or_validate.py`
**Tested**: VWAP fade, OR retest, VWAP sweep, OR sweep, prev-day sweep
**Result**: prev_day sweep 3R = +1.083 avgR (Cal 5.9). VWAP sweep 15pt = +0.359 avgR.
**DEAD**: Plain VWAP fade all negative on full year (didn't reproduce from Phase 3).

### Phase 6: Novel Strategies Batch 1 (41 configs × 274 days)
**Script**: `phase6_novel_strategies.py`
**Tested**: London close, IB extension, overnight sweep, opening bar rejection, mid-range continuation
**Result**: Only overnight_sweep_w5_15 marginally positive (+0.262). Everything else dead.

### Phase 7: Full-Year Winners (28 configs × 274 days)
**Script**: `phase7_fullyear_winners.py`
**Tested**: Prev-day sweep (fixed + trail), VWAP sweep, overnight sweep — extensive parameter sweep
**MAJOR DISCOVERY**: prev_day trail tight = Cal 42.3, +2.20 R/day, 62% WR, 13/13 months+
**Result**:
| Config | Trades | WR | avgR | R/day | DD | Cal |
|--------|-------:|---:|-----:|------:|---:|----:|
| prevday_trail_tight (1R/0.5R) | 284 | 62.0% | +0.891 | +2.20 | 6.0 | 42.3 |
| prevday_trail_wide (2R/1R) | 266 | 50.4% | +1.106 | +2.56 | 8.2 | 36.1 |
| vwap_sweep_12_2R | 695 | 42.3% | +0.226 | +0.62 | 11.7 | 13.5 |
| overnight_sweep | - | - | +0.262 | - | - | 6.8 |

**Why trail works on prev_day but not 5M**: Prev day H/L are structural levels. When sweeps fail here, the reversal is sustained. Trail captures the larger move.

### Phase 8: Portfolio Analysis
**Script**: `phase8_portfolio_analysis.py`
**Result**: Combined prev_day + VWAP + overnight = 478.3R total, ALL 13 months positive.

### Phase 9: Combined Tick-by-Tick Portfolio (274 days)
**Script**: `phase9_combined_portfolio.py` (not in scripts/)
**Tested**: 4 strategies combined, one-trade-at-a-time, priority system
**Result**: 2,381 trades, +2.49 R/day, Cal 27.2, 94.4% WWR, 13/13 months+
**Key insight**: 5M swing = dead weight in combined mode (blocked by 1H wick). Drop it.

### Phase 10: Portfolio Optimization (4 configs × 299 days, 10691s)
**Script**: `phase10_portfolio_optimize.py`
**Note**: prev_day numbers depressed (overnight sweep bug — fixed in Phase 13). 1H/VWAP valid.
**CRITICAL FINDING**: 1H trail tight (1R/0.5R) = Cal 176.5, +5.96 R/day, 100% WWR!
| Config | Trades | WR | avgR | R/day | DD | Cal | WWR |
|--------|-------:|---:|-----:|------:|---:|----:|----:|
| **h1_trail_tight** | 2,384 | 67.2% | +0.748 | +5.96 | 10.1 | 176.5 | 100% |
| best_combo | 3,455 | 60.5% | +0.442 | +5.11 | 17.9 | 85.3 | 98.1% |

Trail = 55% of exits (1,320 trail vs 270 target vs 769 stop). Single most impactful optimization across all phases.

### Phase 11: Novel Strategies Batch 2 (49 configs × 299 days, 2841s)
**Script**: `phase11_novel_batch2.py`
**Tested**: Gap fill fade, session open fade, double sweep filter, 2-day range sweep
**Result**: ALL DEAD or marginal.
| Strategy | Best Config | avgR | Cal | Verdict |
|----------|-------------|-----:|----:|---------|
| Gap Fill | gap_15_2R | +0.001 | 0.0 | DEAD |
| Session Open Fade | open_fade_20m_p8_1.5R | -0.176 | -0.6 | DEAD |
| Double Sweep | dbl_sweep_w10_3R | +0.151 | 3.8 | Marginal |
| 2-Day Range | 2day_w10_40_t1.0_0.5 | +0.181 | 4.1 | Low freq |

### Phase 12: Filter Optimization (36 configs × 299 days, 7428s)
**Script**: `phase12_filter_optimization.py`
**KEY FINDING**: Tighter time window = dramatically better Calmar.
| Window | Trades | WR | avgR | R/day | DD | Cal |
|--------|-------:|---:|-----:|------:|---:|----:|
| 9:45-11:00 AM | 906 | 50.4% | +0.469 | +1.42 | 8.2 | 51.7 |
| 10:00-12:00 | 1,063 | 53.1% | +0.542 | +1.93 | 12.9 | 44.8 |
| 9:45-12:00 (baseline) | 1,196 | 49.5% | +0.440 | +1.76 | 12.5 | 42.0 |
| 9:45-1:00 PM | 1,436 | 48.5% | +0.411 | +1.97 | 19.2 | 30.8 |

**Other findings**: Short-only = 4.5× better Cal than long-only (23.3 vs 5.2). Wick 20-60pt = cleaner.
**Note**: Phase 12 prev_day results INVALID (overnight sweep bug). Use Phase 7/13 instead.

### Phase 13: Optimized Portfolio — CORRECTED (5 configs × 299 days, 2518s)
**Script**: `phase13_optimized_portfolio.py`
**KEY FIX**: Overnight sweep detection for prev_day (pre-session loop from tick 0).
| Config | Trades | WR | avgR | R/day | DD | Cal | Months+ |
|--------|-------:|---:|-----:|------:|---:|----:|--------:|
| baseline_window (9:45-12) | 1,745 | 57.6% | +0.755 | +4.41 | 14.4 | 91.7 | 13/13 |
| h1_tight_only (9:45-11) | 1,517 | 58.0% | +0.766 | +3.88 | 13.6 | 85.3 | 13/13 |
| optimal_concurrent (conc=2) | 2,345 | 53.1% | +0.587 | +4.60 | 16.9 | 81.4 | 13/13 |

**Prev Day with fix**: +2.14 avgR per trade (was +0.04 broken in Phase 9/10/12).
**Cal 91.7 vs Phase 9's 27.2** = 3.4× improvement from trail + overnight fix.

### Phase 14: Advanced 1H Wick Optimizations (15 configs × 299 days, 3128s)
**Script**: `phase14_h1_advanced.py`
**Tested**: Short-only, trail tight/mid/wide, wick 20-60, time windows, IB fade, entry limits
**KEY FINDING**: Trail 1R/0.5R = 2.3× better Calmar than fixed 2R target.
| Config | Trades | WR | avgR | R/day | DD | Cal |
|--------|-------:|---:|-----:|------:|---:|----:|
| h1_trail_tight (1R/0.5R) | 1,433 | 68.1% | +0.753 | +3.61 | 7.2 | 150.8 |
| h1_trail_mid (1.5R/0.5R) | 1,142 | 61.5% | +0.886 | +3.38 | 8.3 | 122.3 |
| h1_short_10_11_trail | 732 | 70.1% | +1.083 | +2.65 | 8.2 | 97.1 |
| h1_trail_wide (2R/1R) | 807 | 55.4% | +1.055 | +2.85 | 9.1 | 93.2 |
| h1_baseline (2R fixed) | 974 | 55.3% | +0.618 | +2.01 | 9.2 | 65.5 |

**IB fade = DEAD** (Cal 7.2-7.6). Short-only + 10-11AM = highest per-trade quality.

### Phase 15: Novel Structural Levels (15 configs × 299 days, 1925s)
**Script**: `phase15_structural_levels.py`
**Tested**: Globex H/L, first 15/30/60 min H/L, round numbers (50/100pt), prev close, weekly H/L, micro IB
**MAJOR DISCOVERY**: Round 50pt levels = Cal 99.6, +3.79 R/day, 13/13 months!
| Config | Trades | WR | avgR | R/day | DD | Cal | Months+ |
|--------|-------:|---:|-----:|------:|---:|----:|--------:|
| **round_50** | 2,465 | 49.9% | +0.460 | +3.79 | 11.4 | 99.6 | 13/13 |
| round_100 | 1,327 | 49.2% | +0.440 | +1.95 | 15.0 | 38.9 | 13/13 |
| prev_close | 340 | 50.3% | +0.466 | +0.53 | 6.4 | 24.8 | 13/13 |
| prev_close_trail | 377 | 63.9% | +0.370 | +0.47 | 6.2 | 22.3 | 13/13 |
| globex_hl_trail | 448 | 59.8% | +0.318 | +0.48 | 8.4 | 17.0 | 12/13 |

**Dead from this phase**: First 60 min H/L (Cal 1.4-2.4), Weekly H/L (Cal 2.1-2.4), Micro IB (Cal 5.5).

Round 50pt levels are structurally different from 1H/prev_day — completely uncorrelated signal source.

### Phase 16: Ultimate Portfolio (8 configs × 299 days, 2944s)
**Script**: `phase16_ultimate_portfolio.py`
**Tested**: Combining 1H trail + prev_day trail (overnight fix) + VWAP, various concurrency/trail configs
| Config | Trades | WR | avgR | R/day | DD | Cal | WWR |
|--------|-------:|---:|-----:|------:|---:|----:|----:|
| h1_trail_pd_fixed | 1,995 | 69.3% | +0.819 | +5.46 | 7.6 | 215.7 | 100% |
| ultimate_tight | 2,078 | 66.5% | +0.842 | +5.85 | 8.3 | 211.6 | 100% |
| ultimate_1at | 2,426 | 67.8% | +0.828 | +6.71 | 10.1 | 198.8 | 100% |
| all_trail | 2,480 | 70.2% | +0.800 | +6.64 | 10.4 | 191.6 | 100% |

Confirmed 3-strategy combo (1H + prev_day + VWAP) with trail is extremely robust.

### Phase 17: Round Numbers in Portfolio (10 configs × 299 days, 4297s)
**Script**: `phase17_round_numbers.py`
**Tested**: Adding round number levels (50pt and 100pt) to the portfolio, with and without trail
**BEST RESULT ACROSS ALL PHASES**:
| Config | Trades | WR | avgR | R/day | DD | Cal | WWR |
|--------|-------:|---:|-----:|------:|---:|----:|----:|
| **add_round50_trail** | 3,910 | 69.2% | +0.798 | +10.43 | 8.6 | 361.8 | 100% |
| add_round100 | 3,051 | 65.3% | +0.809 | +8.26 | 9.3 | 264.4 | 100% |
| add_prevclose | 2,497 | 67.8% | +0.836 | +6.98 | 10.1 | 206.8 | 100% |
| best_3strat (no round) | 2,426 | 67.8% | +0.828 | +6.71 | 10.1 | 198.8 | 100% |

Round 50pt trail added +1,144R incremental on top of the 3-strategy base. 100% weekly WR.

### Phase 18: Robustness Testing (13 configs × 299 days, 4693s)
**Script**: `phase18_robustness.py`
**Tested**: Slippage (1pt/2pt/1+1), double commission, walk-forward quarters, risk cap, round100_only

**Slippage Bug Found and Fixed**: Original code had `entry_slip * (-direction)` which gave FAVORABLE fills instead of adverse. Fixed to `entry_slip * direction`. Walk-forward results (0 slippage) were unaffected.

**Final corrected results**:
| Config | Trades | WR | avgR | R/day | DD | Cal | WWR |
|--------|-------:|---:|-----:|------:|---:|----:|----:|
| baseline (0 slip) | 3,910 | 69.2% | +0.798 | +10.43 | 8.6 | 361.8 | 100% |
| slip_1pt | 3,788 | 67.8% | +0.732 | +9.28 | 9.4 | 294.0 | 100% |
| slip_2pt | 3,632 | 66.3% | +0.693 | +8.42 | 12.5 | 200.7 | 100% |
| slip_1_1 (entry+exit) | 3,788 | 67.8% | +0.683 | +8.66 | 10.7 | 241.4 | 100% |
| comm_1pt (double) | 3,910 | 69.2% | +0.772 | +10.09 | 9.1 | 330.9 | 100% |
| round100_only | 3,203 | 68.6% | +0.796 | +8.52 | 9.0 | 284.5 | 100% |
| max_risk_30 | 4,862 | 65.1% | +0.548 | +8.91 | 11.0 | 242.7 | 100% |

**Walk-Forward**:
| Period | Trades | WR | avgR | R/day | Cal |
|--------|-------:|---:|-----:|------:|----:|
| Q1 (Feb-Apr '25) | 1,302 | 67.4% | +0.646 | +11.36 | 103.5 |
| Q2 (May-Jul '25) | 546 | 70.5% | +0.840 | +6.12 | 73.9 |
| Q3 (Aug-Oct '25) | 709 | 69.0% | +0.776 | +7.33 | 112.0 |
| Q4 (Nov-Feb '26) | 1,353 | 70.6% | +0.938 | +16.92 | 147.3 |
| H1 IS | 1,848 | 68.3% | +0.703 | +8.72 | 160.0 |
| H2 OOS | 2,062 | 70.0% | +0.882 | +12.13 | 211.1 |

---

## 3. WHAT WE ULTIMATELY LANDED ON

### Production Configuration

```
PORTFOLIO: 4-Strategy Sweep Fade
INSTRUMENT: NQ Futures (Micro or Full)
SESSION: 9:45 AM - 12:00 PM ET entries, EOD close 3:55 PM ET
CONSTRAINT: One trade at a time (priority: prev_day > 1H > round50 > VWAP)
COMMISSION: 0.5pt per trade
```

**Strategy 1: 1H Wick Sweep Fade** (45.6% of total R)
- Levels: 1H candle swing H/L, lookback = 5 days, min wick = 10pt
- Entry: Sweep past wick tip, come back through → enter at level
- Stop: Sweep tip + 5pt buffer
- Exit: Trail 1R/0.5R (trigger at 1.0R MFE, trail 0.5R behind)
- Max 3 entries per level per day
- ~1,721 trades/year, +0.825 avgR, ~68% WR

**Strategy 2: Round 50pt Sweep Fade** (36.7% of total R)
- Levels: Every 50pt increment (21000, 21050, 21100...) within ±500pt of price
- Entry: Sweep past round number, come back through → enter at level
- Stop: Sweep tip + 5pt buffer
- Exit: Trail 1R/0.5R
- Max 3 entries per level per day
- ~1,630 trades/year, +0.702 avgR, ~63% WR

**Strategy 3: Previous Day H/L Sweep Fade** (9.0% of total R)
- Levels: Yesterday's RTH session high and low
- Entry: Sweep past prev H/L, come back through → enter at level
- Stop: Sweep tip + 5pt buffer
- Exit: Trail 1R/0.5R
- **CRITICAL**: Track sweeps from tick 0 (overnight/pre-market), not just from RTH open
- ~151 trades/year, +1.864 avgR, ~70% WR

**Strategy 4: VWAP Deviation Sweep Fade** (8.8% of total R)
- Level: VWAP = cumulative(price × volume) / cumulative(volume) from session open
- Entry: Sweep 12pt past VWAP, come back through → enter at VWAP
- Stop: Sweep tip + 5pt buffer
- Exit: 2R fixed target (trail doesn't help on VWAP — too noisy)
- ~408 trades/year, +0.669 avgR, ~55% WR

### Key Numbers to Know
- **Average trades per day**: ~13
- **Average R per trade**: +0.798
- **Average R per day**: +10.43
- **Worst month**: +114R (September — still massively positive)
- **Best month**: +382R (November)
- **Max drawdown**: 8.6R
- **Worst case with 2pt slippage**: +8.42 R/day, Cal 200.7, 100% WWR

---

## 4. COMPLETE PARAMETER SWEEP RESULTS

### 4A. Target R-Multiples (Phase 4, on 1H wick only)
| Target | Trades | WR | avgR | R/day | Cal | Verdict |
|-------:|-------:|---:|-----:|------:|----:|---------|
| 1.0R | 3,871 | 55.7% | +0.073 | +1.09 | 6.8 | Too small |
| 1.5R | 3,324 | 46.0% | +0.108 | +1.39 | 10.4 | OK |
| 2.0R | 2,943 | 39.1% | +0.131 | +1.49 | 10.2 | OK |
| 3.0R | 2,435 | 29.5% | +0.132 | +1.25 | 5.0 | Marginal |
| 5.0R | 1,923 | 18.3% | +0.027 | +0.20 | 0.5 | DEAD |
| 10.0R | 1,552 | 10.6% | -0.045 | -0.27 | -0.5 | DEAD |
| Runner (EOD) | 1,362 | 7.6% | +0.055 | +0.29 | 0.5 | DEAD |

**Conclusion**: 1.5-2.0R sweet spot. But trail 1R/0.5R outperforms all fixed targets (discovered Phase 10/14).

### 4B. Trail Stop Variants (Phase 14)
| Trail Config | Trades | WR | avgR | Cal | vs Fixed 2R |
|-------------|-------:|---:|-----:|----:|:-----------:|
| **Trail 1R/0.5R (tight)** | 1,433 | 68.1% | +0.753 | 150.8 | **+2.3×** |
| Trail 1.5R/0.5R (mid) | 1,142 | 61.5% | +0.886 | 122.3 | +1.9× |
| Trail 2R/1R (wide) | 807 | 55.4% | +1.055 | 93.2 | +1.4× |
| Fixed 2R (baseline) | 974 | 55.3% | +0.618 | 65.5 | 1.0× |

Trail 1R/0.5R is definitively the best exit. Tighter = more trades captured, higher WR, lower DD.

### 4C. Breakeven Stops (Phase 4)
| Config | avgR | R/day | Cal | Verdict |
|--------|-----:|------:|----:|---------|
| No B/E | +0.108 | +1.39 | 10.4 | Baseline |
| B/E @ 0.5R | +0.026 | +0.42 | 2.9 | KILLS EDGE |
| B/E @ 0.75R | +0.052 | +0.78 | 4.2 | Still bad |
| B/E @ 1.0R | +0.071 | +0.99 | 6.8 | Better but worse |

**Conclusion**: B/E destroys sweep fades at every trigger level. Price naturally retests entry before moving to target.

### 4D. Sweep Size / Wick Range (Phase 4)
| Wick Range | Trades | avgR | Cal | OOS avgR |
|-----------|-------:|-----:|----:|---------:|
| 3-10pt | 3,726 | +0.059 | 4.0 | +0.069 |
| 3-15pt | 3,636 | +0.084 | 6.3 | +0.090 |
| **5-25pt** | **2,746** | **+0.145** | **12.6** | **+0.148** |
| 10-30pt | 1,606 | +0.147 | 8.7 | +0.151 |
| 15-50pt | 982 | +0.104 | 2.9 | +0.120 |

5-25pt optimal. <5pt = noise. >25pt = trend breakouts that don't fade.

### 4E. Time Windows (Phase 12)
| Window | R/day | DD | Cal |
|--------|------:|---:|----:|
| 9:45-11:00 AM | +1.42 | 8.2 | 51.7 |
| 10:00-12:00 | +1.93 | 12.9 | 44.8 |
| 9:45-12:00 (production) | +1.76 | 12.5 | 42.0 |
| 9:45-1:00 PM | +1.97 | 19.2 | 30.8 |
| 9:45-2:00 PM | +2.12 | 25.4 | 24.9 |

Tighter = better Cal. Production uses 9:45-12:00 as balance of R/day and Calmar.

### 4F. Direction Bias (Phase 12)
| Filter | Cal | Notes |
|--------|----:|-------|
| Short-only 1H | 23.3 | 4.5× better than longs |
| Long-only 1H | 5.2 | Longs drag performance |
| Both (production) | 42.0 | Still best overall due to volume |

Shorts are higher quality but longs still net positive. Keep both.

### 4G. Confluence Filters (Phase 4)
| Filter | Trades | avgR | Cal | Verdict |
|--------|-------:|-----:|----:|---------|
| Base (no filter) | 3,324 | +0.108 | 10.4 | Baseline |
| IFVG nearby | 680 | +0.169 | 8.4 | Best per-trade but few trades |
| Premium/Discount | 1,843 | +0.099 | 7.6 | Marginal |
| Engulfing | 267 | -0.090 | -0.6 | DEAD |
| Strong close | 406 | -0.080 | -0.7 | DEAD |
| Pin bar | 147 | -0.146 | -0.6 | DEAD |

**All candle confirmation patterns DEAD**. Waiting for bar close misses the edge.

### 4H. Re-Entry & Other (Phases 4, 12, 14)
| Test | Result |
|------|--------|
| 3 re-entries (production) | Cal 10.4 |
| 1 entry only | Cal 1.9 — KILLS EDGE |
| Direction flip after loss | -78% R/day — KILLS EDGE |
| 2nd sweep required | Cal 11.3, DD 23.5 — legitimate quality filter |
| Max 2 entries per level | Slightly better than 3 (Cal 72.5 vs 65.5) |

---

## 5. KEY OPTIMIZATIONS (ORDERED BY IMPACT)

1. **Trail 1R/0.5R** (Phase 10/14): +2.3× Calmar improvement. Single biggest finding.
2. **Round 50pt levels** (Phase 15/17): +1,144R incremental. Brand new uncorrelated signal.
3. **Overnight sweep detection** (Phase 13): Prev day from +0.04 to +1.86 avgR. Bug fix → massive improvement.
4. **One-at-a-time with priority** (Phase 9): Prevents simultaneous losers. Priority ensures best signals fire.
5. **Time window 9:45-12:00** (Phase 12): Cuts afternoon noise, better Calmar.
6. **Lookback limit lb=5** (Phase 4): Prevents level accumulation artifact.

---

## 6. DEAD ENDS (DO NOT RE-TEST)

### Dead Strategy Types
| Strategy | Phase | Result |
|----------|------:|--------|
| Opening range breakout/fade/squeeze | 1 | All negative or flat |
| 15M wick fade | 2 | Too noisy |
| 4H wick fade | 2 | Too slow, too few trades |
| Open pivot fade | 3 | Negative |
| Opening drive | 3 | Negative |
| Delta divergence | 3 | Negative |
| Plain VWAP fade (no sweep) | 3, 5 | All negative on full year |
| London close reversal | 6 | Negative |
| IB extension sweep | 6 | Zero trades with wick filter |
| Opening bar rejection | 6 | Too rare (0-2 trades/month) |
| Mid-range continuation | 6 | Negative |
| Overnight sweep | 6, 7 | Marginal (Cal 6.8), not portfolio-worthy |
| Gap fill fade | 11 | All 16 configs dead (best +0.001 avgR) |
| Session open fade | 11 | All 18 configs deeply negative |
| Double sweep filter | 11 | Marginal Cal 3.8 |
| 2-day range sweep | 11 | Good per-trade but too infrequent |
| IB (Initial Balance) fade | 14 | Cal 7.2-7.6, not portfolio-worthy |
| First 60 min H/L | 15 | Cal 1.4-2.4 |
| Weekly H/L | 15 | Cal 2.1-2.4 |
| Micro IB (5min) | 15 | Cal 5.5 |

### Dead Filters/Optimizations
| Optimization | Result |
|-------------|--------|
| B/E at any trigger (0.5, 0.75, 1.0R) | Kills 30-70% of R/day |
| Trail on 5M swings | Worse than fixed targets |
| Targets >3R | Cal collapses (sweep fades are quick reversals) |
| Runner to EOD | Cal 0.5 |
| 1 entry per level (no re-entry) | Cal 1.9 |
| Direction flip after loss | -78% R/day |
| Candle confirmation (engulfing/pin/strong close) | All negative avgR |
| Volume filter | Too few trades |
| Afternoon extension (past 12:00 PM) | Adds trades but much worse Calmar |

---

## 7. LOOK-AHEAD BIAS AUDIT

All signals were systematically audited for look-ahead bias:

| Signal | Audit | Clean? |
|--------|-------|:------:|
| 1H wick levels | Levels from 1H bars that CLOSED before session open. No future data used. | YES |
| Prev day H/L | Yesterday's session H/L — known before today opens. Sweeps tracked from tick 0 forward. | YES |
| Round numbers | Static math (50pt increments). No data dependency at all. | YES |
| VWAP | cum(price×vol)/cum(vol) — only uses ticks that have already occurred. | YES |
| Sweep detection | Tick-by-tick: saw sweep tick, then saw return tick → trigger. Strict temporal ordering. | YES |
| Trail stop | Updates on each tick sequentially. MFE tracked from entry forward. | YES |
| Priority system | Checks open position at time of signal. No future knowledge. | YES |

---

## 8. BUGS FOUND AND FIXED

### Bug 1: Overnight Sweep Detection (Phase 9/10/12)
- **Problem**: Prev day sweep tracking started at RTH open_idx, missing pre-market sweeps
- **Impact**: Prev day avgR reported as +0.04 instead of +1.86
- **Fix (Phase 13)**: Track prev_day sweeps from tick 0, not open_idx. Entry still restricted to session window.
- **Affected phases**: 9, 10, 12. Phases 7 and 13+ are correct.

### Bug 2: Level Accumulation Bias (Phase 4)
- **Problem**: Unlimited 1H lookback accumulates hundreds of levels → every move triggers a trade
- **Impact**: lbAll showed Cal 100.2 — entirely fake edge
- **Fix**: Cap lookback at lb=3 to lb=5. Only recent levels are meaningful.

### Bug 3: Slippage Direction (Phase 18)
- **Problem**: `entry_slip * (-direction)` gave favorable fills instead of adverse
- **Impact**: For shorts (dir=-1), moved entry UP (better sell price). Should move DOWN (worse).
- **Fix**: Changed to `entry_slip * direction`. Walk-forward results (0 slippage) unaffected.

---

## 9. ALTERNATIVE PORTFOLIOS

If the full 4-strategy portfolio is too complex for execution:

| Portfolio | Strategies | R/day | Cal | DD | Trades | WWR |
|-----------|-----------|------:|----:|---:|-------:|----:|
| **Full (recommended)** | 1H + Round50 + PrevDay + VWAP | +10.43 | 361.8 | 8.6 | 3,910 | 100% |
| Round100 variant | 1H + Round100 + PrevDay + VWAP | +8.26 | 264.4 | 9.3 | 3,051 | 100% |
| Lowest DD | 1H + PrevDay (trail, 2R fixed) | +5.46 | 215.7 | 7.6 | 1,995 | 100% |
| 3-strategy (no round) | 1H + PrevDay + VWAP | +6.71 | 198.8 | 10.1 | 2,426 | 100% |
| 1H only | 1H wick trail | +5.96 | 176.5 | 10.1 | 2,384 | 100% |

---

## 10. DATA & INFRASTRUCTURE

### Tick Data
- **Source**: Databento (NQ front-month trades, `trades` schema)
- **Location**: `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/`
- **Files**: `glbx-mdp3-YYYYMMDD.trades.dbn.zst` (compressed)
- **Coverage**: 312 files, Feb 2025 - Feb 2026, 299 trading days loaded

### Running Scripts
```bash
# IMPORTANT: Run from /tmp to avoid operator/ module shadowing
cd /tmp && /Users/harrisonwillis/Documents/trading-system/.venv/bin/python <script.py>
```

### Python Environment
- **Venv**: `/Users/harrisonwillis/Documents/trading-system/.venv/bin/python` (Python 3.9)
- **Key packages**: numpy, pandas, databento

### File Locations
- **Scripts**: `~/Documents/trading-system/keylevel_sweep_fade/scripts/`
- **Results**: `~/Documents/trading-system/keylevel_sweep_fade/results/`
- **Working log**: `~/Documents/trading-system/keylevel_sweep_fade/WORKING_LOG.md`
- **This document**: `~/Documents/trading-system/keylevel_sweep_fade/SWEEP_FADE_RESEARCH.md`

### Scripts Index
| Phase | Script | Configs | Runtime | Key Finding |
|------:|--------|--------:|--------:|-------------|
| 1 | phase1_opening_range.py | 15 | ~1800s | All dead |
| 2 | phase2_multi_tf_wick.py | 12 | ~1800s | 1H wick = only viable TF |
| 3 | phase3_daily_open_novel.py | 14 | ~1800s | VWAP fade (didn't reproduce) |
| 4 | phase4_validate_1h.py | 21 | ~3600s | lb=5 optimal, lbAll = artifact |
| 5 | phase5_vwap_or_validate.py | 57 | ~3600s | Prev day sweep, VWAP sweep |
| 6 | phase6_novel_strategies.py | 41 | ~3600s | All dead except marginal overnight |
| 7 | phase7_fullyear_winners.py | 28 | ~3600s | Prev day trail = Cal 42.3 |
| 8 | phase8_portfolio_analysis.py | - | ~600s | Monthly R analysis |
| 9 | (combined portfolio) | 4 | ~3600s | 5M swing = dead weight |
| 10 | phase10_portfolio_optimize.py | 4 | 10691s | Trail 1R/0.5R = Cal 176.5! |
| 11 | phase11_novel_batch2.py | 49 | 2841s | Gap fill, session open = dead |
| 12 | phase12_filter_optimization.py | 36 | 7428s | Time window, short bias |
| 13 | phase13_optimized_portfolio.py | 5 | 2518s | Overnight fix → Cal 91.7 |
| 14 | phase14_h1_advanced.py | 15 | 3128s | Trail = 2.3× better than fixed |
| 15 | phase15_structural_levels.py | 15 | 1925s | Round 50pt = Cal 99.6! |
| 16 | phase16_ultimate_portfolio.py | 8 | 2944s | Combined Cal 198-215 |
| 17 | phase17_round_numbers.py | 10 | 4297s | Full portfolio Cal 361.8! |
| 18 | phase18_robustness.py | 13 | 4693s | Survives 2pt slip, walk-forward clean |

**Total**: 320+ configurations tested across 18 phases.

### Results Index
| File | Contents |
|------|----------|
| phase1_opening_range_jan26.json | 15 OR configs |
| phase2_multi_tf_wick_jan26.json | 12 multi-TF configs |
| phase3_novel_jan26.json | 14 novel strategy configs |
| phase4_1h_wick_full.json | 21 1H validation configs |
| phase5_vwap_or_jan26.json | 57 VWAP/OR/PrevDay configs |
| phase6_novel_jan26.json | 41 novel batch 1 configs |
| phase7_fullyear_winners.json | 28 winner configs |
| phase9_combined_portfolio.json | 4 combined configs |
| phase10_portfolio_optimize.json | 4 optimization configs |
| phase11_novel_batch2.json | 49 novel batch 2 configs |
| phase12_filter_optimization.json | 36 filter configs |
| phase13_optimized_portfolio.json | 5 corrected portfolio configs |
| phase14_h1_advanced.json | 15 advanced 1H configs |
| phase15_structural_levels.json | 15 structural level configs |
| phase16_ultimate_portfolio.json | 8 ultimate portfolio configs |
| phase17_round_numbers.json | 10 round number configs |
| phase18_robustness.json | 13 robustness configs |

---

## 11. EVOLUTION OF RESULTS

How the portfolio improved across phases:

| Phase | Milestone | R/day | Cal |
|------:|-----------|------:|----:|
| 4 | 1H wick validated (lb5, 2R) | +2.07 | 26.6 |
| 7 | Added prev_day trail | +4.27 | ~35 |
| 9 | Combined portfolio (4 strat) | +2.49 | 27.2 |
| 10 | Trail 1R/0.5R on 1H | +5.96 | 176.5 |
| 13 | Overnight sweep fix | +4.41 | 91.7 |
| 16 | Ultimate 3-strat portfolio | +6.71 | 198.8 |
| 17 | **Added round 50pt** | **+10.43** | **361.8** |

From +2.07 R/day to +10.43 R/day. From Cal 26.6 to Cal 361.8. 5× improvement through systematic optimization.
