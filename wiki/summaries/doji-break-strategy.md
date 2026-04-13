---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sweep/DOJI_BREAK_STRATEGY.md
related:
  - wiki/summaries/sweep-cluster.md
  - wiki/maps/strategies-moc.md
tags: [summary, doji-break, bracket-orders]
---

# Doji Break Strategy — Summary

> Bracket stop orders on both sides of a doji candle. Edge is NOT in entry direction — it's entirely in B/E management. Without B/E, dead (~43% WR, net negative). With B/E, massively positive. Asia session outperforms US.

## Key findings

Validated on 312 days of Databento NQ tick data. Doji definition: body/range ≤ 0.25, minimum range 5pts, max risk <50pts. Buy stop above doji high + 1pt, sell stop below low - 1pt, whichever fills first = direction. Break-even management is THE edge: after 1 bar, move stop to entry + 2pt offset. Average risk 17.3pts (median 14.5pts).

Asia session mining complete — Asia outperforms US session.

## Status / caveats

The B/E management dependency is a yellow flag — verify this isn't the same phantom B/E mechanism that killed BOS_FVG results. The difference is that doji break uses a simple 1-bar B/E shift (not trailing), which may be more realistic on tick data. Cross-check the simulation method.
