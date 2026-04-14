---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/NEW_SIGNAL_CLASSES_RESEARCH.md
related:
  - wiki/concepts/ifvg.md
  - wiki/maps/strategies-moc.md
tags: [summary, signals, research, new-strategies]
---

# New Signal Classes Research — Summary

> 7 genuinely new signal classes discovered across V7 studies. FVG Bifurcation (Priority 1), Inverse/Hedge (Priority 2), Composite GOLD (Priority 3). Current system at +11.31 R/day from 4 signals.

## Key findings

Research report identifying 7 new trade entry types with independent logic that would generate additional non-correlated trade populations. Each evaluated on edge evidence, expected metrics, implementation spec, and priority.

Top three by priority: FVG Bifurcation (passive vs momentum entry, expected +2–4 R/day, strong evidence from V7aa/V7r), Inverse/Hedge (expected +1–2 R/day additive, validated in V7k-V7m), Composite GOLD (expected +1–3 R/day, strong evidence from V7r/V7t). Lower priority: Wick Follow, Stop Limit Breakout, Sweep Close Market, and Pattern Mining (6,502 combos).

## Status / caveats

**Suspect.** Feb 2026 research predating the [[wiki/concepts/v10i-look-ahead-bug|V10i look-ahead bug]] and [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] discoveries. The V7-era studies that support these signals are contaminated. Expected R/day figures should not be trusted. The FVG Bifurcation concept is sound but needs re-validation on clean data.

## Source documents

- [[raw-sources/trading/NEW_SIGNAL_CLASSES_RESEARCH]]
