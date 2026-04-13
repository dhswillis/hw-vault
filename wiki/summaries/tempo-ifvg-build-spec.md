---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/TEMPO_IFVG_BUILD_SPEC.md
related:
  - wiki/concepts/ifvg.md
  - wiki/entities/ninjatrader-v5.md
  - wiki/maps/tempo-moc.md
tags: [summary, ifvg, ninjatrader, build-spec]
---

# Tempo IFVG NinjaTrader Build Spec — Summary

> Implementation spec for TempoIFVGv1: sweep at key level → inverse FVG forms → candle closes through → market order reversal → partial TP + B/E + runner to session end.

## Key findings

NinjaTrader build spec (unmanaged orders, like SweepBreakv17.cs) for the canonical Tempo [[wiki/concepts/ifvg|IFVG]] strategy on NQ/MNQ. Step-by-step entry logic: (1) build key levels daily (PDH/PDL, Asia H/L, London H/L, AM H/L, 1H swing H/L, de-duplicated within 2pt), (2) detect sweep during entry window (bar pierces level then closes back through), (3) wait for inverse FVG formation, (4) enter on candle close through FVG with market order.

DST handling delegated to NinjaTrader's exchange timezone. Entry window, partial profit targets, break-even management, and runner to session end are all specified.

## Status / caveats

Build spec — describes intended behavior, not validated results. Cross-reference [[wiki/summaries/tempo-v14-corrections|V14 Corrections]] for bugs found in the actual NinjaTrader implementation that deviated from this spec.
