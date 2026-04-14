---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sf-portfolio/DEEP_MINE_FINDINGS.md
related:
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/maps/strategies-moc.md
tags: [summary, mining, fib-retracement, filters]
---

# Deep Mine Findings (Fib Retracement) — Summary

> 96,552 fib trades mined, 30,000+ filter configs tested. Three breakthrough filters produce 100% positive weeks: No-Poison Candle, EMA21 Direction, Minimum Leg Size. Top config: +7.26 R/day.

## Key findings

Exhaustive mining on NQ fib retracement strategy across 258 trading days. Three transformational filters discovered: No-Poison Candle (removes marubozu + strong_body confirmation, biggest single improvement: 79% → 100% positive weeks), EMA21 Direction (long only above, short only below, eliminates counter-trend), Minimum Leg Size (≥5 candles, ensures real impulse).

Top config: 1M L+S s0 lg>=5 c>=1 no_poison with BE0.3→1R — +7.26 R/day, 100% positive weeks, worst week is breakeven. After commission: +5.51 R/d, 98% weeks positive. Half-split robust (H1: +8.09, H2: +6.52 R/d).

## Status / caveats

**Suspect.** Uses B/E management which may be vulnerable to the [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] depending on simulation method. The +7.26 R/day headline needs tick-level validation. Also check if EMA21 filter creates a subtle look-ahead when calculated on the entry bar.

## Source documents

- [[raw-sources/trading/sf-portfolio/DEEP_MINE_FINDINGS]]
