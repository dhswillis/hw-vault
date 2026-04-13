---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/context-engine/Tempo_Context_Engine_Spec_V1.2_Addendum.docx
related:
  - wiki/summaries/tempo-context-engine-spec
  - wiki/maps/context-engine-moc
  - wiki/concepts/smt
tags: [summary, volatility-regime, smt-divergence, context-engine]
---

# Tempo Context Engine V1.2 Addendum — Summary

> Adds volatility regime sizing (A6) and smart money technique divergence (A7); 64 total features across 7 layers.

## Key findings

**A6: Volatility Regime & Dynamic Sizing** — Absorbs AlgoFlows severity scoring (realized vol + VIX momentum + VIX acceleration, weighted 0.40/0.35/0.25) into base k multiplier (0.06 LOW → 0.18 EXTREME). Four adjustments: VIX percentile, term structure stress (VIX9D>VIX), realized vol scaling, gap risk. Outputs recommended stop_distance (ATR × k_final) and position size (1% account risk). Session×Vol matrix applies multipliers (Trend 1.0x stop normal, Expansion 1.5x stop EXTREME, News-Shock STAND DOWN at extreme).

**A7: SMT Divergence Detection** — Compares NQ and ES swing structure on 15M and 1H to detect institutional disagreement (one makes new high, other fails). Three features: smt_divergence_15m/1h (-1/0/+1), smt_composite (-1 to +1). Bearish SMT during Trend-Up increases P(Trend→Reversal) by 15-25%; bullish SMT during Trend-Down inverted; SMT during Range increases P(Range→Expansion). Phase 1 uses SPY as ES proxy; Phase 3+ uses Databento ES tick for real-time detection.

**Output schema updated:** session_probabilities, opportunity_score, vol_regime (bucket 0-3, severity_score, recommended_stop_pts, recommended_size_contracts, session_vol_multiplier), smt_signal (composite, interpretation), confidence_meta (stability, flip_count, regime_alignment, decay_factor). Example JSON output provided.

Feature count: 46 (V1.0) + 9 (V1.1) + 9 (V1.2) = 64 features total.

## Status / caveats

Volatility regime uses free Yahoo data (VIX, realized vol). SMT Phase 1 uses SPY; full intraday power awaits Databento ES integration (Phase 3). Severity score formulas identical to AlgoFlows; integration adds session context and opportunity confidence. No external dependencies except data feeds. Ready for Phase 1 implementation.
