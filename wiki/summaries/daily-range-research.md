---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sweep/DAILY_RANGE_RESEARCH.md
related:
  - wiki/summaries/sweep-cluster.md
  - wiki/summaries/big-ride-research.md
  - wiki/maps/strategies-moc.md
tags: [summary, daily-range, pyramiding]
---

# Daily Range Capture Research — Summary

> 2-signal slippage-robust core: +11.3 pts/day, Calmar 7.5, 11/13 months positive. Trend gap-up LONG + early SHORT with pyramiding. Survives 2pt slippage at Calmar 1.7.

## Key findings

Portfolio of strategies capturing the daily range with scaled-in positions (up to 20 contracts). Slippage-robust core uses two signals: trend_L_gapUP_t30 (gap up >10, still up >10 at 60min → LONG, trail 30pt; 78% WR, Cal 4.0 at 1pt slip) and early_S_p15x5 (down >10 at 15min → SHORT, pyramid every 15pts max 5, trail 50pt; 16% WR but captures big down moves).

Full 4-signal portfolio (adding midnight_cross and early_short pyramiding variants) reaches +24.7 pts/day but is less robust to slippage. The 2-signal core trades ~every other day (144 trades in 299 days) with only 21% day overlap between signals.

## Status / caveats

Trailing stop exits make this [[wiki/concepts/bar-sim-trailing-bug|bar-sim vulnerable]] — but the research uses tick data (299 days Databento), so verify the sim iterates ticks not bars. The pyramiding approach amplifies both edge and risk.

## Source documents

- [[raw-sources/trading/sweep/DAILY_RANGE_RESEARCH]]
