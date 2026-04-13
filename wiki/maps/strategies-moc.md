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
- [[wiki/summaries/wickfade-5m-research-findings]] — Earlier 5M research session with flip timing artifact analysis.
- [[wiki/summaries/wickfade-strategy-findings]] — Optimization research. Trail on both legs was the biggest improvement.
- [[wiki/summaries/15s-wick-fade]] — 15-second variant. Flip variant dead (0/216 configs); fade alone is real.
- [[wiki/concepts/wick-fade]] — Concept page.

### SF Portfolio

- [[wiki/summaries/sf-portfolio-cluster]] — Sweep-Fail portfolio cluster. Tick-level simulator confirmed.
- [[wiki/summaries/final-portfolio-spec]] — Final NQ+ES spec: +3.34 R/Day, 13 legs, zero look-ahead.
- [[wiki/summaries/sf-portfolio-state]] — March 2026 portfolio state: 12-leg breakdown, +2.26 R/Day.
- [[wiki/summaries/nq-sf-engulfing-strategy]] — V1 SF Engulfing docs. Zero losing weeks across 260 days.
- [[wiki/summaries/nq-sf-engulfing-strategy-v2]] — V2 with Leppyrd ICT framework.
- [[wiki/summaries/nq-portfolio-trading-system]] — 9-strategy portfolio: ~987R/year.
- [[wiki/summaries/deep-mine-findings]] — Fib retracement deep mine: 96K trades, three breakthrough filters.
- [[wiki/entities/ninjatrader-v5]] — The live execution stack. 7 open bugs per [[wiki/summaries/v5-strategy-bug-audit]].

### IFVG (sim only, not yet live)

- [[wiki/concepts/ifvg]] — The signal definition.
- [[wiki/summaries/tempo-ifvg-research]] — Tick-level research. Cleanest Layer 2 doc.
- [[wiki/summaries/ifvg-full-year-verification-memo]] — 244-day full year verification, code-audited.
- [[wiki/summaries/ifvg-optimization-report-v2]] — Hard stop discovery: triples $/day, halves drawdown.
- [[wiki/summaries/ifvg-tp-optimization-report]] — TP optimization: fixed 1.25R beats session-aware partials.
- [[wiki/summaries/tempo-ifvg-build-spec]] — NinjaTrader build spec for TempoIFVGv1.
- [[wiki/summaries/tempo-ifvg-audit-report]] — V14 post-fix tick audit: US +24.5 pts/day.
- [[wiki/summaries/fvg-entry-timing-analysis]] — Passive fill vs momentum inversion entry regimes.
- [[wiki/summaries/tempo-rules-v3]] — What Tempo actually teaches (Layer 1 canonical).
- [[wiki/summaries/tempo-portfolio-v15]] — V15 IFVG + Lumi combined: +19.8 pts/day.

## Dead strategies — lessons learned

- [[wiki/maps/bos-fvg-saga-moc|BOS FVG Saga]] — Full case study in how a signal dies.
- [[wiki/summaries/v8-mining-synthesis]] — V8 mining: impressive numbers, all invalidated by look-ahead + bar-sim bugs.
- [[wiki/summaries/sweep-cluster]] — Sweep-and-fade: why most died.
- [[wiki/summaries/sweep-fade-research]] — Full tick-level sweep fade research (+10.43 R/day) — needs bar-sim check on trailing exits.
- [[wiki/summaries/keylevel-sweep-cluster]] — NinjaTrader build docs for the sweep strategy.
- [[wiki/summaries/lumi-strategy-spec]] — Lumi: why 20% WR isn't viable.

## Additional research

- [[wiki/summaries/new-signal-classes-research]] — 7 new signal classes from V7 studies (suspect — V7-era contamination).
- [[wiki/summaries/tempo-carmine-strategy-2026-02-16]] — Tempo + Carmine integrated analysis (suspect).
- [[wiki/summaries/tempo-strategy-analysis-2026-02-16]] — Deep multivariate analysis, 400 variants (suspect).
- [[wiki/summaries/big-ride-research]] — HOD/LOD capture research, daily range timing.
- [[wiki/summaries/doji-break-strategy]] — Doji bracket orders. Edge is entirely in B/E management.
- [[wiki/summaries/daily-range-research]] — Daily range capture with pyramiding.
- [[wiki/summaries/willis-holdings-strategy-audit]] — March 2026 strategy status report.

## Invalidation rules

Before trusting ANY pre-April 2026 number, check [[wiki/concepts/invalidation-rules]]. The "never again" list:

- No bar-sim trailing stops
- No V10i-style alignment
- No break-even stops on Model 2
- No phantom B/E fills
- No P/D full-day H/L filters (future data)

## Architecture & infrastructure

- [[wiki/summaries/option4-hybrid-architecture|Option 4: Hybrid Full Stack]] — Architecture document for the hybrid handoff approach.
- [[wiki/summaries/implementation-review]] — Multi-model agent system + backtesting engine selection review.
- [[wiki/summaries/autonomous-trading-system-progress-summary]] — Autonomous system build progress.
- [[wiki/summaries/openclaw-vps-setup-guide]] — VPS setup for autonomous operation.
- [[wiki/summaries/conglomerate-ceo]] — Willis Holdings conglomerate CEO vision.
- [[wiki/summaries/tempo-ibkr-migration-plan]] — IBKR Python migration from QC.
- [[wiki/summaries/aio-system-blueprint]] — All-in-one autonomous trading + broadcasting.
- [[wiki/summaries/ai-trading-show-blueprint]] — AI Trading Show: Nova & Sable live stream concept.
- [[wiki/summaries/unified-brain-options-and-growth]] — Unified Brain deployment tiers and cost analysis.
- [[wiki/summaries/batch-runner-cluster]] — Batch backtesting infrastructure docs.
- [[wiki/summaries/tempo-pipeline-cluster]] — Tempo extraction → QC implementation pipeline.
- [[wiki/summaries/tempo-batch-cheatsheet]] — QC batch runner quick reference.
- [[wiki/summaries/video-ingestion-pipeline-guide]] — Video → strategy extraction pipeline.

## What to build next

Per [[wiki/summaries/unified-brain-architecture]], the Unified Brain pivot means:

1. Only sim-trade IFVG MTF cascade (v24+)
2. Wick Fade and SF Portfolio are the only live-ready strategies
3. Context Engine ([[wiki/maps/context-engine-moc]]) provides the session filter
4. Nova + Sable ([[wiki/entities/nova-sable-brains]]) handle research tournaments
