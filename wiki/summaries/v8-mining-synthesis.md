---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sf-portfolio/V8_MINING_SYNTHESIS.md
related:
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/maps/audit-history-moc.md
tags: [summary, mining, v8, suspect-results]
---

# V8 Mining Synthesis — Summary

> 26 mining scripts, 1,690 BOS_FVG trades, walk-forward validated with "zero overfit detected." Headline: V8g>=4 at 61.7% WR, Calmar 121.2. Tier 2 weekly WR 90%.

## Key findings

The V8 synthesis is the most comprehensive BOS_FVG mining doc in the vault — 312 trading days, walk-forward split (174 train / 75 test), Monte Carlo 10K runs. The execution model defines three tiers: Tier 1 (max conviction, 0.4 signals/day, 82.4% WR), Tier 2 (high conviction, 1.2/day, 70% WR, 90% weekly WR), Tier 3 (standard, 2.9/day, 51.7% WR). Multi-timeframe confluence (both_pro alignment) is identified as the single strongest filter. Confluence score scales monotonically — at conf>=5, sweep_reversal hits 85.3% WR.

## Status / caveats

**DEAD.** This entire document is invalidated by two contamination layers: the [[wiki/concepts/v10i-look-ahead-bug|V10i look-ahead bug]] (MTF alignment used forming bar closes) and the [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] (trailing stops produce phantom edge on 1-min OHLC). The "zero overfit" claim is meaningless when both train and test sets contain the same structural bias. See [[wiki/summaries/bos-fvg-failure-consolidated|BOS FVG Failure Consolidated]] for the definitive paired bar-vs-tick test that killed BOS_FVG.

## Source documents

- [[raw-sources/trading/sf-portfolio/V8_MINING_SYNTHESIS]]
