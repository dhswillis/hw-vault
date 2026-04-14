---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sf-portfolio/FINAL_PORTFOLIO_SPEC.md
related:
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/summaries/sf-portfolio-state.md
  - wiki/maps/strategies-moc.md
tags: [summary, sf-portfolio, final-spec, tick-confirmed]
---

# NQ+ES Final Portfolio Spec — Summary

> +3.34 R/Day, Calmar 53.1, Sharpe 6.57, 100% winning months over 260 days. 10 NQ legs + 3 ES legs + 1 runner. All look-ahead bugs fixed.

## Key findings

Final clean specification for the NQ+ES SF portfolio: 13 signal legs total (NQ: sweep-fail, VWAP, AMD, late fade, London fade; ES: VWAP multi-touch, late fade, AMD; Runner: 1H London sweep-fail with BE5). Max drawdown 16.4R, 87% winning weeks, 68% daily win rate. 8 look-ahead bugs found and fixed during development — this is the "zero look-ahead" verified version.

Runner signal (1H London PDH Sweep-Fail) is the highest-value single leg with strict conditions: London session only, prior day high sweep direction, pre-market range ≥100 pts, 5pt stop from entry.

## Status / caveats

This is the spec that [[wiki/summaries/sf-portfolio-cluster|SF Portfolio Cluster]] validates at tick level. The +3.34 R/Day is the tick-confirmed number. The runner leg is fat-tailed — expect high variance on short evaluation windows.

## Source documents

- [[raw-sources/trading/sf-portfolio/FINAL_PORTFOLIO_SPEC]]
- [[raw-sources/trading/sweep/TEMPO_PORTFOLIO_BUILD_SPEC]]
