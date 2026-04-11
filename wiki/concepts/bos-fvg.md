---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
  - raw-sources/STRATEGY_MINING_REPORT.md
  - ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md
  - ~/Documents/trading-system/results/BOS_FVG_10PT_AUDIT.md
related:
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/summaries/strategy-mining-report-312d.md
tags: [trading, signal, nq, dead-strategy, suspect-results]
---

# BOS_FVG (Break of Structure + Fair Value Gap)

> **STATUS: DEAD.** BOS_FVG is not a validated signal and does not have tradeable edge. Every prior "validated" number was a bar-level simulation artifact. Harrison has confirmed this multiple times in `~/Documents/trading-system/CLAUDE.md`. On 2026-04-10, a fresh tick-level replay reproduced the failure on the same 60-day sample. See [[bos-fvg-failure-consolidated]].

## Why this is tagged suspect

Every performance number that appeared in earlier versions of this page — 63.5% WR, +15,368 total R, +1.354 R/T, +13.37 R/day, Calmar 151/237/263.6 — came from simulators that trailed stops on 1-minute bar OHLC. A paired bar-vs-tick replay on 2026-04-10 showed:

| Simulation | n | avgR | WR | R/day |
|---|---|---|---|---|
| Bar-level (`clean_backtest.py` logic) | 875 | **+0.3590** | 41.0% | +5.24 |
| **Tick-level (ground truth)** | **842** | **+0.0010** | **66.9%** | **+0.01** |

The bar simulator inflates BOS_FVG trailing results by **+0.29 R per trade**, and without the inflation the signal is indistinguishable from zero. This is consistent with the tick-level 10+pt static audit (`BOS_FVG_10PT_AUDIT.md`), which reported a +0.015 avgR baseline and no filter surviving multiple-comparison correction. See [[bar-sim-trailing-bug]] for the mechanism.

## What it is (signal definition — still accurate)

A compound signal combining:
- **Break of structure (BOS)** — the closing price of a bar breaks the most recent confirmed swing high/low in the trading direction by at least 2.0 points of displacement, and the previous bar's close did not
- **Fair value gap (FVG)** — a three-bar imbalance where `bar[i].low > bar[i-2].high` (bullish) or `bar[i-2].low > bar[i].high` (bearish), forming within 15 bars of the BOS in the same direction
- **Entry** — limit at the far edge of the FVG (gap_high for longs, gap_low for shorts) with 0.25pt slippage
- **Stop** — opposite edge of the FVG
- **Exit** — pure trailing stop, 0.5R trigger, 0.1R buffer

The signal generator itself is fine. It fires where and when you'd expect. The problem is that the exit mechanism (the trailing stop) cannot be accurately simulated on bar data, and when you replay it against ticks the edge collapses.

## What the invalidation affects

- **V1-era 97.9% / 98.8% / 90.6% WR tiers** from the 312-day mining report. Already flagged suspect for [[v10i-look-ahead-bug|V10i alignment look-ahead]]. Now also invalidated for bar-sim path reconstruction.
- **V2-corrected "honest" numbers**: 63.5% WR, +1.354 R/T, +15,368 total R from [[comprehensive-mining-report-v2]]. The V2 rerun fixed alignment look-ahead but still ran on 1-minute bar OHLC. It did not identify the path-reconstruction problem.
- **`clean_backtest.py` / `clean_backtest.py` output CSVs** in `~/Documents/trading-system/results/` (`bos_disp_*.csv`, `fvg_fill*_bos*.csv`, `full_year_all_trades.csv`). All bar-level trailing sims. Suspect until re-run with tick replay.
- **V10y/V10z 15S trailing results** (+1.141 avgR, Calmar 263.6) from the auto-memory. 15-second bars compress less path than 1-minute bars, but the same structural bug applies. CLAUDE.md has explicitly rejected these numbers since March 2026.
- **The "mega portfolio" composites** that include BOS_FVG as a leg — the composite R/day figures inherit the BOS_FVG inflation proportional to the leg's weight.

## What is NOT affected

- The **NinjaTrader engine-native implementations** (`SweepBreakv17.cs`, v24 IFVG MTF cascade). These run on the platform's tick-level execution, not on bar OHLC. If the v24 cascade demonstrates edge on forward sim, that's a real signal.
- **Fixed-R-target backtests** (static T3R, T5R, etc.) on bar data. Fixed targets are robust to intra-bar path ordering. The 10+pt static T3R audit's finding of +0.015 avgR is a tick-level result anyway, and it's the honest number for that variant.
- The **signal definition itself**. It is still a sensible way to identify a particular market structure. It is the trade-management layer that fails.

## Correlation and portfolio implications

If and when BOS_FVG is re-tested at tick resolution, its correlation with sweep-family signals and IFVG should be re-measured too. The pre-audit claim that BOS_FVG correlates near-zero with those signals remains plausible — but "near-zero correlation with something that also has no edge" is not portfolio value. Correlation arguments only help when every leg is individually positive, and in the current picture the only leg that has survived any tick-level scrutiny is the NinjaTrader engine-native live sim.

## See also

- [[bos-fvg-failure-consolidated]] — full failure report with paired bar-vs-tick comparison and reproducer scripts
- [[bar-sim-trailing-bug]] — structural explanation of why trailing stops can't be simulated on bar OHLC
- [[v10i-look-ahead-bug]] — the earlier contamination source that the V2 rerun partially fixed
- [[be-trail-mechanism]] — the exit mechanism whose apparent edge was the illusion
- [[mtf-alignment]] — the filter the V1-era tiers relied on before the look-ahead was found
- `~/Documents/trading-system/CLAUDE.md` — "Critical Corrections" section, the canonical ground truth on what is and isn't validated
