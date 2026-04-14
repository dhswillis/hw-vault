---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/Tempo_Context_Engine_Spec.docx
related:
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/concepts/volume-profile.md
  - wiki/concepts/bayesian-belief-engine.md
tags: [trading, context-engine, taxonomy]
---

# Session Type Taxonomy

The Tempo [[tempo-context-engine-spec|Context Engine]] classifies every trading session into one of five primary types. These are **not mutually exclusive** — the engine emits a probability distribution across all five, and the [[bayesian-belief-engine|Bayesian belief update]] refines the distribution at each checkpoint.

## The five primary types

| Type | Characteristics | Best playbooks |
|---|---|---|
| **TREND** | Directional, pullbacks hold, POC migrates, delta stays one-sided, ATR expanding | Pullback entries, runner exits, trend continuation, hold through lunch |
| **RANGE** | Rotational, edges rejected, POC stable, delta mean-reverts, ATR contracting | Sweep reversals, edge fades, mean reversion exits, tight targets |
| **EXPANSION** | Breakout from compression, volume spike at break, POC shifts rapidly, range doubles prior day | Breakout entries, wide stops, momentum targets, reduce trade frequency |
| **COMPRESSION** | Narrowing range, declining volume, POC flat, delta negligible, inside day forming | Stand down or very tight scalps, wait for expansion trigger |
| **NEWS-SHOCK** | Event-driven spike, 3x+ volume, immediate directional move, fast mean reversion possible | Pre-event: stand down. Post-event: fade extremes or ride continuation depending on delta |

## Sub-classifications

Each primary type has sub-variants that refine playbook selection:

- **TREND-CONTINUATION** — same direction as prior session/overnight. Lean harder into pullbacks.
- **TREND-REVERSAL** — opposite direction from prior bias. Wait for confirmation, tighter stops.
- **RANGE-WIDE** — 80+ point range. Multiple edge-to-edge opportunities.
- **RANGE-TIGHT** — <40 point range. Few opportunities, chop risk.
- **EXPANSION-GAP** — gap outside prior day range triggers expansion. Gap direction matters.
- **NEWS-SCHEDULED** — known event (CPI, FOMC, NFP). Pre-position rules apply.
- **NEWS-SURPRISE** — unexpected headline. React-only mode.

## How the probability distribution maps to playbooks

| Distribution | Playbook | Config summary |
|---|---|---|
| P(TREND) > 0.50 AND `poc_migration` confirmed | TREND_PULLBACK | `sweep_fvg`, R 3.0-5.0, BE 0.5R, hold runners |
| P(RANGE) > 0.50 AND prior range > median | EDGE_FADE | `sweep_fvg`, R 1.0-2.0, BE 0.25R, NY_AM+NY_PM, tight stops |
| P(EXPANSION) > 0.40 AND volume_spike | BREAKOUT_RIDE | `htf_fvg_fill`, R 5.0-10.0, BE 1.0R, wide stops |
| P(COMPRESSION) > 0.45 | COMPRESSION_WAIT | No trades or minimal scalps, 50% size |
| P(NEWS_SHOCK) > 0.35 OR news_proximity < 30min | NEWS_REACTIVE | Pre: flat. Post: first 15min only |
| P(TREND) > P(RANGE), neither > 0.50 | MIXED_LEAN_TREND | Trend entries, tighter targets (2R), faster BE |
| P(RANGE) > P(TREND), neither > 0.50 | MIXED_LEAN_RANGE | Edge fades, looser stops, willing to flip |

## Dynamic playbook switching

- Switch requires probability shift **> 0.20** at a single checkpoint (prevents noise-driven flipping)
- Maximum **2 switches per day** (prevents whipsawing)
- **No switching during active trades** — existing positions follow original playbook to exit
- Each switch is logged with the evidence that triggered it
- If max probability drops below **0.35**, reduce position size by 50% (uncertain regime)

## How it's learned

The engine doesn't assume which features distinguish the types. Historical sessions are labeled from full-day data (range vs ATR, POC stability, delta persistence). For each feature and checkpoint, `P(feature_value | session_type)` is estimated. At inference, the [[bayesian-belief-engine|Bayes update]] combines these likelihoods with the running prior to produce the posterior distribution.

Rolling 60-day training window adapts to regime changes. Features that don't improve accuracy over baseline are automatically pruned.

## Accuracy targets

- Classification accuracy > 55% (random = 20% across 5 classes)
- Top-2 accuracy > 75%
- Brier score < 0.35
- Calibration within 10% across probability bins
- Positive checkpoint lift at `OPEN_15MIN` and `OPEN_60MIN`

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE|START HERE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/trading_operator/logs/health_check|health check]]
- [[raw-sources/imports/2026-04-11/trading-system/START_HERE|START HERE]]
- [[raw-sources/imports/2026-04-11/trading-system/repo_inventory|repo inventory]]
