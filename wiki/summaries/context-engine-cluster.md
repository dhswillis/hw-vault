---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/QUICK_REFERENCE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/TRADE_TYPES_VISUAL_GUIDE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/context-engine/QUICK_REFERENCE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/data_ingestion/README.md
related:
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/summaries/tempo-context-engine-spec-v1-1-addendum.md
  - wiki/summaries/tempo-context-engine-spec-v1-2-addendum.md
  - wiki/maps/context-engine-moc.md
  - wiki/concepts/bayesian-belief-engine.md
  - wiki/concepts/session-type-taxonomy.md
tags: [summary, cluster, context-engine, reference]
---

# Context Engine Reference — Cluster Summary

> Quick reference cards, trade type visual guide, and data ingestion README for the Tempo Context Engine.

## Contents

Supplementary documentation for the Context Engine, supporting the main spec (V1.0) and its addenda (V1.1, V1.2):

**QUICK_REFERENCE.md** (×2, duplicates across paths) — Condensed reference card for the context engine's session classification, feature layers, and checkpoint schedule. Designed for quick lookup during live trading.

**TRADE_TYPES_VISUAL_GUIDE.md** — Visual guide to the five session types (trend, range, expansion, compression, news-shock) and their sub-variants. Maps each type to recommended playbook configurations.

**data_ingestion/README.md** — Setup instructions for the data ingestion pipeline feeding the context engine.

## Status / caveats

Reference and operational docs. The Context Engine is not yet built (spec only) — see [[wiki/maps/context-engine-moc|Context Engine MOC]] for current status. These docs will become useful when implementation begins.

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/QUICK_REFERENCE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/TRADE_TYPES_VISUAL_GUIDE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/context-engine/QUICK_REFERENCE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/data_ingestion/README]]
