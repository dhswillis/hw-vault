---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
  - raw-sources/STRATEGY_MINING_REPORT.md
related:
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/summaries/strategy-mining-report-312d.md
tags: [trading, signal, nq, core-signal]
---

# BOS_FVG (Break of Structure + Fair Value Gap)

BOS_FVG is the **core signal** of the [[tempo-trading-system|Tempo NQ system]]. Across mining iterations and cleanups, it has consistently been the strongest single signal by total R and walk-forward stability.

## What it is

A compound signal combining:
- **Break of structure (BOS)** — price breaking the most recent swing high/low in the trading direction, confirming a change in short-term directional bias
- **Fair value gap (FVG)** — a three-bar pattern where the middle bar's range leaves a gap between bar 1's extreme and bar 3's extreme in the direction of the break

Entry is taken on a retest of the FVG in the direction of the BOS.

## Performance (V2-corrected numbers)

From [[comprehensive-mining-report-v2|COMPREHENSIVE_MINING_REPORT V2]] (34,323 trades, 258 days, look-ahead fixed, slippage added, 1.5pt min risk):

- n = 11,348 trades
- **63.5% WR** with [[be-trail-mechanism|BE+Trail]] exit
- **+15,368R** total
- **R/T +1.354**
- Walk-forward: IS 63.6% → OOS 63.4% — stable
- Static targets negative through 1.5R; profitable at 2.0R+
- **Fat-tail distribution:** 29.5% of trades are 2R+ runners generating +17,461R. The other 70.5% combined for -2,094R.

Best signal in the dataset by total R and WF stability.

## Performance (V1-era — SUSPECT)

From [[strategy-mining-report-312d|STRATEGY_MINING_REPORT]] (312d, 8,358 trades):

- BOS_FVG + mtf_5m + body confirms: **97.9% WR**, n=140
- BOS_FVG + mtf_5m + body break + close near extreme: **98.8% WR**, n=82
- BOS_FVG + mtf_5m + failed auction: **90.6% WR**, n=127

These numbers are almost certainly inflated by the [[v10i-look-ahead-bug]]. Cross-check against V2 (63.5% WR).

## What makes the edge real

The edge is entirely in the [[be-trail-mechanism|BE+Trail exit]]: convert losers to near-zero breakevens, let winners run. Don't cap R targets — fat-tail runners drive all profit. At R=1.0 fixed target, BOS_FVG is -0.260 R/T (loser). With BE+Trail it's +1.354 R/T.

## Required filters

- **[[mtf-alignment|MTF alignment]]** — highest single filter for WR. Monotonic improvement from 40.8% (0 TFs aligned) to 81.0% (6 aligned). V1 research also showed MTF_5M gate alone flips the system from losing to winning.
- **Body confirmation on the entry candle** (V1-era claim; suspect magnitude, direction probably real).

## What DOESN'T help

- Static R targets below 2R — negative
- `strong_body > 70%` filter on entries — paradoxically hurts WR (36.8%)
- Capping R — kills fat-tail runners that drive edge

## Correlation with other signals

BOS_FVG correlates 0.324 with `fvg_exit_fvg_stop` (same FVG family). Near-zero correlation with sweep-family signals (sweep_wick, sweep_close_through, swing_failure) — so the two books are genuinely diversifying in a portfolio.

## Known contamination history

The V1-era research used alignment logic that (via [[v10i-look-ahead-bug|V10i]]) consulted unclosed higher-TF bars, pulling in their future close values. When the V2 rerun fixed the bug (completed-bar alignment only), the 90%+ WR configurations collapsed — but **BOS_FVG as a signal survived** and remains the top performer at honest 63.5% WR. The signal is real; only the WR magnitude changed.
