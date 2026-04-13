---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.1_Addendum.docx
related:
  - wiki/summaries/tempo-context-engine-spec
  - wiki/maps/context-engine-moc
  - wiki/concepts/bayesian-belief-engine
tags: [summary, context-engine, bayesian, regime, transition]
---

# Tempo Context Engine V1.1 Addendum — Summary

> V1.0 reviewed & expanded with 5 high-value additions: multi-day regime memory, gap intelligence, opportunity scoring, confidence decay, and transition modeling.

## Key findings

**A1: Multi-Day Regime Memory** — Rolling regime state above daily classifier answering "what has the market been doing this week?" Six features (regime_3d_vol, regime_5d_trend, regime_week_range, consecutive_day_type, macro_event_density, prior_week_outcome) feed into PRE_OVERNIGHT checkpoint as prior context, modulating daily classifier confidence.

**A2: Gap Intelligence Sub-Engine** — Expands 2 gap features to 8-10 dedicated features (gap_size_percentile, gap_direction_vs_trend, gap_into_hvn/lvn, gap_vs_prior_va, gap_fill_history, gap_news_context, gap_first_5min_reaction, weekly_gap). Classifies gaps as FILL_PROBABLE, CONTINUATION_PROBABLE, AMBIGUOUS, or NO_GAP with specific playbooks.

**A3: Opportunity Score Layer** — Separate output (0.0-1.0) independent of session type. Weighted components: classification confidence (30%), historical playbook performance (25%), volume participation (20%), prediction stability (15%), regime alignment (10%). Enables stand-down decisions (<0.30 score), reducing drawdown from low-quality days.

**A4: Confidence Stability & Decay** — Detects flip-flop predictions via stability score (1.0 - mean(prob_changes)) and flip_count. Response rules: stability >0.70 & flip ≤1 = full playbook; 0.50-0.70 = 25% size reduction; <0.50 = 50% reduction. All metrics logged for post-hoc validation.

**A5: Behavior Transition Model** — Probability matrix for regime switching (Compression→Expansion, Range→Trend, Trend→Reversal, etc.). Flags "transition warning" if P(transition) >0.40, triggering tighter stops and alternate playbook prep. Phase 3+ feature requiring volume/order flow data.

## Status / caveats

All additions integrate into existing checkpoint system (PRE_OVERNIGHT, OPEN_15MIN, etc.) without breaking prior architecture. Phases 1-2 enable most features via price + volume; Phases 3-5 unlock intraday transition detection. External review excluded "meta layer above everything" (redundant), "time of day personality" (checkpoint already encodes this), and deferred "execution friction awareness" to Phase 3+. No show-stoppers; ready to build sequentially.
