---
created: 2026-04-14
updated: 2026-04-14
type: concept
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE.md
related:
  - wiki/summaries/trading-system-start-here.md
  - wiki/summaries/doji-break-strategy.md
  - wiki/entities/tempo-trading-system.md
tags: [trading, signal, nq, doji-break, asia]
---

# Doji Break

A bracket-order strategy that places stop orders on both sides of doji candles. The edge comes entirely from break-even stop management — without B/E the strategy is completely dead.

## The core bet

When a doji candle forms (body <= 25% of range, range >= 5pts), place buy stop above high + 1pt and sell stop below low - 1pt. Whichever fills = entry, other side + 3pt = stop. B/E after 1 bar, TP at bar 3 close.

## Key finding: Asia session dominates

| Session | WR | R/Day | Calmar |
|---|---|---|---|
| Asia (UTC 0-8) | 88.5% | +48.0 | 293.8 |
| US (UTC 14:30-21) | 81.9% | +107.1 | 96.4 |

Best Asia config: 89% WR, ~59 R/day, Calmar 589. Walk-forward: H1 Calmar 162 to H2 Calmar 208 (OOS BETTER than IS).

## Why Asia works (Leppyrd framework)

Asia/London creates the "manipulation wick" — the daily candle's wick = the doji. The doji breakout IS the London reversal that discretionary traders trade. B/E management mechanically captures what Leppyrd trades with HTF context.

## Critical caveat

B/E IS the edge. Without B/E the strategy is completely dead (~43% WR). The B/E converts small winners into zero-cost exits while letting runners hit target. This is the opposite of the project-wide finding that B/E kills edge on [[be-trail-mechanism|Model 2 signals]] — Doji Break has fundamentally different mechanics (bracket orders vs directional fades).

## Cross-references

- [[trading-system-start-here]] — contains full Doji Break results
- [[doji-break-strategy]] — earlier summary of the strategy
