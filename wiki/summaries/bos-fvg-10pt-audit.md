---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/concepts/v10i-look-ahead-bug.md
tags: [trading, bos-fvg, audit, tick-replay, dead-strategy, statistical-rigor]
---

# BOS_FVG 10+pt Tick-Based Audit (2026-03-08)

**Source:** `raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md`
**Date:** 2026-03-08
**Data:** 312 days NQ tick data (Feb 2025 – Feb 2026)
**Config:** 3-min candles, 10+pt FVGs, far-edge entry, **static T3R target**, tick-by-tick simulation

The earlier (March) sibling of [[bos-fvg-failure-consolidated|the April 10 consolidated failure report]]. This one ran tick-level on the **static T3R target** variant and independently established that BOS_FVG has no edge — well before the April trailing-stop tick replay generalized the finding.

## TL;DR

**The 10+pt BOS_FVG strategy with static T3R targets has no tradeable edge.**

- **Baseline avgR: +0.015** (barely above zero)
- The primary "P/D" filter that produced the original headline was **100% look-ahead bias** — edge vanishes with clean data
- **No surviving filter** passes multiple-comparison correction (all p > 0.10)
- Rolling walk-forward: **5/10 folds positive (coin flip)**, Calmar < 1.0
- One outlier month (Feb 2026) props up all OOS averages

## P/D look-ahead bias — the original "edge"

The pre-audit "premium/discount" filter used the day's full high/low to define the 50% line — i.e. **future data** for any signal generated before the close.

| P/D method | Aligned avgR | Misaligned avgR | Verdict |
|---|---|---|---|
| **Broken (full-day H/L)** | **+0.241** | -0.107 | **LOOK-AHEAD — uses future data** |
| Rolling (up-to-signal-time) | +0.031 | +0.009 | DEAD — no separation |
| Prev-day range | +0.061 | -0.014 | Marginal — OOS collapses (+0.209 → -0.116) |

The entire +0.267 avgR "edge" was future data. With clean P/D, baseline reverts to **+0.015**.

**Alternative P/D methods tested — all DEAD:** VWAP-based (+0.009 vs +0.016), opening-range (+0.024 vs +0.010). Zero separation in every case.

## Stacked FVG filter — real but not significant

| ATR tercile | no_stacked avgR | stacked avgR | Gap |
|---|---|---|---|
| Low | -0.017 | -0.050 | +0.033 |
| Mid | +0.071 | -0.005 | +0.076 |
| High | +0.074 | -0.042 | +0.116 |

**Mechanism:** multiple FVGs in the same price zone = "fought over" level = BOS less likely to sustain.

**Statistical reality:** Bootstrap 95% CI spans zero `[-0.044, +0.157]`, permutation p = 0.118. Captures something structurally real, but the effect is too small relative to noise. **Not statistically significant.**

Important: stacked FVGs are **NOT a volatility proxy** — they occur in LOW vol (ATR 19.7 vs 29.7, r = -0.30).

## DST-corrected session breakdown

| Session | n | WR | avgR | Verdict |
|---|---|---|---|---|
| London | 625 | 27.4% | **+0.094** | Best session |
| NY Lunch | 278 | 27.7% | +0.108 | Good but thin |
| NY AM | 530 | 25.1% | +0.004 | Flat |
| Asia | 285 | 27.0% | -0.020 | Flat |
| **NY PM** | **307** | **22.1%** | **-0.182** | **Consistently negative** |

**Time-of-day:** UTC 18-20 (1-3pm CT) is toxic at -0.356 avgR, confirmed OOS. Best windows: UTC 06-08 and 12-14.

## FVG size buckets — 15-20pt is a dead zone

| Size | n | avgR | WF (IS → OOS) |
|---|---|---|---|
| 10-15pt | 1006 | +0.016 | baseline |
| **15-20pt** | **450** | **-0.165** | **negative both halves** |
| **20-30pt** | **319** | **+0.188** | **+0.249 → +0.115** |
| 30+pt | 250 | +0.094 | +0.055 → +0.137 |

20-30pt is the sweet spot, but permutation p = 0.027 — **fails Bonferroni correction** (threshold 0.002).

## Rolling walk-forward (3mo train / 1mo test, 10 folds)

| Filter | OOS avgR | Folds positive | Calmar |
|---|---|---|---|
| Baseline | +0.031 | 5/10 | 0.5 |
| No stacked | +0.090 | 5/10 | 0.8 |
| Prev-day P/D + no stack | +0.069 | 4/10 | 0.6 |

**Feb 2026 outlier:** baseline +0.983, no_stack +1.549. Without it, OOS near zero. The strategy's positive cumulative result is one month carrying everything else.

## Multiple comparison correction — nothing survives

25 filter combos tested. **Bonferroni threshold: p < 0.002.**

| Filter | avgR | 95% bootstrap CI | Perm p | Survives? |
|---|---|---|---|---|
| no_stacked | +0.052 | [-0.044, +0.157] | 0.118 | No |
| no_opposing | +0.041 | [-0.053, +0.133] | 0.142 | No |
| prev_day P/D + no_stack | +0.102 | [-0.076, +0.268] | 0.116 | No |
| pd_prev + no_stack + no_opp | +0.183 | — | 0.041 | No |
| fvg_20_30 | +0.188 | — | 0.027 | No |

**Every 95% CI spans zero.** Filter improvements are consistent with random subsetting.

## Trailing vs static — static wins (for the 10+pt config)

Full 312-day confirmation:
- **Static T3R:** +0.015 avgR, 26.0% WR
- **Trailing (0.5R/0.1R):** -0.004 avgR, 65.2% WR (high WR, tiny winners, net negative)

Trailing gets stopped out on noise within large FVGs. **Static is definitively better for the 10+pt config.**

This trailing line is interesting in light of [[bos-fvg-failure-consolidated|the April tick-replay audit]], which showed that the bar-level trailing sim was reporting much higher numbers for the broader BOS_FVG population. The 10+pt subset's tick-level trailing being net negative is consistent with that finding — trailing at tick resolution is honestly negative; the bar simulator was the one that fabricated the +0.895 / +1.141 inflated numbers.

## Best possible "clean" config

20+pt FVGs, no stacked (10pt threshold), exclude 1-3pm CT → ~300-400 trades, ~+0.15 avgR, Calmar ~2.

**This is NOT tradeable.** At 26% WR and ~+0.15 avgR, after commissions ($4.12 RT on NQ ≈ 0.2pts) and slippage (1-2 pts on far-edge limit fills) and execution variance, the expected edge is **zero or negative**.

## Conclusion

The 10+pt BOS_FVG with static targets does not have sufficient edge for live trading. The strategy's prior results were inflated by:

1. **P/D look-ahead bias** (+0.241 avgR — fake)
2. **Bar-based simulation inflation** (+0.209 avgR per trade vs tick — see [[bar-sim-trailing-bug]])
3. **Multiple-comparison artifacts** from filter sweeps

The BOS_FVG signal **may** still have value as a component in the broader Tempo system, but the 10+pt static-target configuration tested here is **dead**. The April 10 consolidated audit (this file's successor) generalized this finding to the trailing-stop case across the full BOS_FVG population: same conclusion, +0.001 avgR ground truth.

## Scripts

| Script | Location | Purpose |
|---|---|---|
| `fvg_10pt_audit.py` | `~/Documents/` | P/D look-ahead fix + opposing FVG fix |
| `fvg_stacked_audit.py` | `/tmp/` | Stacked FVG volatility-proxy analysis |
| `fvg_dst_wf_audit.py` | `/tmp/` | DST session fix + rolling walk-forward |
| `fvg_stats_audit.py` | `/tmp/` | Bootstrap CIs + permutation tests + fill rate |
| `fvg_alt_edges.py` | `/tmp/` | VWAP, opening range, time-of-day, size, trailing |
