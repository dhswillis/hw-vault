---
created: 2026-04-10
updated: 2026-04-10
type: entity
sources:
  - raw-sources/Tempo_Quick_Start_Guide.docx
  - raw-sources/Tempo_Context_Engine_Spec.docx
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/tempo-quick-start-guide.md
  - wiki/summaries/tempo-context-engine-spec.md
tags: [platform, backtest, broker, trading]
---

# QuantConnect (QC)

Algorithmic trading and backtesting platform. Used by [[tempo-trading-system|Tempo]] as the execution engine for NQ futures strategies.

## Role in Tempo

- **Backtest runner** — strategies run here, not locally. The VPS sends variant configs via QC API; QC runs them; results come back as JSONL.
- **Primary strategy file** — `main.py` (currently uploaded from `main_v4.py`)
- **Batch target** — Batch 3 is 2000 variants being run through QC with realistic cost modeling (slippage, fees, tick-through, BE offset)
- **Project ID** — `28083727`
- **User ID** — `113205`

## Credentials

Located in `~/Documents/trading-system/.local/config.json` under the `quantconnect` key. Also in VPS `/root/.env`. **Never** paste in chat, docs, or committed code.

## Relation to other data sources

QC provides OHLCV bars (1s, 1m, 5m, 15m, 1D) and trade data for NQ futures — enough for Layer 1 (price structure) of the [[tempo-context-engine-spec|Context Engine]]. It does NOT provide tick-level aggressor-side data needed for Layer 2 (volume profile) or Layer 3 (footprint) features — that comes from [[databento|Databento]].

## Workflow

```
VPS batch-runner                         QuantConnect
    │                                         │
    ├── POST variant config  ─────────────►   │
    │                                         ├── run backtest
    │   ◄─────────── results JSON ────────────┤
    │                                         │
    └── write to ~/trading-system/batch-runner/results/
```

The two-stage batch runner works in this order: Stage A screens across a 3-month window, Stage B confirms the survivors across 12 months.

## Subscription

QC subscription is the only recurring cost for the backtest pipeline — Databento tick data is purchased separately for Layer 2/3 features.
