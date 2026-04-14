---
created: 2026-04-10
updated: 2026-04-10
type: entity
sources:
  - raw-sources/Tempo_Context_Engine_Spec.docx
  - raw-sources/Tempo_Quick_Start_Guide.docx
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/quantconnect.md
  - wiki/concepts/volume-profile.md
  - wiki/summaries/tempo-context-engine-spec.md
tags: [data, tick-data, market-microstructure, trading]
---

# Databento

Market data vendor providing tick-level trade data with aggressor side and L2 book snapshots. Used by [[tempo-trading-system|Tempo]] for everything beyond Layer 1 (price structure) features in the [[tempo-context-engine-spec|Context Engine]].

## What it provides

- **Tick-level trades** — every trade with timestamp, price, size, aggressor side (buy/sell)
- **L2 book snapshots** — bid/ask ladder state
- **NQ futures** — primary instrument, with ES/RTY/YM/cross-asset also available
- **Historical + real-time** feeds

## Why it's needed

[[quantconnect|QuantConnect]] provides OHLCV bars and trade data but **not** per-trade aggressor side. Aggressor-side data is the foundation for:

- [[volume-profile|Volume profile construction]] — POC, VA, HVN/LVN with buy/sell split
- **Delta analysis** — cumulative buy minus sell volume, divergence detection
- **Footprint imbalance stacks** — 3+ consecutive levels with >3:1 buy:sell ratio
- **Absorption detection** — high volume with minimal price movement
- **Exhaustion signals** — volume spike + delta extreme + price stall
- **Large trade attribution** — direction of trades ≥ 20 contracts for institutional flow tracking

Without aggressor side, you have to proxy it from mid-to-bid/mid-to-ask inference, which is noisy and often wrong near the inside.

## Cost

~$50–150/month for NQ historical tick data (per the Context Engine spec).

## Integration status (as of Feb 2026)

- **Key saved** on VPS at `/root/.env`
- **Not yet integrated** — planned for Context Engine Phase 2 (Week 2-3 of the build)
- **Phase 2 gate:** before shipping volume profile features, validate they improve classification accuracy over the Layer 1 baseline. "If volume doesn't improve accuracy, don't ship it — measure first."

## File layout (planned)

```
~/trading-system/context-engine/
  tick_puller.py             # Databento data retrieval
  data/tick_cache/           # Cached historical ticks
  data/volume_profiles/      # Pre-computed profiles per day
```

## Databento in prior Tempo work

The Tempo memory mentions `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/` — 312 days of Databento NQ trade data used for the mining reports. So Databento data has already been used offline for the [[strategy-mining-report-312d|V1 312d mine]] and the [[comprehensive-mining-report-v2|V2 cleaned rerun]]; it just hasn't been wired into the live Context Engine checkpoint runner yet.

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/EXCEL_STRATEGY_OVERVIEW|EXCEL STRATEGY OVERVIEW]]
- [[raw-sources/imports/2026-04-11/Documents/README_EXCEL_ANALYSIS|README EXCEL ANALYSIS]]
- [[raw-sources/imports/2026-04-11/Documents/V17_AUDIT_PROMPT|V17 AUDIT PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec|WickFade Strategy Spec]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/CONTEXT|CONTEXT]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/CLAUDE|CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/WORKING_LOG|WORKING LOG]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/CLAUDE_BOOTSTRAP|CLAUDE BOOTSTRAP]]
