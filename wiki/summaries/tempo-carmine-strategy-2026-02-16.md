---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/trading-system/TEMPO_Carmine_Strategy_2026-02-16.docx
related:
  - wiki/maps/tempo-moc.md
  - wiki/maps/strategies-moc.md
  - wiki/summaries/tempo-strategy-analysis-2026-02-16.md
  - wiki/concepts/ifvg.md
tags: [summary, tempo, carmine, multi-strategy, suspect-results]
---

# Tempo + Carmine Integrated Strategy — Summary

> 400 variants screened, 50 deep-tested, 2 complete strategies. Overlap session is 100% profitable. Combo signals produce +1,401% Sharpe improvement over single signals.

## Key findings

Integrates Tempo's batch runner data with Carmine Rosato's supply/demand and order flow methodology. Two strategies emerged: "The Confluence Sniper" (targeting 60% daily WR) and "The Asymmetric Portfolio" (targeting 20R/day).

The single most powerful finding is session timing: overlap session (London/NY crossover, 8am–12pm ET) produced a 100% profitable variant rate. Combined with dual-signal requirement (both sweep_fvg and htf_fvg_fill active), outside-day filtering, and 1.0R BE management, these four factors create the confluence stack. The top variant (v4_combo_0056) achieved Sharpe 2.571 with 12.8% max drawdown.

A confluence scoring model assigns point scores based on measured Sharpe impact: session timing is the strongest factor, followed by combo signal synergy, inside day mode, and BE settings.

## Status / caveats

**Suspect.** This analysis predates the [[wiki/concepts/v10i-look-ahead-bug|V10i look-ahead bug]] discovery and the [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]]. Any configs using trailing stops or V10i-era alignment should be considered invalidated. Cross-reference [[wiki/concepts/invalidation-rules|invalidation rules]] before trusting any numbers.
