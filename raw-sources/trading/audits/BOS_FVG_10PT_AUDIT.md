# BOS_FVG 10+pt Tick-Based Backtest — Full Audit Report
**Date**: 2026-03-08
**Data**: 312 days NQ tick data (Feb 2025 – Feb 2026)
**Config**: 3min candles, 10+pt FVG, far edge entry, static T3R target, tick-by-tick simulation

---

## Executive Summary

**The 10+pt BOS_FVG strategy with static T3R targets has no tradeable edge.**

- Baseline avgR: **+0.015** (barely above zero)
- The primary filter (P/D) was **100% look-ahead bias** — edge vanishes with clean data
- No surviving filter passes multiple comparison correction (all p > 0.10)
- Rolling walk-forward: 5/10 folds positive (coin flip), Calmar < 1.0
- One outlier month (Feb 2026) props up all OOS averages

---

## 1. P/D Look-Ahead Bias — CONFIRMED DEAD

| P/D Method | Aligned avgR | Misaligned avgR | Verdict |
|---|---|---|---|
| **Broken (full day high/low)** | **+0.241** | -0.107 | **LOOK-AHEAD — uses future data** |
| Rolling (up to signal time) | +0.031 | +0.009 | DEAD — no separation |
| Prev day range | +0.061 | -0.014 | Marginal — OOS collapses (+0.209→-0.116) |

The entire +0.267 avgR "edge" was future data. With clean P/D, baseline reverts to +0.015.

### Alternative P/D Methods Tested — ALL DEAD
- **VWAP-based P/D**: Zero separation (aligned +0.009 vs misaligned +0.016)
- **Opening range P/D** (first 30min high/low): Zero separation (+0.024 vs +0.010)

---

## 2. Stacked FVG Filter — REAL but NOT SIGNIFICANT

**Not a volatility proxy** — stacked FVGs occur in LOW vol (ATR 19.7 vs 29.7, r = -0.30).

The filter adds value within every ATR tercile:
| ATR Tercile | no_stacked avgR | stacked avgR | Gap |
|---|---|---|---|
| Low | -0.017 | -0.050 | +0.033 |
| Mid | +0.071 | -0.005 | +0.076 |
| High | +0.074 | -0.042 | +0.116 |

**Mechanism**: Multiple FVGs in the same price zone = "fought over" level = BOS less likely to sustain.

**However**: Bootstrap 95% CI spans zero [-0.044, +0.157], permutation p = 0.118. Not statistically significant. The filter captures something real structurally, but the effect is too small relative to noise.

Optimal threshold: 10pt proximity (vs current 20pt).

---

## 3. DST-Corrected Session Breakdown

| Session | n | WR | avgR | Verdict |
|---|---|---|---|---|
| London | 625 | 27.4% | +0.094 | Best session |
| NY Lunch | 278 | 27.7% | +0.108 | Good but thin |
| NY AM | 530 | 25.1% | +0.004 | Flat |
| Asia | 285 | 27.0% | -0.020 | Flat |
| **NY PM** | **307** | **22.1%** | **-0.182** | **Consistently negative** |

**DST-specific**: Asia EST (winter) = +0.444 avgR (96 trades), Asia EDT (summer) = -0.255. Small sample.

**Time-of-day**: UTC 18-20 (1-3pm CT) is toxic at -0.356 avgR, confirmed OOS. UTC 06-08 and 12-14 are best windows.

---

## 4. FVG Size Buckets — 15-20pt Dead Zone

| Size | n | avgR | WF (IS→OOS) |
|---|---|---|---|
| 10-15pt | 1006 | +0.016 | baseline |
| **15-20pt** | **450** | **-0.165** | **negative both halves** |
| **20-30pt** | **319** | **+0.188** | **+0.249→+0.115** |
| 30+pt | 250 | +0.094 | +0.055→+0.137 |

20-30pt is the sweet spot, but permutation p = 0.027 — fails Bonferroni correction (threshold 0.002).

---

## 5. Rolling Walk-Forward (3mo train / 1mo test, 10 folds)

| Filter | OOS avgR | Folds Positive | Calmar |
|---|---|---|---|
| Baseline | +0.031 | 5/10 | 0.5 |
| No stacked | +0.090 | 5/10 | 0.8 |
| Prev day P/D + no stack | +0.069 | 4/10 | 0.6 |

Feb 2026 outlier: +0.983 baseline, +1.549 no_stack. Without it, OOS near zero.

---

## 6. Multiple Comparison Correction — NOTHING SURVIVES

25 filter combos tested. Bonferroni threshold: p < 0.002.

| Filter | avgR | 95% Bootstrap CI | Perm p | Survives? |
|---|---|---|---|---|
| no_stacked | +0.052 | [-0.044, +0.157] | 0.118 | No |
| no_opposing | +0.041 | [-0.053, +0.133] | 0.142 | No |
| prev_day P/D + no_stack | +0.102 | [-0.076, +0.268] | 0.116 | No |
| pd_prev+no_stack+no_opp | +0.183 | — | 0.041 | No |
| fvg_20_30 | +0.188 | — | 0.027 | No |

**Every 95% CI spans zero.** Filter improvements are consistent with random subsetting.

---

## 7. Fill Rate & Selection Bias

- **Overall fill rate**: 79.3% (2025 of 2554 signals)
- 10-15pt: 82.2%, 15-20pt: 76.4%, 20-30pt: 77.5%, 30+pt: 75.6%
- **Median time to fill**: 21.5 minutes (mean 70 min, heavy right tail)
- Long/short fill rates symmetric — no directional bias

---

## 8. Trailing vs Static — Static T3R Wins

Full 312-day confirmation:
- **Static T3R**: +0.015 avgR, 26.0% WR
- **Trailing (0.5R/0.1R)**: -0.004 avgR, 65.2% WR (high WR, tiny winners, net negative)

Trailing gets stopped out on noise within large FVGs. Static is definitively better for 10+pt.

---

## 9. Other Dead Ends

- **Direction**: Long +0.030, Short -0.000 — no reliable directional edge
- **FVG age**: Inconsistent walk-forward, no reliable pattern
- **Opposing FVGs** (fixed unfilled check): +0.041 avgR for no_opposing, not significant

---

## Conclusions

### What's real (but not tradeable):
1. **Stacked FVG filter** captures structural price behavior (not vol proxy), but effect is too small (p = 0.118)
2. **NY PM is toxic** — removing it helps, but doesn't create an edge
3. **20-30pt FVGs** are the best size bucket, but p = 0.027 (fails Bonferroni)
4. **UTC 18-20 is consistently negative** — avoidable drag

### Best possible clean config:
20+pt FVGs, no stacked (10pt threshold), exclude 1-3pm CT → ~300-400 trades, ~+0.15 avgR, Calmar ~2

**This is NOT tradeable.** At 26% WR and ~+0.15 avgR, after commissions ($4.12 RT on NQ, ~0.2pts), slippage (1-2pts on far-edge limit fills), and execution variance, the expected edge is zero or negative.

### Recommendation:
The 10+pt BOS_FVG with static targets does not have sufficient edge for live trading. The strategy's prior results were inflated by:
1. P/D look-ahead bias (+0.241 avgR — fake)
2. Bar-based simulation inflation (+0.209 avgR per trade vs tick)
3. Multiple comparison artifacts from filter sweeps

The BOS_FVG signal may still have value as a component in the broader Tempo system (V10z showed BOS_FVG with 15S trailing at +1.141 avgR), but the 10+pt static-target configuration tested here is dead.

---

## Scripts
| Script | Location | Purpose |
|---|---|---|
| fvg_10pt_audit.py | ~/Documents/ | P/D look-ahead fix + opposing FVG fix |
| fvg_stacked_audit.py | /tmp/ | Stacked FVG volatility proxy analysis |
| fvg_dst_wf_audit.py | /tmp/ | DST session fix + rolling walk-forward |
| fvg_stats_audit.py | /tmp/ | Bootstrap CIs + permutation tests + fill rate |
| fvg_alt_edges.py | /tmp/ | VWAP, opening range, time-of-day, size, trailing |
