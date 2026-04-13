---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/trading-system/TEMPO_Strategy_Analysis_2026-02-16.docx
related:
  - wiki/maps/tempo-moc.md
  - wiki/maps/strategies-moc.md
  - wiki/summaries/tempo-carmine-strategy-2026-02-16.md
  - wiki/concepts/invalidation-rules.md
tags: [summary, tempo, multivariate-analysis, suspect-results]
---

# Tempo Deep Multivariate Analysis — Summary

> 400 variants → 50 deep-tested → 9 stable strategies. Overlap session dominance confirmed. Multi-strategy portfolio projects 10.5R/day baseline.

## Key findings

Complete mining of all 400 Stage A variants and 50 Stage B deep-confirmed variants from the Feb 15, 2026 nightly batch run. Identifies edges across five dimensions: session timing, signal type, breakeven mode, R-target selection, and inside day filtering.

Session timing is the single strongest edge factor — the difference between overlap (best) and London (worst) spans 640+ score points. Every overlap variant was profitable. The top variant achieved Sharpe 2.571, 12.8% max drawdown, $63,682 net profit on $50K over 12 months. The full multi-strategy portfolio combining four complementary sub-strategies projects 10.5R/day with a path to 20R/day.

Key targets: achieve 60% daily WR with 3 high-confluence setups, scale toward 20R/day via multi-strategy portfolio, build all-weather system with all weeks profitable.

## Status / caveats

**Suspect.** Oct 2024–Sep 2025 backtest window predates the [[wiki/concepts/v10i-look-ahead-bug|V10i look-ahead bug]] and [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] discoveries. The headline numbers (Sharpe 2.571, 10.5R/day) are almost certainly inflated. Companion to the [[wiki/summaries/tempo-carmine-strategy-2026-02-16|Tempo+Carmine report]]. Do not use for live trading decisions without re-running on clean data per [[wiki/concepts/invalidation-rules|invalidation rules]].
