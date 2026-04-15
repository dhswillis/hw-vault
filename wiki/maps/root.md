---
created: 2026-04-11
updated: 2026-04-11
type: moc
tags: [moc, root, architecture]
related:
  - BRAIN.md
  - index.md
---

# Root Map of Content

> Single entry point to the entire brain. Start here.
> See [[BRAIN]] for the six-layer architecture. See [[index]] for the flat catalog.

## The brain at a glance

```
              ┌────────────────────────────┐
              │      ROOT MOC (you are here)│
              └─────────┬──────────────────┘
        ┌───────────────┼───────────────────────┐
        │               │                       │
   ┌────▼────┐    ┌─────▼─────┐          ┌─────▼─────┐
   │ TRADING │    │ PERSONAL  │          │   WORK    │
   └────┬────┘    └─────┬─────┘          └─────┬─────┘
   ┌────┼────┐          │                      │
   │    │    │    goals, health,         Haven Park,
   │    │    │    finance, projects       decisions,
   │    │    │                           meetings
   │    │    │
   │    │    └── Context Engine
   │    └─────── Tempo Methodology
   └──────────── Research Arc + Audits
```

## Domain maps

### Trading — the core research system

- [[wiki/maps/tempo-moc|Tempo]] — Full Tempo methodology arc. Layer 1 canonical rules → Layer 2 mining/backtests → Layer 3 implementation. The primary strategy.
- [[wiki/maps/bos-fvg-saga-moc|BOS FVG Saga]] — Case study in how a signal goes from "97.9% WR" to DEAD. Three contamination layers, three audits.
- [[wiki/maps/audit-history-moc|Audit History]] — Every audit sorted by outcome. Bugs found, what they invalidated, rules produced. The quality-control backbone.
- [[wiki/maps/context-engine-moc|Context Engine]] — The planned Bayesian session classifier. Five feature layers, nine checkpoints. Not yet built.
- [[wiki/maps/strategies-moc|Strategies]] — All signal families: IFVG + Lumi V15 (live portfolio), SF Portfolio (tick-confirmed), Wick Fade (unproven), Sweep Fade (dead/marginal), BOS FVG (dead). Status tracker.

### Research arc

- [[wiki/syntheses/research-arc-map|Research Arc Map]] — Master synthesis of V7→V11. Five phases, five bug waves, three parallel tracks, Unified Brain pivot.
- [[wiki/syntheses/tempo-three-layers|Three-Layer Framework]] — Every Tempo claim must be placed into Layer 1/2/3 before evaluation.
- [[wiki/syntheses/mining-reports-v1-v2-reconciliation|V1 vs V2 Reconciliation]] — Why V1 claimed 97.9% WR while V2 shows 63.5%.
- [[wiki/syntheses/bos-fvg-claim-vs-reality|BOS FVG Claim vs Reality]] — Why "BOS_FVG is the core signal of Tempo" was a two-layer mistake.

### Personal

- [[wiki/maps/personal-moc|Personal]] — Goals, health, finance, personal projects. Tempo trading as a personal project lives here.

### Work

- [[wiki/maps/work-moc|Work]] — Haven Park Communities. Director of Asset Management, PE-backed MHC platform (30k→60-80k units). Decisions, meetings, people.

### Operations & automation

- [[wiki/maps/automation-moc|Automation]] — Scheduled tasks, ingest pipelines, lint sweeps. How the brain maintains itself.
- [[wiki/maps/import-triage-moc|Import Triage]] — The 166-file import batch from 2026-04-11. Mapping to existing wiki pages.

## Key concepts (top 10 by connectivity)

These are the most-linked nodes in the graph — the brain's load-bearing ideas.

1. [[wiki/concepts/bos-fvg]] — **DEAD.** Bar-sim trailing inflated results. Tick replay: +0.001 avgR.
2. [[wiki/concepts/v10i-look-ahead-bug]] — Contaminated all V1-era mining. Fixed in V2.
3. [[wiki/concepts/bar-sim-trailing-bug]] — Trailing stops can't be backtested on 1-min OHLC. Caused the BOS_FVG illusion.
4. [[wiki/concepts/ifvg]] — Tempo's canonical entry signal. The real thing BOS_FVG was trying to be.
5. [[wiki/concepts/smt]] — NQ vs ES divergence. Primary confluence since Nov 2025.
6. [[wiki/concepts/tempo-v14-corrections]] — Four bugs in v14 NinjaTrader. Different strategy than Tempo teaches.
7. [[wiki/concepts/be-trail-mechanism]] — Break-even + trailing stop. The mechanism behind the bar-sim illusion.
8. [[wiki/concepts/dol-framework]] — Draw on Liquidity hierarchy.
9. [[wiki/concepts/volume-profile]] — POC, VA, HVN/LVN. Context Engine Layer 2/3.
10. [[wiki/concepts/bayesian-belief-engine]] — Heart of the Context Engine.

## Key entities

- [[wiki/entities/tempo-methodology]] — Tempo the educator. 281 recaps, 88% WR.
- [[wiki/entities/tempo-trading-system]] — The overall system at `~/Documents/trading-system/`.
- [[wiki/entities/quantconnect]] — Backtest platform. Project `28083727`.
- [[wiki/entities/ninjatrader-v5]] — V5 SF portfolio. 7 open bugs.
- [[wiki/entities/databento]] — Tick-level data vendor.
- [[wiki/entities/nova-sable-brains]] — AI researchers in the Unified Brain.

## Vault health

- **305 files** | 264 have inbound links | 26 orphans (mostly .gitkeep placeholders)
- **224 raw sources** (58 original + 166 imported) | **26 summaries written** | ~200 sources pending ingest
- **17 concepts** | **6 entities** | **4 syntheses** | **4+ MOCs**
- Last lint: pending — see [[wiki/maps/automation-moc|Automation]] for schedule.
