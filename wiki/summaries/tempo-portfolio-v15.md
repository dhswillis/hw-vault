---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/Tempo_Portfolio_V15.docx
related:
  - wiki/concepts/ifvg.md
  - wiki/summaries/lumi-strategy-spec.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/maps/tempo-moc.md
  - wiki/maps/strategies-moc.md
tags: [summary, portfolio, ifvg, lumi, tick-level]
---

# Tempo Portfolio V15 — Summary

> Combined IFVG + Lumi portfolio produces +19.8 pts/day across 1,208 trades over 264 days at tick level, Calmar 23.1.

## Key findings

V15 is a three-leg NQ portfolio audited tick-by-tick on Databento data across a full year (Feb 2025–Feb 2026, 264 trading days with 35 news days excluded). The portfolio combines the [[wiki/concepts/ifvg|IFVG]] strategy (US M60 + London Fade configs) with a rewritten Lumi engine based on the @LumiTraders specification.

The major V14→V15 changes are: Lumi engine rewritten to spec (HTF levels on M15/M30/H1/H4), MSS requirement removed (not in original spec, increased trade count from 605 to 775 without WR degradation), FVG soft stop replacing swing-extreme stops (average risk drops from ~29pt to ~9pt), and source filtering to exclude M15 levels which are the biggest drag. The recommended config is IFVG (US M60 + London Fade) + Lumi (H1/M30 sources, FVG soft stop, 2R from FVG).

## Status / caveats

The Lumi engine was separately tested at 20% WR in the v14 audit (see [[wiki/summaries/lumi-strategy-spec|Lumi Strategy Spec]]). The V15 rewrite to true @LumiTraders spec may have fixed the issues, but independent tick-level validation of the Lumi leg alone is recommended before trusting the combined portfolio numbers.
