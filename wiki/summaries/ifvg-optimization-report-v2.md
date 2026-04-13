---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/strategies/tempo/results/IFVG_Optimization_Report_v2.docx
related:
  - wiki/concepts/ifvg.md
  - wiki/summaries/ifvg-full-year-verification-memo.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/maps/tempo-moc.md
tags: [summary, ifvg, optimization, hard-stop]
---

# IFVG Optimization Report V2 — Summary

> Hard stop at FVG edge + 1.5pts cuts average loss from -1.95R to -1.07R, nearly triples $/day, and halves max drawdown.

## Key findings

Three major discoveries from the 30-day IFVG optimization sample (Jan–Feb 2026, short-only, 1-per-sweep):

**Pipeline diagnostic:** The signal pool is massive — 42.8 valid sweeps/day, 224.3 FVGs. Previous low trade counts (0.36–0.78 T/d) were entirely caused by ultra-tight filters (gap ≤2, risk ≤10) discarding 99% of signals. Relaxing to gap ≤6 with reaction ≥3 produces 13.6 entries/day.

**R-per-trade validation:** Soft stop losers average -1.9R instead of the theoretical -1.0R. Winners collect +1.89R (RT2) or +2.88R (RT3), close to target. The stop overshoot is the single biggest leak.

**Hard stop discovery:** Replacing the bar-close soft stop with a tick-level hard stop at FVG edge + 1.5pts fixes the leak — average loss drops to -1.07R, nearly triples $/day on RT3 configs, and halves max drawdown.

## Status / caveats

30-day sample only — needs full-year validation with hard stop enabled. The pipeline diagnostic is valuable for understanding trade frequency tuning across all IFVG configs.
