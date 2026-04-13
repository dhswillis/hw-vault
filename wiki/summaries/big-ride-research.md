---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sweep/BIG_RIDE_RESEARCH.md
related:
  - wiki/summaries/sweep-cluster.md
  - wiki/summaries/daily-range-research.md
  - wiki/maps/strategies-moc.md
tags: [summary, big-ride, hod-lod, trend-capture]
---

# Big Ride Research — Summary

> Catching HOD/LOD early and riding all day. U-shaped timing: 30% of daily extremes made in first 30 min of globex, dead zone 19:00–07:00 CST. Five entry techniques tested with wide trailing stops.

## Key findings

Analyzed when the daily high/low is established across 297 full sessions. U-shaped / bathtub distribution: extremes overwhelmingly at session open or close. Three clusters: globex open (17:00–18:00, ~20%), pre-RTH/RTH open (07:30–09:00, ~15%), RTH close (13:30–15:30, ~20%). 50% of both highs and lows settled by 07:30–08:00 CST.

Five entry techniques tested: open crossback, overnight fade, IB fade, VWAP cross, first 5M wick. All with 80pt stops and 30–75pt trailing.

## Status / caveats

Research exploratory — check whether any of these made it into the final portfolio. The wide trailing stop approach is [[wiki/concepts/bar-sim-trailing-bug|bar-sim vulnerable]] if tested on OHLC bars rather than ticks.
