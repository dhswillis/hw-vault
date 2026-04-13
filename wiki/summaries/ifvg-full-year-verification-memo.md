---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/strategies/tempo/results/IFVG_Full_Year_Verification_Memo.docx
related:
  - wiki/concepts/ifvg.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/ifvg-optimization-report-v2.md
  - wiki/maps/tempo-moc.md
tags: [summary, ifvg, verification, tick-level]
---

# IFVG Full-Year Verification Memo — Summary

> 244-day tick-level IFVG MTF Cascade backtest, code-audited and look-ahead verified. Two bugs found and fixed.

## Key findings

Full-year backtest of the [[wiki/concepts/ifvg|IFVG]] Multi-Timeframe Cascade on NQ tick data (Mar 2025–Feb 2026, 244 trading days). The strategy identifies liquidity sweeps on 15-minute charts and enters on 15-second FVG reactions with tight risk. Slippage: 0.5pts/side, commission: 0.5pts RT.

A line-by-line code audit of `market_vs_limit_compare.py` (916 lines) found two look-ahead issues. Bug 1 (fixed, moderate impact): swing level detection for 1H and 15M bars used incomplete bar data — fixed by requiring bar i+2 to be fully closed. Bug 2 (not fixed, negligible): 1-minute bar OHLC at boundaries includes the full minute rather than partial values, but this only affects range levels that are already complete by RTH.

Soft stop uses bar-close-only exit with average overshoot of ~1.6x nominal risk — this is the biggest leak addressed in the [[wiki/summaries/ifvg-optimization-report-v2|V2 optimization report]].

## Status / caveats

The cleanest IFVG verification document in the vault. Code audit is thorough. Read alongside the V2 optimization report which discovered the hard stop fix that nearly triples $/day.
