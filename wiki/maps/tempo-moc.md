---
created: 2026-04-11
updated: 2026-04-11
type: moc
topic: Tempo Strategy Research Arc
tags: [moc, tempo, trading, canonical]
related:
  - wiki/maps/audit-history-moc.md
  - wiki/maps/bos-fvg-saga-moc.md
  - wiki/syntheses/tempo-three-layers.md
  - wiki/syntheses/research-arc-map.md
---

# Tempo Strategy — Map of Content

> The full research arc from Tempo's 281 trade recaps through mining, implementation, audits, and the current Unified Brain pivot. Organized by the [[tempo-three-layers|three-layer framework]]: what Tempo teaches (Layer 1), what Harrison's mining found (Layer 2), and what's actually running (Layer 3).

## Orientation

Tempo is an NQ futures trading methodology created by educator "Tempo" (@tony31 on Discord). Harrison spent 12+ months extracting, mining, backtesting, and implementing it. The headline finding: Tempo's own live results (88% WR) are real, but **Harrison's backtest interpretations consistently over-estimated edge** due to look-ahead bias, phantom B/E fills, and bar-sim trailing bugs. The project has now pivoted to the [[unified-brain-architecture|Unified Brain]] — two AI researchers (Nova + Sable) that inherit the full dead-end list and are tasked with finding a strategy that actually works at tick level.

## Layer 1 — What Tempo Actually Teaches (canonical)

- [[wiki/summaries/tempo-rules-v3]] — **Start here.** 281 transcriptions, 257k words. The canonical extraction of Tempo's methodology from Oct 2024 to Jan 2026. 88% WR reported.
- [[wiki/concepts/ifvg]] — The primary entry signal: Inverted Fair Value Gap. FVG forms BEFORE sweep → sweep catalyzes reversal → close back through the FVG → entry at FVG edge.
- [[wiki/concepts/smt]] — Smart Money Technique (NQ vs ES divergence). **Primary confluence** since Nov 2025. Without SMT, WR drops from ~80% to ~60-65%.
- [[wiki/concepts/dol-framework]] — Draw on Liquidity. Where price is heading. Hierarchy: London H/L → data wicks → hourly → 50% range → PDH/PDL.
- [[wiki/concepts/bpr]] — Balance Price Range. Overlap of opposite-direction FVGs. "FVG on steroids."
- [[wiki/concepts/setup-quality-grading]] — A+/A/B+/B tier system. 7+ confluences for A+; minimum 3+ for funded accounts (Jan 2026 rule change).
- [[wiki/entities/tempo-methodology]] — Tempo the educator: profile, trading style, the copy-trading operation (30+ accounts, $42k single day).
- [[wiki/summaries/tempo-project-state]] — The master "read this first" doc. File inventory, standing corrections, extraction pipeline status.

## Layer 2 — Harrison's Mining / Backtests (mostly suspect)

### The BOS_FVG saga (dead — see [[maps/bos-fvg-saga-moc]])
- [[wiki/concepts/bos-fvg]] — **DEAD.** The mining-era interpretation of Tempo's IFVG. Bar-sim trailing inflated all prior numbers by +0.29 R/trade.
- [[wiki/summaries/comprehensive-mining-report-v2]] — V2 mining: 34,323 trades, look-ahead fixed but bar-sim still active. 63.5% WR headline now invalidated.
- [[wiki/summaries/strategy-mining-report-312d]] — V1 mining: 312d, 8,358 trades. 97.9% WR "sniper tiers" were V10i look-ahead.
- [[wiki/summaries/nq-playbook]] — V10 flagship findings doc. +1.141 avgR headline is bar-sim artifact.
- [[wiki/summaries/research-journal]] — V7→V11 narrative journal. Historical context for the research arc.

### Wick-fade family (tick-validated — rare Layer 2 survivor)
- [[wiki/concepts/wick-fade]] — **TICK-VALIDATED.** 5m and 15s fade variants. 10/10 walk-forward OOS positive. Fixed stop + target, bar-sim-safe.
- [[wiki/summaries/wickfade-complete]] — 5-minute WickFade: +77-94 R/day, Calmar 2.12, **10/10 WF OOS positive**. Strongest anti-overfit evidence in the corpus. **Also flagged the bar-sim trailing bug a month before the BOS_FVG audit.**
- [[wiki/summaries/15s-wick-fade]] — 15-second wick fade: 43% WR, $30.89/trade. Flip variant dead (0/216 configs).

### SF Portfolio (tick-level confirmed — fat-tailed)
- [[wiki/summaries/sf-portfolio-cluster]] — SF portfolio: +3.34 R/day on 260d. Tick-level simulator confirmed 2026-04-11. RUNNER leg is 16% WR fat-tail — needs full 260d for edge to materialize.
- [[wiki/entities/ninjatrader-v5]] — The V5 NinjaTrader strategies. 7 bugs found in the V5 bug audit.
- [[wiki/summaries/v5-strategy-bug-audit]] — VWAP double-counting + 6 other bugs. ES VWPM 44% live vs 66% backtest.

### Sweep cluster (mostly dead)
- [[wiki/summaries/sweep-cluster]] — Sweep-and-fade experimental cluster. Most variants dead or marginal. sweep_reversal is one of the 7 dead signals.

### IFVG research + Tempo IFVG (Layer 2 cleanest)
- [[wiki/summaries/tempo-ifvg-research]] — Tick-level IFVG research. 74.7% WR baseline, 83% with SMT + reaction quality. **Cleanest Layer 2 doc** — uses close-through soft stops (bar-sim-safe by construction).
- [[wiki/summaries/lumi-strategy-spec]] — Lumi engine (@LumiTraders). V14 broken (20% WR); **V15 rewrite fixed it** — combined IFVG+Lumi: +19.8 pts/day, Calmar 23.1.
- [[wiki/summaries/tempo-portfolio-v15]] — V15 tick-level portfolio audit (IFVG + Lumi). The definitive combined result.

### Mining machinery + audits
- [[wiki/summaries/mining-analysis]] — V7aa_b cross-tab of 2,600 combos. Suspect (V10i alignment).
- [[wiki/summaries/look-ahead-audit]] — Foundation audit, Feb 2026. Found V10i look-ahead.
- [[wiki/summaries/v8-v9-audit-reports]] — V8/V9 phase audits. Suspect (pre-V10o).
- [[wiki/summaries/bos-fvg-failure-consolidated]] — **2026-04-10.** The canonical failure report that killed BOS_FVG.
- [[wiki/summaries/bos-fvg-10pt-audit]] — 2026-03-08. Tick-level static audit. +0.015 avgR baseline.

### Orderflow methodology (reference material)
- [[wiki/summaries/transcript-mining-findings]] — Orderflow synthesis from 7 educators, 30 transcripts. CLC framework, volume cluster pullbacks, POC reversal.

## Layer 3 — Current Implementation

- [[wiki/summaries/tempo-v14-corrections]] — 4 bugs in the v14 C# code. Correct signal sequence (FVG before sweep, not after). London 0% WR structural issue. US sessions 70.5% WR post-correction. **v24 IFVG MTF cascade is the ONLY strategy currently on NinjaTrader sim.**
- [[wiki/summaries/unified-brain-architecture]] — **THE CURRENT PROJECT STATE.** Nova + Sable AI researchers with Mem0, weekly tournaments, hard-coded Risk Officer, live YouTube show pipeline. "BOS_FVG does not work. v24 is the only thing on sim. The brain's job is to FIND a strategy, not execute one."
- [[wiki/entities/nova-sable-brains]] — The two DeepSeek R1-14B researchers. Nova = methodical. Sable = aggressive.

## Context Engine (supporting infrastructure)
- [[wiki/summaries/tempo-context-engine-spec]] — Bayesian session classifier (68.6% accuracy, 46-feature state vector). Already built and working. Feeds the brains.
- [[wiki/concepts/session-type-taxonomy]] — TREND / RANGE / EXPANSION / COMPRESSION / NEWS-SHOCK classification.
- [[wiki/concepts/bayesian-belief-engine]] — The multinomial naive Bayes classifier core.
- [[wiki/concepts/volume-profile]] — POC, VA, HVN/LVN + footprint analysis. Layer 2/3 features.
- [[wiki/summaries/nautilus-spec]] — NautilusTrader spec. BOS_FVG at +0.286 avgR — the early warning sign.

## The ruleset (what we learned)
- [[wiki/concepts/invalidation-rules]] — **The 15 rules.** Every bug, every contamination, every false positive. This is the "never again" ledger that the Unified Brain inherits.
- [[wiki/maps/audit-history-moc]] — Every audit, sorted by outcome, with the invalidation chain.
- [[wiki/syntheses/bos-fvg-claim-vs-reality]] — Why "BOS_FVG is the core signal of Tempo" was a two-layer mistake.

## The evolution arc

```
Oct 2024    Tempo rules v1 extracted (29 transcriptions)
Feb 2026    V2 extraction (90 transcriptions), V3 (281 transcriptions)
Feb 2026    V10i look-ahead audit → MTF alignment DEAD
Feb 2026    V2 comprehensive mining → BOS_FVG "63.5% WR" (still bar-sim)
Mar 2026    10pt tick audit → static BOS_FVG has +0.015 avgR (zero)
Mar 2026    WickFade complete findings → FLAGS bar-sim trailing bug
Mar 2026    V14 corrections → signal sequence was WRONG in the code
Mar 2026    V5 bug audit → VWAP double-counting in live NinjaTrader
Apr 2026    BOS_FVG failure consolidated → bar-sim trailing kills all trailing results
Apr 2026    Unified Brain pivot → Nova + Sable AI researchers
Apr 2026    SF portfolio tick-level confirmed → RUNNER fat-tail, needs 260d
Apr 2026    Wick-fade tick-validated → 10/10 WF OOS, bar-sim-safe
Apr 2026    HW vault created → everything ingested, rules codified
```

## Open questions

- Does v24 IFVG MTF cascade have edge on NT sim? Collecting data.
- Are vol_spike_FVG, ORB_breakout, pre_gap_fill worth re-testing at tick resolution?
- Can the wick-fade family compound with IFVG for a diversified portfolio?
- Will Nova or Sable discover something entirely new from the tick data?

## Related MOCs

- [[maps/audit-history-moc]] — the bug-finding chronology
- [[maps/bos-fvg-saga-moc]] — the BOS_FVG invalidation case study
