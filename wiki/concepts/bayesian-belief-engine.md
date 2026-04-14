---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/Tempo_Context_Engine_Spec.docx
related:
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/concepts/session-type-taxonomy.md
  - wiki/concepts/volume-profile.md
tags: [trading, context-engine, bayesian, classifier]
---

# Bayesian Belief Engine (Tempo Context Engine Core)

The classifier at the heart of the [[tempo-context-engine-spec|Tempo Context Engine]]. It maintains a live probability distribution over [[session-type-taxonomy|session types]] and updates that distribution at each intra-session checkpoint as new evidence arrives.

## Not a price predictor

The engine does **not** predict price. It predicts **session behavior type**, then maps that type to which strategy configuration historically performs best in that environment.

> The question is not "what is the best strategy?" It is "what is the best strategy for THIS environment?"

## Belief update mechanism

At each checkpoint, compute a likelihood ratio for each session type based on the new feature values, then apply Bayes rule to update the prior:

```python
def update_beliefs(prior_probs, new_features, checkpoint_name):
    likelihoods = compute_likelihoods(new_features, checkpoint_name)
    posterior = {
        t: prior_probs[t] * likelihoods[t]
        for t in SESSION_TYPES
    }
    total = sum(posterior.values())
    posterior = {k: v / total for k, v in posterior.items()}
    log_checkpoint(checkpoint_name, prior_probs, likelihoods, posterior, new_features)
    return posterior
```

Starting model: multinomial naive Bayes. Each feature's `P(value | session_type)` is learned from history; the posterior is re-normalized to sum to 1.0.

## Checkpoint schedule

Nine checkpoints span overnight → EOD:

| Checkpoint | Time (ET) | What arrives | Purpose |
|---|---|---|---|
| `PRE_OVERNIGHT` | 6:00 PM prior day | Prior day structure, volume profile, tomorrow's calendar | Initial overnight prior |
| `OVERNIGHT_CLOSE` | 8:00 AM | Overnight range, direction, volume, gap | Pre-market prediction |
| `PRE_MARKET` | 9:15 AM | Pre-market range, delta, cross-asset | Refine with live data |
| `OPEN_5MIN` | 9:35 AM | Open drive, initial volume surge | Confirm/reject pre-market view |
| `OPEN_15MIN` | 9:45 AM | First-15 range, POC formation, delta, large trades | Primary confirmation checkpoint |
| `OPEN_60MIN` | 10:30 AM | First-hour range, POC migration, value area, delta trend | Day type classification anchor |
| `MID_SESSION` | 12:00 PM | Lunch volume decline, POC stability | Regime change detection for PM |
| `PM_OPEN` | 1:30 PM | PM energy, volume return, delta shift | PM playbook selection |
| `LATE_SESSION` | 3:00 PM | MOC flow proxies, late delta, final POC | EOD exit/hold decision |

## Learning loop

After each trading day:

1. Classify the **actual** session type from full-day data
2. Compare actual to what the engine predicted at each checkpoint
3. Update `P(feature_value | session_type)` tables
4. Rolling **60-day window** — not all-time, so the model adapts to regime changes
5. Daily incremental update; weekly full retrain
6. Automatic feature pruning: drop any feature that doesn't improve accuracy over the prior-layer baseline
7. Weekly report: top 10 features by permutation importance, movement in/out of top 10

## Logging

Every checkpoint writes a JSON record with the prior, likelihoods, posterior, evidence, selected playbook, and confidence. End-of-day entries add the actual classification and per-checkpoint accuracy scores. This creates a full audit trail and is how the engine measures its own calibration.

## Success criteria

- **Classification accuracy > 55%** (random baseline 20% across 5 classes)
- **Top-2 accuracy > 75%**
- **Brier score < 0.35** (mean squared error of probability estimates)
- **Calibration within 10%** across all probability bins — when the engine says 60% TREND, TREND should happen ~60% of the time
- **Positive checkpoint lift** at `OPEN_15MIN` and `OPEN_60MIN` (evidence must actually improve the distribution)
- **Playbook PnL delta positive** over 30+ day windows (context-selected config beats best static config)
- **Correctly identifies stand-down days** ≥ 70% of the time

## Why Bayesian, not ML?

The spec starts with multinomial naive Bayes explicitly because it's interpretable, trains incrementally, and outputs calibrated probabilities out of the box. An opaque neural classifier could match accuracy but wouldn't expose the likelihood table — and the whole point of checkpoint logging is post-hoc analysis of *which features moved the posterior which way*.

If Bayesian is insufficient after the Phase 1-4 buildout, the architecture supports swapping in a more expressive classifier — but only after the baseline's calibration is measured.

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE|START HERE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/trading_operator/logs/health_check|health check]]
- [[raw-sources/imports/2026-04-11/trading-system/START_HERE|START HERE]]
- [[raw-sources/imports/2026-04-11/trading-system/repo_inventory|repo inventory]]
