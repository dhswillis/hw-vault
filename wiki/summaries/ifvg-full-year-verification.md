---
created: 2026-04-14
updated: 2026-04-14
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/ifvg-backtest/results/IFVG_Full_Year_Verification_Memo.docx
related:
  - wiki/concepts/ifvg.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/ifvg-optimization-report.md
  - wiki/concepts/v10i-look-ahead-bug.md
tags: [summary, trading, ifvg, tick-level, audit]
---

# IFVG Full Year Verification Memo — Summary

244 trading days of NQ tick-level backtest results for the IFVG Multi-Timeframe Cascade strategy (March 2025 to February 2026).

## Strategy definition

HTF = 15M, LTF = 15S. Identifies liquidity sweeps of structural levels on 15-minute charts, enters on 15-second FVG reactions. Death Candle = 6pts, Min Displacement = 3pts, Sweep Min = 1.5pts, Min Reaction Quality = 3. Soft stop = bar-close-only exit (average overshoot ~1.6x nominal risk). Slippage: 0.5pts/side. Commission: 0.5pts RT.

## Code audit findings

Line-by-line audit of market_vs_limit_compare.py (916 lines). Two look-ahead issues found:

**Bug 1 (Fixed, Moderate):** Swing level detection for 1H and 15M bars used a guard that checked if bar i+2 had STARTED but not if it was COMPLETE. Fixed to require bar i+2 fully closed.

**Bug 2 (Not Fixed, Negligible):** 1M bar OHLC at boundary includes full minute's final values rather than partial. Impact at most 1 minute for range levels already complete by RTH sweep time.

## R-Target findings

R1.5 and R2.0 MARKET are net losers or breakeven. R3.0+ is where positive expectancy emerges for market orders. LIMIT_MID profitable at every R-target from R1.5+. R4.0 LIMIT_MID is the best risk-adjusted target (+1.75 R/day, lowest MaxDD 119R).

## Key takeaway

Shorts outperform longs in this uptrending year — sweeps of structural highs that fail are reliable short entries. Unfiltered is a disaster (-$593/day); filters are essential.

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/ifvg-backtest/results/IFVG_Full_Year_Verification_Memo]]
