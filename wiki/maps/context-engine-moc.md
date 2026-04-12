---
created: 2026-04-11
updated: 2026-04-11
type: moc
tags: [moc, context-engine, trading, architecture]
sources:
  - raw-sources/trading/context-engine/MINING_ANALYSIS.md
  - raw-sources/trading/context-engine/NQ_PLAYBOOK.md
  - raw-sources/trading/context-engine/NAUTILUS_SPEC.md
  - raw-sources/trading/context-engine/TRANSCRIPT_MINING_FINDINGS.md
  - raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.1_Addendum.docx
  - raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.2_Addendum.docx
  - raw-sources/Tempo_Context_Engine_Spec.docx
related:
  - wiki/maps/root.md
  - wiki/maps/tempo-moc.md
---

# Context Engine MOC

> The planned Bayesian session classifier that sits between market data and trade decisions.
> Status: **spec only** — not yet built.

## What the Context Engine is

A real-time, multi-layer belief system that answers "what kind of session is this?" before any trade signal fires. Five feature layers, nine intraday checkpoints, Bayesian posterior updates. The idea is to stop taking A-setups in the wrong session type.

## Architecture (from spec v1.0)

```
Market data → [Layer 1: Session classifier]
            → [Layer 2: Volume profile]
            → [Layer 3: Footprint / orderflow]
            → [Layer 4: Cross-asset (SMT, DXY)]
            → [Layer 5: Calendar / news / time-of-day]
                    ↓
            Bayesian Belief Engine
                    ↓
            Session-type posterior
                    ↓
            Trade filter: allow / throttle / block
```

## Key concepts

- [[wiki/concepts/bayesian-belief-engine]] — The multinomial naive Bayes classifier at the heart.
- [[wiki/concepts/session-type-taxonomy]] — TREND / RANGE / EXPANSION / COMPRESSION / NEWS-SHOCK.
- [[wiki/concepts/volume-profile]] — POC, VA, HVN/LVN. Layers 2 and 3.
- [[wiki/concepts/smt]] — Smart Money Technique. Layer 4 cross-asset.
- [[wiki/concepts/dol-framework]] — Draw on Liquidity. Informs Layer 1 directional bias.

## Source documents

- [[wiki/summaries/tempo-context-engine-spec]] — Engineering spec v1.0. The canonical reference.
- [[wiki/summaries/transcript-mining-findings]] — Orderflow methodology from 7 educators. CLC framework, volume cluster pullbacks, POC reversal, footprint absorption. Feeds Layers 2–3.
- [[wiki/summaries/mining-analysis]] — V7aa_b cross-tab. **Suspect** (V10i bug). But session-type breakdowns may still inform the taxonomy.

## Raw sources not yet summarized

These context-engine raw sources need Ingest:

- `raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.1_Addendum.docx` — addendum v1.1
- `raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.2_Addendum.docx` — addendum v1.2

## Build phases (from spec)

1. Session classifier (posterior updates at 9 checkpoints)
2. Volume profile integration (Databento tick data)
3. Footprint / orderflow layer
4. Cross-asset layer (SMT, DXY, VIX)
5. Full Bayesian integration + trade filter API

**Current state:** Phase 0 — spec written, no code. [[wiki/entities/databento]] key saved on VPS but not integrated.

## Open questions

- Should the Context Engine run on QC ([[wiki/entities/quantconnect]]) or on the VPS alongside NinjaTrader?
- How to handle the transition from bar-level to tick-level data mid-session (Databento vs NinjaTrader feed)?
- Does the session-type taxonomy need a sixth category for FOMC / NFP sessions?
