---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources: [raw-sources/Tempo_Context_Engine_Spec.docx]
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/databento.md
  - wiki/entities/quantconnect.md
  - wiki/concepts/session-type-taxonomy.md
  - wiki/concepts/volume-profile.md
  - wiki/concepts/bayesian-belief-engine.md
tags: [trading, tempo, context-engine, spec, design-doc]
---

# Tempo Context Engine — Engineering Specification v1.0

**Source:** `raw-sources/Tempo_Context_Engine_Spec.docx`
**Dated:** February 2026
**Subsystem of:** [[tempo-trading-system]]

## What this system does

A Python service on the VPS that forms a probabilistic view of what kind of trading session is about to happen, then updates that view in real time as evidence arrives. It outputs a probability distribution over [[session-type-taxonomy|session types]] and a recommended playbook configuration for the QC strategy.

> It is NOT a price prediction model. It predicts session **behavior type**, then maps that to which strategy configurations historically perform best in that environment.

Core loop: Gather features → Classify session type → Assign probabilities → Select playbook → Log state → Update at checkpoints → Repeat.

## Session types

Five primary types (not mutually exclusive — the engine outputs a distribution):

| Type | Characteristics | Best playbooks |
|---|---|---|
| **TREND** | Directional, pullbacks hold, POC migrates, delta one-sided, ATR expanding | Pullback entries, runner exits, hold through lunch |
| **RANGE** | Rotational, edges rejected, POC stable, delta mean-reverts | Sweep reversals, edge fades, tight targets |
| **EXPANSION** | Breakout from compression, volume spike, POC shifts rapidly | Breakout entries, wide stops, momentum targets |
| **COMPRESSION** | Narrowing range, declining volume, POC flat, inside day forming | Stand down or tight scalps only |
| **NEWS-SHOCK** | Event spike, 3x+ volume, immediate directional move | Pre-event stand down; post-event fade or ride based on delta |

Sub-variants refine further: TREND-CONTINUATION vs TREND-REVERSAL, RANGE-WIDE vs RANGE-TIGHT, EXPANSION-GAP, NEWS-SCHEDULED vs NEWS-SURPRISE.

## Five feature layers

Feature engineering is incremental — each layer adds predictive power, built on top of the last. MVP is Layer 1 only.

1. **Layer 1 — Price Structure** (no external data): `prior_day_type`, `gap_size_pct`, `inside_day`, `open_drive_range`, `first_hour_range`, etc. Computable from QC historical data alone.
2. **Layer 2 — [[volume-profile|Volume Profile]]**: `session_poc`, `poc_migration`, `value_area_width`, `hvn_proximity`, `lvn_proximity`, `relative_volume`, `volume_at_extremes`.
3. **Layer 3 — Order Flow / Footprint Proxies**: `session_delta`, `delta_divergence`, `large_trade_count`, `absorption_score`, `imbalance_stacks`, `exhaustion_signal`. Requires tick-level data with aggressor side.
4. **Layer 4 — News & Calendar Context**: `news_impact_today`, `news_time_proximity`, `fomc_week`, `earnings_impact`. Free economic calendar APIs.
5. **Layer 5 — Cross-Asset Signals**: `vix_level`, `nq_es_spread`, `yield_direction`, `dxy_direction`, `sector_rotation`.

## Intra-session checkpoints

The engine maintains a live belief state, updated at defined checkpoints:

| Checkpoint | Time (ET) | Purpose |
|---|---|---|
| PRE_OVERNIGHT | 6:00 PM prior day | Initial prior |
| OVERNIGHT_CLOSE | 8:00 AM | Pre-market prediction |
| PRE_MARKET | 9:15 AM | Refine with live data |
| OPEN_5MIN | 9:35 AM | Confirm/reject pre-market view |
| OPEN_15MIN | 9:45 AM | Primary confirmation |
| OPEN_60MIN | 10:30 AM | Day type classification anchor |
| MID_SESSION | 12:00 PM | Regime change detection |
| PM_OPEN | 1:30 PM | PM playbook selection |
| LATE_SESSION | 3:00 PM | EOD exit/hold decision |

Each checkpoint recomputes features and does a [[bayesian-belief-engine|Bayesian belief update]]:
```python
posterior[type] = prior[type] * likelihoods[type]
posterior /= posterior.sum()
```

Likelihoods are learned from historical data: given a day turned out TREND, what was the feature distribution at this checkpoint?

## Playbook selection

The probability distribution maps to a specific QC strategy config:

| Playbook | Trigger | QC config |
|---|---|---|
| TREND_PULLBACK | P(TREND) > 0.50 AND poc_migration confirmed | `signal=sweep_fvg, R_target=3.0-5.0, BE=0.5R, hold_runners=true` |
| EDGE_FADE | P(RANGE) > 0.50 AND prior_day_range > median | `signal=sweep_fvg, R_target=1.0-2.0, BE=0.25R, NY_AM+NY_PM, tight_stops` |
| BREAKOUT_RIDE | P(EXPANSION) > 0.40 AND volume_spike | `signal=htf_fvg_fill, R_target=5.0-10.0, BE=1.0R, NY_AM, wide_stops` |
| COMPRESSION_WAIT | P(COMPRESSION) > 0.45 | No trades or minimal scalps, 50% size |
| NEWS_REACTIVE | P(NEWS_SHOCK) > 0.35 OR news_proximity < 30min | Pre-event: flat. Post-event: first 15min only |
| MIXED_LEAN_TREND/RANGE | Neither > 0.50 | Blended config |

Dynamic switching rules: probability shift > 0.20 required, max 2 switches per day, no switching during active trades, confidence < 0.35 → reduce size 50%.

## Volume profile engine (deep dive)

See [[volume-profile]] for the full concept. Key metrics computed from tick data:

- **POC** — highest-volume price level, fair-value anchor. POC migration confirms trend.
- **Value Area** — 70% volume range, dynamic S/R.
- **HVN/LVN** — high/low volume nodes. HVNs act as magnets, LVNs as air pockets.
- **Delta at level** — buy − sell volume at a price. Divergence = reversal signal.
- **Imbalance stacks** — 3+ consecutive levels with >3:1 buy:sell ratio.
- **Absorption** — high volume + minimal price change = large resting orders.
- **Exhaustion** — volume spike + delta extreme + price stall = reversal.

## Data pipeline

| Source | Data | Cost |
|---|---|---|
| [[quantconnect|QuantConnect]] | OHLCV bars, NQ trade data | Included |
| [[databento|Databento]] | Tick-level trades with aggressor side, L2 book | ~$50-150/mo |
| Econ Calendar API | CPI, NFP, FOMC, earnings | Free |
| Cross-Asset | ES, RTY, VIX, DXY, 10Y yield | Included |

File layout on VPS:
```
~/trading-system/context-engine/
  tick_puller.py
  volume_engine.py
  calendar_tagger.py
  feature_builder.py
  classifier.py
  checkpoint_runner.py
  trainer.py
  data/ {tick_cache, volume_profiles, features, models}
  logs/ {probability_log.jsonl, accuracy_log.jsonl}
```

## Self-improving learning loop

- After each day, classify actual session type from full-day data
- Compare to predictions at each checkpoint
- Update `P(feature_value | session_type)` tables
- Rolling 60-day training window (regime adaptation)
- Weekly full retrain, daily incremental
- Automatic feature pruning via permutation importance

**Accuracy targets:**
- Classification accuracy > 55% (random = 20%)
- Top-2 accuracy > 75%
- Brier score < 0.35
- Calibration within 10% across bins
- Checkpoint lift positive at OPEN_15MIN and OPEN_60MIN

## Build phases

1. **Phase 1 (Week 1)** — Price-only classifier, Layer 1 only, multinomial naive Bayes. Data already in QC.
2. **Phase 2 (Week 2-3)** — Buy Databento tick data Jul-Sep 2025, add volume profile. Measure whether it improves accuracy before shipping.
3. **Phase 3 (Week 3-4)** — Footprint features, intra-session checkpoint runner.
4. **Phase 4 (Week 4-5)** — News/calendar + cross-asset.
5. **Phase 5 (Week 5-6)** — Live integration, checkpoint runner on real-time Databento, Telegram alerts via OpenClaw.

## Success criteria

- Classification accuracy > 55% on session type
- Context-selected playbook beats best static config over 60+ day window
- Correctly identifies stand-down days ≥ 70% of the time
- Feature importance rankings stable across 30-day windows
- Intra-session checkpoint improves final prediction by ≥ 10 percentage points vs pre-session
- Full pipeline runs autonomously on VPS
