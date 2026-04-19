---
created: 2026-04-11
updated: 2026-04-19
type: concept
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/tempo-cluster.md
tags: [trading, signal, tempo, confluence, canonical]
---

# SMT — Smart Money Technique

The **primary confluence filter** in the canonical [[tempo-methodology|Tempo methodology]] since November 2025. Defined as an intermarket divergence between NQ and ES at the same key liquidity level.

## Definition (Tempo's usage)

**SMT** = **S**mart **M**oney **T**echnique — *not* "Supply Manipulation Test" (this mistranscription is explicitly corrected in `TEMPO_PROJECT_STATE.md`).

Operationally: build the same set of structural key levels (PDH/PDL, session H/L, 1H swing points, prior day range, weekly H/L) on both NQ and ES. When NQ sweeps one of its levels but ES does not sweep the equivalent structural level in the same window — or vice versa — you have **SMT divergence**. The sweep on the non-divergent side is a liquidity trap, and the reversal that follows is the trade.

## Why it's primary since November 2025

Tempo explicitly elevated SMT to the top of the confluence stack in late 2025. `tempo_rules_v3_complete.md` is emphatic: without SMT, reported WR drops from ~80% to ~60–65%; with SMT + [[ifvg|IFVG]] + liquidity sweep, you're "in the money most of the time."

For funded accounts, SMT is required as part of the 3+ confluence minimum established in the January 2026 rule update (see [[setup-quality-grading]]).

## The subtlety: SMT alone is not the edge

Harrison's tick-level IFVG research (`TEMPO_IFVG_RESEARCH.md`) hit a surprising result: **SMT alone does not improve backtested WR**. Baseline 64.4% → with SMT alone 63.3%. The edge only materializes when SMT is paired with a **reaction-quality filter**:

- Death candle (single 3+ point candle through the gap), OR
- Three consecutive directional candles after the sweep

With SMT + reaction quality, the sample jumps to **83% WR**. The working hypothesis is that SMT identifies reversal *candidates*, but reaction quality is what confirms the reversal is actually happening. You need both. This is one of the few places where Harrison's backtests and Tempo's teaching converge cleanly — the 83% WR number lines up with Tempo's self-reported 80%+ for A+ setups.

## How to measure (from `TEMPO_IFVG_RESEARCH.md`)

1. Maintain two parallel structure trees — NQ and ES — with the same rule set.
2. At the moment of a candidate sweep on NQ, check ES's price action in the same window.
3. "Did ES sweep the equivalent level?"
   - If YES on both → no SMT → pass (or demote setup quality)
   - If NO on ES → SMT divergence confirmed → proceed to reaction check

The equivalent level on ES is *structural*, not price-identical. ES trades at a different nominal level than NQ, so the comparison is "did ES break the 1H swing low" / "did ES sweep the session low," not "is ES at price X."

## Cross-cluster notes

- SMT is not mentioned in Harrison's mining research journal or `NQ_PLAYBOOK.md` — those docs are mining-era and predate the SMT elevation. Absence of SMT from the mining corpus is one of the structural reasons why mining-era BOS_FVG ≠ canonical IFVG (see [[bos-fvg-claim-vs-reality]]).
- SMT requires **dual-feed market data** (simultaneous NQ + ES tick). Any backtester running on a single instrument cannot test SMT. This is a non-trivial infrastructure requirement for re-testing IFVG at the A+ tier.
- SMT is a *structural* filter, not a *temporal* one, which means it should not be affected by the [[bar-sim-trailing-bug|bar-sim trailing bug]] that killed BOS_FVG. Filters that depend on intra-bar price sequencing are the suspect kind; SMT only requires completed-bar comparisons.

## Intraday SMT Testing (Apr 2026)

Tested intraday SMT divergence on 15m and 1H bars using 280 days of NQ + ES tick data as an overlay filter on [[lumi-overlay-research-2026-04-19|Lumi trades]]:

- **1H SMT**: Zero divergences detected across 280 days. The swing lookback of 3 bars is too strict — intraday swings form and resolve within a single session, so the pattern rarely completes.
- **15m SMT**: 222 signals across 280 days. Mild directional value (aligned 55.3% WR vs against 50.5% WR) but not strong enough to use as a standalone filter.

**Takeaway**: Intraday SMT in its current implementation (swing-based) does not produce enough signal for filtering. The concept remains valid at the daily/multi-day level where swing structure has time to develop. A possible improvement would be using a smaller lookback (2 bars instead of 3) or using price-level divergence instead of swing-structure divergence.

Engine: `trading-system/context-engine/features/smt.py` + `strategies/tempo/scripts/lumi_overlay_research.py`

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/BACKTEST_ISSUES_LOG|BACKTEST ISSUES LOG]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/CONTEXT|CONTEXT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS|IFVG AUDIT RESULTS]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS_V2|IFVG AUDIT RESULTS V2]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT|IFVG OPTIMIZATION REPORT]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_from_videos|tempo rules from videos]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_summary|tempo rules summary]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/TEMPO_RULES_IMPLEMENTATION|TEMPO RULES IMPLEMENTATION]]
