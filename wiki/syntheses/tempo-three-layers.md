---
created: 2026-04-11
updated: 2026-04-11
type: synthesis
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
  - raw-sources/trading/COMPREHENSIVE_MINING_REPORT.md
  - raw-sources/trading/context-engine/NQ_PLAYBOOK.md
related:
  - wiki/summaries/tempo-cluster.md
  - wiki/entities/tempo-methodology.md
  - wiki/entities/tempo-trading-system.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/tempo-v14-corrections.md
  - wiki/syntheses/bos-fvg-claim-vs-reality.md
tags: [trading, synthesis, tempo, canonical]
---

# The Tempo Three Layers

**Thesis:** Every claim, backtest number, or "validation" in the Tempo cluster needs to be placed into one of three distinct layers before it can be evaluated. Conflating the layers is the single largest source of false confidence in the project, and has caused every major reversal in Harrison's research arc so far. The layers, in order of epistemological strength:

1. **Layer 1 — Tempo the educator's canonical methodology** (reported 88% WR on 400+ live trades, but unaudited by anyone outside Tempo's Discord)
2. **Layer 2 — Harrison's mining and backtest interpretations** (deeply contaminated by look-ahead bugs, bar-sim artifacts, and signal mislabeling)
3. **Layer 3 — Current NinjaTrader implementation(s)** (v14 corrected, v24 on sim, neither yet proven live)

## Layer 1 — Canonical Tempo methodology

**What it is:** The actual rules [[tempo-methodology|Tempo teaches]], as transcribed from 281 Discord trade recap videos into `tempo_rules_v3_complete.md` (654 lines, 257k words). Tempo is a live trader running 30+ copy-traded prop firm accounts, reporting 88% WR on 400+ trades in Nov 2025 – Jan 2026, and a single-day record of $42k on 2026-01-29.

**Core rules (from Layer 1):**
- Entry: [[ifvg|IFVG]] + liquidity sweep + [[smt|SMT]] divergence, close through the gap
- Gap sizing: 8–15pt optimal on NQ, skip >30pt
- Stops: soft stop at opposite FVG edge, checked on closes; hard stop only on funded accounts away from screen
- Targets: [[dol-framework|DOL]] hierarchy (London H/L → data wicks → hourly → 50% range → PDH/PDL → weekly)
- Session management: London 100% runner to B/E at +17, NY AM 30/70, NY Lunch+PM 55/35/10
- Setup quality: 3+ confluences required on funded accounts ([[setup-quality-grading]])
- Risk: 0.4–0.6% per trade, $200–300 per account, optimized for base-hit × 30 accounts

**Epistemological status:** Strong but not independently verifiable. Tempo's reported numbers are from a self-interested source, and the setup is inherently screen-time-heavy and discretionary (soft stops, subjective reaction-quality judgment). Layer 1 is what the *methodology* actually is, not what backtests of it would say.

**What can and cannot be measured:**
- ✅ Signal mechanics can be coded (3-bar FVG detection, sweep detection, SMT via dual-feed NQ/ES)
- ✅ Session rules can be automated (time windows, TP rules)
- ❌ Reaction-quality judgment ("death candle," "strong rejection") is partly subjective
- ❌ [[setup-quality-grading|Confluence counting]] requires human-level understanding of "what counts as a major sweep"
- ❌ SMT requires dual-feed data and structural-level parity between NQ and ES (non-trivial infrastructure)

## Layer 2 — Harrison's mining and backtest interpretations

**What it is:** The research corpus that accumulated in `~/Documents/trading-system/` from roughly mid-2025 through early 2026 — `COMPREHENSIVE_MINING_REPORT.md`, `STRATEGY_MINING_REPORT.md`, `BOS_FVG_RESEARCH.md`, `NQ_PLAYBOOK.md`, `RESEARCH_JOURNAL.md`, `context-engine/MINING_ANALYSIS.md`, and the associated CSV grids in `results/`. This layer is where the *ideas* from Layer 1 were encoded into Python backtesters and swept across 312 days of Databento NQ tick data.

**What it produced:** Hundreds of labeled "strategies" with headline numbers:
- BOS_FVG: +13.37 R/day, Calmar 151 (bar-sim)
- BOS_FVG V10z 15S trailing: +1.141 avgR, Calmar 263.6 (bar-sim)
- V8g ≥ 4 MTF alignment: 61.7% → 79.6% WR OOS (look-ahead bias)
- V7/V8 phases: 100+ experiments across signal and confluence variations

**Contamination sources (this is the critical list):**
1. **[[v10i-look-ahead-bug|V10i alignment look-ahead]]** — the `tf_c.index ≤ candles_1m.index[idx]` pattern included uncompleted bars. V8 mining's 80%+ WR tiers collapsed to ~50% when fixed.
2. **[[bar-sim-trailing-bug|Bar-sim trailing stop bug]]** — trailing stops can't be simulated on 1-minute OHLC. Inflated every trailing-stop result by +0.29 R/trade. BOS_FVG's apparent +13.37 R/day collapses to +0.01 R/day on tick replay.
3. **BOS_FVG ≠ IFVG mislabeling** — see [[bos-fvg-claim-vs-reality]]. The entire mining corpus was testing a momentum-continuation strategy while calling it a liquidity-fade strategy.
4. **Phantom B/E fill bug** (`simulate_outcomes()` in micro_structure.py) — inflated BE=1.0 results for BOS_FVG, VA_fade, pre_gap_fill, vol_spike_FVG, and doji_break variants.
5. **Bad tick data** — 0.6% of Databento NQ ticks contaminated with MNQ-bleed prices; fixed with the 10,000–30,000 price filter in `micro_structure.py` and `clean_backtest.py`.
6. **DST session classification** — pre-fix scripts hardcoded EST offsets and misclassified 7 months of EDT data.
7. **Multiple-comparison artifacts** — 25+ filter combos swept, rankings declared without Bonferroni correction. `BOS_FVG_10PT_AUDIT.md` found zero filters survived p < 0.002.

**Epistemological status:** Nearly all quantitative claims in Layer 2 are suspect until re-validated at tick resolution, with multiple-comparison correction, and against the correct signal definition. The *qualitative* observations about dead strategies (london_breakout, VP_POC_retest, fib_retracement, etc.) are more reliable because "negative at every parameter" is robust to most of the above bugs.

**Which Layer 2 results are still defensible:**
- `BOS_FVG_10PT_AUDIT.md` — was tick-level from the start, correctly identifies BOS_FVG 10+pt static T3R as +0.015 avgR baseline (no edge)
- The dead-ends list — the signals that were negative across every parameter set aren't rehabilitated by fixing bugs
- The slippage, commission, and fill-rate infrastructure — bug-adjacent but the underlying numbers are fine
- `TEMPO_IFVG_RESEARCH.md` — built on the Tempo-corrected signal definition with close-through entries, survives bar-sim caveat because it uses close-based exits

**Which Layer 2 results are dead:**
- Every BOS_FVG trailing-stop headline number
- Every MTF alignment result before the V10o audit fixed the look-ahead
- Every BE=1.0 result generated through micro_structure.py before the phantom-B/E fix
- Every doji-break number
- The entire "core signal of Tempo" framing in `COMPREHENSIVE_MINING_REPORT.md` V2

## Layer 3 — Current NinjaTrader implementation

**What it is:** The live/sim execution stack on NinjaTrader, driving the actual NQ account(s). Two implementations in this layer:

- **v14** — the C# IFVG implementation audited in `TEMPO_V14_CORRECTIONS.md`. Had four critical bugs (entry sequence, stop placement, entry price, emergency stop on wrong side). Post-correction 30-day backtest: US sessions 70.5% WR at +24.5 pts/day, Calmar 5.7. **Historical reference** — not currently the active version.
- **v24 — IFVG MTF cascade** — the current active implementation on NinjaTrader sim. Referenced in Harrison's CLAUDE.md as "the ONLY strategy currently on sim." Details live in `The_Unified_Brain_Architecture.docx`. Not yet proven live.

**Epistemological status:** Layer 3 is where the rubber meets the road — real execution on a real platform with real tick data. Layer 3 runs are the only ones that can produce *verified* performance numbers, because the platform's trade management runs tick-by-tick on real fills rather than reconstructing intra-bar paths from OHLC.

**v14's disposition:** The corrected v14 is a documented reference for what went wrong and how to fix it, but is not the active version. Whatever lessons v14's debugging produced (especially about London's structural incompatibility with IFVG's 100%-runner session rule) carry forward into v24.

**v24's status:** Running on sim, insufficient forward data to call it validated. Harrison's CLAUDE.md and `The_Unified_Brain_Architecture.docx` both explicitly frame v24 as "not yet proven." The Unified Brain architecture (Nova/Sable researchers, Mem0 memory, weekly tournament judging) is built around the premise that v24 needs to prove itself before scaling.

## The canonical rule: which layer is which claim in?

When reading any performance number anywhere in the Tempo cluster, ask these three questions in order:

1. **Which layer does this claim come from?** Layer 1 (Tempo's self-report), Layer 2 (mining backtest), or Layer 3 (NT sim/live)?
2. **If Layer 1:** Is it consistent with Tempo's published numbers, or does someone have it on a different number? (The 88% WR number has been consistent across months; any claim that contradicts it should be treated as a misread.)
3. **If Layer 2:** Does it use a trailing stop? A B/E rule through `simulate_outcomes()`? An alignment filter? A BOS_FVG/IFVG conflation? If yes to any, default to suspect until tick-validated.
4. **If Layer 3:** How many days of forward data? Has it been OOS-verified?

If the layer can't be identified, the claim can't be evaluated.

## Why this synthesis matters

Harrison's `CLAUDE.md` at `~/Documents/trading-system/` now explicitly states: "NOTHING from the backtesting research is validated." That's the project-side correction. This synthesis is the vault-side version: a framework that makes the CLAUDE.md directive actionable across every Tempo-adjacent document in the corpus. When a future session reads `NQ_PLAYBOOK.md` and sees +1.141 avgR / Calmar 263.6 / 13/13 months positive, the three-layer framework tells you immediately: Layer 2, trailing-stop-based, contaminated by [[bar-sim-trailing-bug]], not defensible. When the same future session reads `tempo_rules_v3_complete.md`'s 88% WR number, the framework says: Layer 1, unaudited but internally consistent, treat as methodology not as evidence.

## See also

- [[tempo-cluster]] — the raw-sources summary that this synthesis organizes
- [[bos-fvg-claim-vs-reality]] — deep dive on the specific BOS_FVG vs IFVG conflation
- [[bar-sim-trailing-bug]] — structural explanation of Layer 2's backtesting bug
- [[v10i-look-ahead-bug]] — Layer 2's earlier alignment look-ahead
- [[tempo-v14-corrections]] — Layer 3's historical bug inventory
