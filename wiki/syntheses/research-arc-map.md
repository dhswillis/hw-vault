---
created: 2026-04-11
updated: 2026-04-11
type: synthesis
sources:
  - raw-sources/trading/RESEARCH_JOURNAL.md
  - raw-sources/trading/context-engine/NQ_PLAYBOOK.md
  - raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18.md
  - raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md
  - raw-sources/trading/audits/V8_Full_Audit_Report.md
  - raw-sources/trading/audits/V9_Full_Audit_Report.md
  - ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
  - raw-sources/trading/sf-portfolio/FINAL_PORTFOLIO_SPEC.md
related:
  - wiki/syntheses/tempo-three-layers.md
  - wiki/syntheses/bos-fvg-claim-vs-reality.md
  - wiki/summaries/research-journal.md
  - wiki/summaries/nq-playbook.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/tempo-v14-corrections.md
  - wiki/summaries/unified-brain-architecture.md
tags: [trading, synthesis, research-arc, canonical]
---

# Tempo Research Arc — Master Map

**Purpose:** A single-page timeline of the research, its bugs, its corrections, and its current state. Every other document in the trading section of this vault is a node; this synthesis is the graph that connects them. If you are returning to the project after a gap, read this first, then [[tempo-three-layers]], then whichever summary is relevant to your current question.

## The five phases

The research arc has moved through five distinct phases, each ending in a crisis that reset the project's understanding of what was "validated":

### Phase 1 — Pre-V10 mining (through early 2026)

**What was happening:** Mining tick data for edges. Signal families: BOS_FVG (momentum continuation), sweep-reversal family, Fib retracements, VWAP patterns, doji breaks, engulfing family. Core framework: "find a signal → stack confluence filters → declare validated."

**Key outputs:**
- [[comprehensive-mining-report-v2]] (V2 mining, 34,323 trades across 258 days)
- [[strategy-mining-report-312d]] (V1 mining, 312 days, 8,358 trades)
- [[mining-analysis]] (V7aa_b cross-tab of 2,600 combos)
- `V8_Full_Audit_Report.md`, `V9_Full_Audit_Report.md` (phase audits)

**Crisis (2026-02-18):** The first foundation audit ([[look-ahead-audit]]) found swing-confirmation and footprint-absorption look-ahead bugs. Partially fixed, but most V8/V9 results were still intact because the alignment-layer bug was still hiding.

### Phase 2 — V8/V9 alignment peak (Feb–early Mar 2026)

**What was happening:** MTF alignment confluence scoring, V8g ≥ N tiers. Apparent 80–98% WR tiers. Monte Carlo walk-forward "validated" the alignment edge. This was the peak of the "we have a massive edge" era.

**Key outputs:**
- Flashy headline numbers: V8g ≥ 4: 61.7% → 79.6% WR OOS, "sniper tier" 90%+ WR
- [[v8-v9-audit-reports|V8 and V9 audit reports]]
- NQ_Condition_Matrix.xlsx (Excel grid of signal × session × alignment × DOW)

**Crisis (V10o discovery):** The MTF alignment check was using bars whose *start* time was before the entry timestamp but whose *close* was in the future. For a 1H bar starting at 10:00 with entry at 10:03, the sim was effectively reading a close 57 minutes in the future. See [[v10i-look-ahead-bug]] for the full bug. All V8/V9 alignment-based results — the entire 80%+ WR era — collapsed.

### Phase 3 — V10 cleanup (early–mid 2026)

**What was happening:** Accepting that alignment-based edge was fake and looking for something that survived the clean rerun. V10p tested 5 clean alignment methods; all dead. V10r found BOS_FVG as the only single-signal survivor at 62.6% WR, +0.460 avgR with native swing targets. V10t walk-forward introduced fixed-R targets and caught the phantom B/E bug in `simulate_outcomes()`. V10v/V10u introduced trailing stops and "rescued" several signals.

**Key outputs:**
- [[research-journal|Research Journal]] — narrative doc for the V10 cleanup
- [[nq-playbook|NQ Playbook]] — flagship consolidation
- [[bos-fvg-10pt-audit|BOS_FVG 10+pt Audit]] — the one tick-level audit that was done correctly

**Interim peak:** V10z conservative sim (fixed intra-bar ordering) + 15-second trailing → BOS_FVG at **+1.141 avgR, Calmar 263.6**, 4/4 walk-forward OOS positive. The project treated this as the post-bug-fix validated number. BOS_FVG became "the core signal of the Tempo system."

**Parallel cleanup (SF portfolio):** An independent research track ([[sf-portfolio-cluster]]) was running on sweep-fail signals with fixed targets, built its own 8-bug fix list, and produced a +2.26 to +3.34 R/day portfolio. This track is structurally cleaner because fixed-target exits are bar-sim-safe.

### Phase 4 — The BOS_FVG collapse (2026-03-08 → 2026-04-10)

**What was happening:** A second round of audits on the V10z "validated" numbers.

**Key outputs:**
- [[bos-fvg-10pt-audit|BOS_FVG 10+pt Audit]] (2026-03-08) — tick-level audit of the static T3R variant. Found +0.015 avgR baseline, nothing survives multiple-comparison correction. Early warning that bar sim was inflating things.
- [[tempo-v14-corrections|TEMPO V14 Corrections]] (2026-03-20) — discovered that the C# NinjaTrader implementation of [[ifvg]] was doing a *different strategy* than Tempo actually teaches. Four structural bugs. Post-correction: 70.5% WR US sessions.
- [[tempo-ifvg-research|Tempo IFVG Research]] (2026-03-08) — implemented the *correct* IFVG signal at tick level. 74.7% WR baseline, 83% WR with SMT + reaction quality.
- [[bos-fvg-failure-consolidated|BOS_FVG Failure Consolidated]] (2026-04-10) — the final nail. Paired bar-vs-tick replay on 60 sampled days. Bar sim: +0.359 avgR. Tick sim: +0.001 avgR. The BOS_FVG trailing "edge" is a bar-reconstruction artifact.

**Crisis (2026-04-10):** The V10z trailing numbers that "validated" BOS_FVG as the core signal were a [[bar-sim-trailing-bug|structural bar-simulation bug]]. Trailing stops cannot be backtested on OHLC bars, period. The interim peak from Phase 3 dissolves. The entire "BOS_FVG is the Tempo core signal" framing is wrong on two separate levels — the mechanic is wrong *and* it's not even Tempo's strategy. See [[bos-fvg-claim-vs-reality]].

### Phase 5 — Unified Brain pivot (2026-04+)

**What is happening now:** Acknowledgment that mining-as-validation has failed. Strategic pivot documented in [[unified-brain-architecture|The Unified Brain Architecture]]: stop mining, start reasoning about market regime and methodology with two local AI researchers ([[nova-sable-brains|Nova and Sable]], DeepSeek R1-Distill-Qwen-14B via Ollama). Hard-coded Risk Officer. Weekly tournament with external AI advisor judging. Strict phase progression: research → paper → micro live → scale.

**Current active strategy:** v24 IFVG MTF cascade on NinjaTrader sim. **Not yet proven.**

**What's baked into Mem0 seed:** Dead strategies (london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion). Anti-stupidity rules (never test MTF alignment, never test B/E stops, always walk-forward with 4 quarters positive OOS, always apply realistic costs).

## The three parallel research tracks

At any given point during Phases 1–4, the project had **three tracks running simultaneously**:

### Track A — BOS_FVG / V10 mining (primary)

- Python backtesters (clean_backtest.py, bos_fvg_full_audit.py, /tmp/mine_*.py)
- Signal family: BOS + FVG fill + trailing stop
- Headline: V10z +1.141 avgR / Calmar 263.6 (invalidated)
- **Verdict as of 2026-04-10: DEAD.** Every trailing-stop number is a bar-sim artifact.

### Track B — SF portfolio (parallel, cleaner)

- Python backtesters (different codebase, /tmp/all_signals_260d.py etc.)
- Signal family: sweep-fail + VWAP + AMD + engulfing, fixed targets
- Headline: +3.34 R/day across 260 days ([[sf-portfolio-cluster]])
- **Verdict: probably defensible but not independently audited.** Structural reasons to trust it (fixed targets, not trailing) are solid. Cross-validation at tick level not yet done.

### Track C — Tempo IFVG (from Tempo's rules, not mining)

- Python backtester (`TEMPO_IFVG_RESEARCH.md`) + NinjaTrader v14/v24 implementations
- Signal family: IFVG close-through + SMT + reaction quality
- Headline: 74.7% WR baseline, 83% WR with full filter stack ([[tempo-ifvg-research]])
- v14 post-corrections: 70.5% WR US sessions ([[tempo-v14-corrections]])
- **Verdict: the only track with a coherent Layer 1 → Layer 2 → Layer 3 story.** Tempo's canonical methodology (Layer 1), Harrison's correctly-implemented backtest (Layer 2 clean), v14 corrected + v24 on sim (Layer 3 active). Still not proven live.

## The bug taxonomy

Across all phases, five distinct classes of bug have contaminated Layer 2 results:

| Bug | Layer | Affects | Phase discovered |
|---|---|---|---|
| Swing confirmation timestamp | Signal layer | V7/V8 early scripts | Phase 1 ([[look-ahead-audit]]) |
| Footprint absorption look-ahead | Feature layer | V8 absorption filters | Phase 1 ([[look-ahead-audit]]) |
| [[v10i-look-ahead-bug|MTF alignment look-ahead]] | Filter layer | All V8/V9 alignment results | Phase 3 (V10o) |
| Phantom B/E bug in `simulate_outcomes()` | Exit layer | BOS_FVG, VA_fade, pre_gap_fill, doji break, vol_spike_FVG with BE=1.0 | Phase 3 (V10t) |
| [[bar-sim-trailing-bug|Bar-sim trailing path reconstruction]] | Exit layer | All trailing-stop backtests on OHLC data | Phase 4 (2026-04-10) |

Two additional data-quality bugs:

| Bug | Layer | Affects | Phase discovered |
|---|---|---|---|
| Databento MNQ bleed contamination | Data layer | Doji break scripts (scripts without symbol filter) | Phase 3 |
| DST session classification | Feature layer | Hardcoded EST offsets, misclassified 7 months of EDT data | Phase 3 (BOS_FVG 10+pt audit) |

The pattern: **each phase invalidates the previous phase's headline numbers.** Phase 1 audit revealed Phase 1 signal-layer bugs. V10o revealed Phase 2's alignment bug. Phase 4's tick audit revealed Phase 3's trailing bug. There has not been a phase whose results were still defensible in the following phase — the project has been in a state of perpetual "the most recent bug hasn't been found yet" since the start.

## The structural insight from Phase 5

The Unified Brain pivot is a direct response to this pattern. Rather than betting that Phase 5 will finally produce "the clean mining results," the project is betting that **mining-as-validation is structurally unreliable** for this dataset and this methodology. The new bet is on **reasoning + forward phase gating + external judging**, with backtests relegated to a hypothesis-generation role rather than a validation role. Whether this works is the open question; but the decision to stop treating successive mining waves as validation is itself informed by the arc documented here.

## The canonical "what to trust" rule

When reading any performance claim anywhere in the Tempo cluster, apply this check in order:

1. **Is this from Layer 1 ([[tempo-methodology|Tempo's canonical teaching]])?** Trust methodology, don't trust specific numbers (self-reported).
2. **Is this from Layer 2 (mining/backtest)?** Default to **suspect**. The claim is defensible only if:
   - It's tick-level (not bar-level trailing)
   - It has Bonferroni correction applied
   - It has walk-forward OOS validation
   - It's been cross-validated on a second engine
   - Its signal definition matches Layer 1 (not the BOS_FVG mining interpretation)
3. **Is this from Layer 3 (NinjaTrader sim/live)?** Only trust numbers with documented forward days and engine-native execution. Sim ≠ live; 30 days ≠ proven.

If a claim can't pass this check, cite it as *methodology* ("what was tested") not as *evidence* ("what works").

## Cross-references (the full map)

- [[tempo-three-layers]] — the framework underlying the "what to trust" rule
- [[bos-fvg-claim-vs-reality]] — deep dive on the BOS_FVG ≠ IFVG conflation
- [[bar-sim-trailing-bug]] — the bug that ended Phase 4
- [[v10i-look-ahead-bug]] — the bug that ended Phase 2
- [[research-journal]], [[nq-playbook]] — Layer 2 narrative and consolidation
- [[sf-portfolio-cluster]] — the parallel Track B
- [[tempo-ifvg-research]], [[tempo-v14-corrections]] — Track C
- [[unified-brain-architecture]] — Phase 5 direction
- [[look-ahead-audit]], [[v8-v9-audit-reports]], [[bos-fvg-10pt-audit]] — the audit history
- [[tempo-cluster]] — the master cluster summary for all Tempo raw-sources
