---
created: 2026-04-11
updated: 2026-04-11
type: moc
tags: [moc, strategies, trading]
related:
  - wiki/maps/root.md
  - wiki/maps/tempo-moc.md
  - wiki/maps/bos-fvg-saga-moc.md
  - wiki/maps/audit-history-moc.md
---

# Strategies MOC

> Every signal family, its current status, and the evidence chain.
> Green = tick-validated. Red = dead. Yellow = suspect or untested.

## Strategy status board

| Strategy | Status | Evidence | Summary |
|---|---|---|---|
| **IFVG (Tempo canonical)** | **SIM ONLY** | 74.7% WR baseline, 83% with SMT + reaction quality | [[wiki/summaries/tempo-ifvg-research]] |
| **Wick Fade 5m** | **VALIDATED** | 10/10 walk-forward OOS positive, +77–94 R/day, Calmar 2.12 | [[wiki/summaries/wickfade-complete]] |
| **Wick Fade 15s** | **VALIDATED** | 43% WR, $30.89/trade on 260d tick-level | [[wiki/summaries/15s-wick-fade]] |
| **SF Portfolio** | **TICK-CONFIRMED** | +3.34 R/day on 260d. RUNNER leg fat-tailed (16% WR) | [[wiki/summaries/sf-portfolio-cluster]] |
| **BOS FVG** | **DEAD** | Bar: +0.359 avgR. Tick: +0.001 avgR. Invalidated. | [[wiki/summaries/bos-fvg-failure-consolidated]] |
| **Sweep Fade** | **DEAD/MARGINAL** | Most variants dead. Some fade-only configs marginal. | [[wiki/summaries/sweep-cluster]] |
| **Lumi** | **EXCLUDED** | v14 audit: 20% WR. Not viable. | [[wiki/summaries/lumi-strategy-spec]] |

## Validated strategies (the only ones that matter)

### Wick Fade family

The sole tick-validated survivors of the entire research arc. Fixed stop + fixed target — immune to [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]].

- [[wiki/summaries/wickfade-complete]] — 5-minute variant. Strongest anti-overfit evidence in the corpus.
- [[wiki/summaries/15s-wick-fade]] — 15-second variant. Flip variant dead (0/216 configs); fade alone is real.
- [[wiki/concepts/wick-fade]] — Concept page.

### SF Portfolio

- [[wiki/summaries/sf-portfolio-cluster]] — Sweep-Fail portfolio. Three legs: NQ VWPM, ES VWPM, NQ LF1170.
- [[wiki/entities/ninjatrader-v5]] — The live execution stack. 7 open bugs per [[wiki/summaries/v5-strategy-bug-audit]].

### IFVG (sim only, not yet live)

- [[wiki/concepts/ifvg]] — The signal definition.
- [[wiki/summaries/tempo-ifvg-research]] — Tick-level research. Cleanest Layer 2 doc.
- [[wiki/summaries/tempo-rules-v3]] — What Tempo actually teaches (Layer 1 canonical).

## Dead strategies — lessons learned

- [[wiki/maps/bos-fvg-saga-moc|BOS FVG Saga]] — Full case study in how a signal dies.
- [[wiki/summaries/sweep-cluster]] — Sweep-and-fade: why most died.
- [[wiki/summaries/lumi-strategy-spec]] — Lumi: why 20% WR isn't viable.

## Invalidation rules

Before trusting ANY pre-April 2026 number, check [[wiki/concepts/invalidation-rules]]. The "never again" list:

- No bar-sim trailing stops
- No V10i-style alignment
- No break-even stops on Model 2
- No phantom B/E fills
- No P/D full-day H/L filters (future data)

## What to build next

Per [[wiki/summaries/unified-brain-architecture]], the Unified Brain pivot means:

1. Only sim-trade IFVG MTF cascade (v24+)
2. Wick Fade and SF Portfolio are the only live-ready strategies
3. Context Engine ([[wiki/maps/context-engine-moc]]) provides the session filter
4. Nova + Sable ([[wiki/entities/nova-sable-brains]]) handle research tournaments
