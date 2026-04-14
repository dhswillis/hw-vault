---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/FVG_ENTRY_TIMING_ANALYSIS.md
related:
  - wiki/concepts/ifvg.md
  - wiki/maps/tempo-moc.md
tags: [summary, fvg, entry-timing, passive-vs-momentum]
---

# FVG Entry Timing Analysis — Summary

> Two distinct entry regimes: Passive Fill (limit at FVG midpoint on retraces) vs Momentum Inversion (market order on confirmation through FVG). Current Model 1 enters on same bar — this timing issue explains performance gaps.

## Key findings

Bifurcated entry strategy for FVG signals. Passive Fill regime: price retracing into FVG → limit order at midpoint, lower urgency, higher fill certainty. Momentum Inversion regime: price blasting through FVG → market order on next bar confirmation. The current implementation doesn't distinguish these, entering on the detection bar regardless. Separating the regimes and applying the appropriate entry method is identified as a key performance improvement.

## Status / caveats

Feb 2026 analysis. Conceptually sound but needs tick-level validation of the two-regime model. The entry timing distinction has implications for the [[wiki/concepts/ifvg|IFVG]] implementation — check whether later IFVG research (V15+) incorporated this.

## Source documents

- [[raw-sources/trading/FVG_ENTRY_TIMING_ANALYSIS]]
