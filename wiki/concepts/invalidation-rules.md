---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/audits/BOS_FVG_FAILURE_CONSOLIDATED.md
  - raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md
  - raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18.md
  - raw-sources/trading/audits/V8_Full_Audit_Report.md
  - raw-sources/trading/audits/V9_Full_Audit_Report.md
  - raw-sources/trading/audits/BACKTEST_INTEGRITY_AUDIT_V2.docx
  - raw-sources/trading/audits/BOS_FVG_AUDIT_FINDINGS.md
related:
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/bos-fvg-10pt-audit.md
  - wiki/summaries/audits-cluster.md
  - wiki/maps/audit-history-moc.md
tags: [trading, methodology, canonical, rules, do-not-forget]
---

# Invalidation Rules — The "Do Not Forget" Ledger

**Purpose:** A single ruleset distilled from every bug, contamination, and false-positive found across the V7-V11 trading research. Every rule here was paid for in months of research time. Do not re-test these dead ends. Do not re-cite the contaminated numbers. Do not bypass the rule without new evidence at least as rigorous as the audit that produced it.

This is a **canonical** page. Future sessions should read it first when making any claim about BOS_FVG, trailing stops, MTF alignment, V8/V9/V10 results, or the phantom B/E mechanism. See also `~/.claude/projects/-Users-harrisonwillis/memory/bar_sim_trailing_bug_rule.md` for the auto-memory version.

## Rule 1 — Do not trail on bars

**Rule:** Do not backtest a trailing stop on OHLC bars. Either run it at tick resolution, or replace the trailing exit with a fixed R target or a close-through soft stop.

**Why:** A trailing stop's exit price depends on the *sequence* in which the underlying touches a bar's extremes, not just the extremes. OHLC discards the sequence. The most natural simulator ordering systematically inflates the winner by locking in high-water marks that, in the real tick path, would have been unwound within the same bar.

**Measured magnitude:** On BOS_FVG trailing (0.5R trigger / 0.1R buffer), 60-day Databento NQ sample: bar-sim +0.359 avgR vs tick-replay +0.001 avgR. **+0.29 R/trade inflation.** The entire reported "edge" was the bug.

**What it invalidated:**
- `clean_backtest.py` outputs — BOS_FVG +0.895 avgR, Calmar 237
- V10z 15S trailing — +1.141 avgR, Calmar 263.6
- V10z 4-signal portfolio — +0.880 avgR, Calmar 94.4
- `BOS_FVG_RESEARCH.md` headline — +0.354 avgR, +13.37 R/d
- `COMPREHENSIVE_MINING_REPORT_V2` BOS_FVG 63.5% WR — V2 fixed alignment look-ahead but still ran bar-sim
- `nq-playbook` flagship numbers
- Every `/tmp/mine_*.py` and `/tmp/ifvg_optimize_v*.py` that trails on bars

**What is bar-sim-safe:**
- Fixed R targets (static T3R, T5R) — unambiguous on bars
- Close-through soft stops (IFVG style) — exit on candle CLOSE past stop, not wick touch
- NinjaTrader engine-native execution — the platform runs tick-level internally; `SweepBreakv17.cs` and v24 IFVG MTF cascade are unaffected

**Audit of record:** [[bos-fvg-failure-consolidated]] (2026-04-10), [[bos-fvg-10pt-audit]] (2026-03-08).

**Concept page:** [[bar-sim-trailing-bug]].

## Rule 2 — Do not consult unclosed higher-TF bars for alignment

**Rule:** Multi-timeframe alignment must only use bars whose *end* time ≤ entry time. Never use a bar whose start time ≤ entry time — that's the currently-open HTF bar and its `close` field contains future data.

**Why:** The V10i `get_alignment()` function filtered `tf_c.index <= candles_1m.index[entry_idx]` — selecting the HTF bar whose START ≤ entry. A 5-minute bar timestamped 10:00 covers 10:00–10:05; if entry is at 10:02, this bar is selected even though it hasn't closed. The `close` field is the 10:05 price — three minutes in the future. For a 1-hour bar and a 10:03 entry, you're using a close that's 57 minutes in the future.

**Measured magnitude:** A config that showed 91.1% WR on 2,035 trades dropped to 37.6% WR on 340 trades after the fix. 0 of 13 green months after. **The entire edge was look-ahead.**

**The fix:** `mask = tf_c.index + TF_DURATION[tf_label] <= entry_time` — only bars that have fully closed strictly before the entry timestamp.

**What it invalidated:**
- All V10a–V10n alignment research
- The V1 312-day mining report's "sniper tiers" (97.9% / 98.8% / 90.6% WR)
- Any strategy/config page that cites > 90% WR on n ≥ 50 without a tick-level re-validation

**Audit of record:** [[look-ahead-audit|LOOK_AHEAD_AUDIT_2026-02-18.md]].

**Concept page:** [[v10i-look-ahead-bug]].

## Rule 3 — Break-even stops kill Model 2 edge

**Rule:** Do not test break-even stops on the Tempo Model 2 / sweep-fail family. Confirmed dead across 11 configurations and a full year of data.

**Why:** BE stops convert winners into near-zero breakevens while leaving losers at full stop distance. For signals with high variance in max-favorable excursion, this systematically destroys the asymmetric payoff that the trailing stop was supposed to capture. The BE "improvement" shows up in bar-sim results because the bar-sim path-reconstruction bug (see Rule 1) masks the destruction.

**Audit of record:** `results/BOS_FVG_RESEARCH.md` BE analysis, also confirmed in the V10t walk-forward results where every BE=1.0 variant is tagged suspect.

**Corollary:** Signals tagged `BE=1.0` in the V10t results (BOS_FVG T5.0R, VA_fade T2.0R, pre_gap_fill T5.0R, vol_spike_FVG T5.0R) are double-suspect: once for the BE bug and once for the bar-sim trailing bug. Do not cite their R numbers.

**Concept page:** [[be-trail-mechanism]].

## Rule 4 — Phantom B/E fills: `micro_structure.py` bug

**Rule:** Any `simulate_outcomes()` call that uses `BE=1.0` is contaminated by the phantom B/E fill bug in `micro_structure.py`. The simulator fills break-even exits that never actually executed.

**Why:** The bug converts losers into false breakevens. Reported WR is inflated; reported avgR looks better than reality.

**What it invalidated:**
- V8 anything with B/E (the "V8g numpy bool bug" compounds: JSON trades have `rej_delta_aligned`/`delta_divergent` as string `"True"`/`"False"`, numpy parses as bool differently)
- Doji break all sessions — clean sim shows -0.171 avgR everywhere; the original edge was the phantom B/E
- V10t variants with `BE=1.0`

**Audit of record:** V8 Full Audit Report; confirmed in the 2026-03-07 full audit in MEMORY.md "What Actually Works" section.

## Rule 5 — P/D (premium/discount) must use up-to-signal-time data only

**Rule:** Premium/Discount levels must be computed from data available strictly before the signal timestamp. Do not use full-day high/low — that's future data.

**Why:** The `BOS_FVG_10PT_AUDIT` found the original P/D filter used full-day H/L (future data) and produced +0.241 avgR on "aligned" trades. Rolling P/D (up-to-signal-time) produces +0.031 avgR — **zero separation**. The entire +0.267 avgR "edge" was future data.

**What it invalidated:**
- Every BOS_FVG variant that cites P/D alignment as a filter
- Any mining grid result with a `pd_aligned` column using the broken P/D method
- Analogous filters: VWAP-based P/D (zero separation), opening-range P/D (zero separation). All dead when clean.

**Audit of record:** [[bos-fvg-10pt-audit]] §1.

## Rule 6 — Walk-forward pass does not rescue a contaminated simulator

**Rule:** 4/4 OOS quarters positive is meaningless if the underlying simulator is wrong. A bug that inflates systematically will inflate across time, producing a confident "stable" WF result that is still zero edge.

**Why:** The V10z numbers showed 4/4 WF positive at +1.141 avgR. They are invalidated by the 2026-04-10 tick replay at +0.001 avgR. The WF was measuring bar-sim consistency, not edge.

**Corollary:** Always cross-validate a trailing-stop WF result against tick replay before citing the number. If you can't run tick replay, use a fixed R target — it's bar-sim-safe.

## Rule 7 — Multiple-comparison correction is non-negotiable

**Rule:** Any mining grid over more than ~10 filter combinations must be reported with Bonferroni-corrected p-values. A 5% false positive rate on 25 combos yields ~1.25 expected false "edges."

**Why:** The 10+pt BOS_FVG audit tested 25 filter combinations. **Every 95% bootstrap CI spanned zero.** No filter — not stacked FVG, not "no opposing," not PD variants, not size buckets — survives Bonferroni (threshold p < 0.002). The best uncorrected p-value was 0.027, which fails.

**What it invalidated:**
- Any "best filter" claim sourced from mining grid sweeps without multiple-comparison correction
- The V1 mining report's "Tier S sniper" configurations (which also have the V10i look-ahead bug)

**Audit of record:** [[bos-fvg-10pt-audit]] §6.

## Rule 8 — Outlier months are not edge

**Rule:** A rolling walk-forward result driven by one or two outlier months is not stable. Remove the biggest contributor month and re-check — if the result goes negative, the cumulative profit is concentration, not edge.

**Why:** The 10+pt BOS_FVG audit found Feb 2026 contributed +0.983 avgR to the baseline and +1.549 to the "no_stacked" variant. Without Feb 2026, OOS was near zero. The positive rolling WF was one month propping up nine flat months.

**Audit of record:** [[bos-fvg-10pt-audit]] §5.

## Rule 9 — NinjaTrader `OnEachTick` bar-dependent accumulators need bar gating

**Rule:** Any accumulator that should update once per bar (VWAP, running delta, candle-close indicators) in a NinjaTrader strategy using `Calculate = Calculate.OnEachTick` must be guarded with `if (CurrentBar != lastXBar)`. Otherwise the accumulator fires ~50-200× per bar in live mode but only once per bar in backtest.

**Why:** The V5 SF portfolio audit (2026-03-23) found this was the primary cause of the **ES VWPM 44.4% live WR vs 66% backtest**. VWAP accumulator ran bar-wise in replay and tick-wise in live — completely different values, completely different entry signals.

**Detection heuristic:** `grep Calculate.OnEachTick` in any NinjaTrader strategy folder, then audit every `+=` that follows.

**Audit of record:** [[v5-strategy-bug-audit]].

**Concept page:** [[vwap-double-counting-bug]].

## Rule 10 — Small-risk trades must be floored

**Rule:** Any R-multiple calculation must apply a minimum risk floor (≥ 1.5pt for NQ, ≥ 6 ticks). Without it, tiny-stop trades produce artificial 10R+ winners when price keeps moving.

**Why:** V1 `fvg_exit_body_stop` produced +15,623R total / +3.68 R/T from 1,892 trades. After the 1.5pt floor: +502R total / +0.27 R/T — **1/30th of V1.** 53% of body-stop trades had <3pt risk. Tiny stops × ratio-based R = fabricated edge.

**Audit of record:** [[comprehensive-mining-report-v2|COMPREHENSIVE_MINING_REPORT V2]].

## Rule 11 — Entry-bar simulation starts at `idx+1`, not `idx`

**Rule:** Bar-level simulation of a trade entered at bar close must start checking stop/target from the *next* bar, not the fill bar itself.

**Why:** The fill bar's high and low are already in the past — they occurred before the close at which entry happened. Starting at `idx` checks whether the target/stop was hit *before* the trade existed. V1 made this mistake; V2 fixed it to `idx+1`.

**Audit of record:** [[comprehensive-mining-report-v2|V2]] §"Critical Corrections."

## Rule 12 — Pre-existing tick-level audits supersede later bar-sim results

**Rule:** If a tick-level audit exists for a signal, no later bar-level simulation result can rehabilitate it. The tick-level number is ground truth; the bar result is suspect unless explicitly cross-validated.

**Why:** `BOS_FVG_10PT_AUDIT.md` established in **March 2026** that the 10+pt static-target variant had +0.015 avgR baseline, zero edge after multiple-comparison correction. The `nq-playbook`'s flagship numbers in late-March/April were bar-level trailing results that appeared to contradict this. The April 10 bar-vs-tick replay resolved the contradiction — the tick-level audit was right, the bar-level trailing was wrong. **The earlier tick result should have been treated as the authoritative baseline all along.**

**Corollary:** Any future research that reports a result on a signal with a prior tick audit must explicitly reconcile. If the new result is higher, the reconciler must show a tick-level validation.

## Rule 13 — Forward-link every invalidated doc

**Rule:** When a document is invalidated by a later audit, the invalidated document's wiki summary must carry a top-of-file banner:

> **⚠️ Partially invalidated.** Bar-sim trailing results in this doc are suspect per [[bos-fvg-failure-consolidated]] (2026-04-10). Read for research shape, not for R-numbers.

This is how the vault stays honest. The raw source file in `raw-sources/` is immutable and keeps its original claims — the vault summary is where the correction lives, and every summary that cites suspect numbers must forward-link to the audit that found the bug.

**Currently banner-tagged:** `nq-playbook.md`, `research-journal.md`, `mining-analysis.md`, `v8-v9-audit-reports.md`, `comprehensive-mining-report-v2.md`, `strategy-mining-report-312d.md`.

## Rule 14 — The NautilusTrader cross-check was an early warning

**Rule:** When two trustworthy simulators disagree on the same signal's edge, the *lower* number is more likely correct. Do not explain away the disagreement — investigate it immediately.

**Why:** NautilusTrader (engine-native, tick-level trade management internally) reported BOS_FVG at **+0.286 avgR** across 9,188 US-session trades. The NQ Playbook's bar-sim reported **+1.141 avgR**. That ~4× gap was an early warning sign that was *explained away* as "different fill methodologies" — Nautilus enters at bar close vs mining's FVG-edge limit entry. The explanation was plausible but wrong: the fill-method difference should have produced ~10% variance, not 4×. The real gap was the bar-sim trailing bug.

**Corollary:** Whenever an explanation "feels convenient" — it accounts for exactly the gap you're trying to explain away — treat it as a yellow flag. Run the independent cross-check that would validate it.

## Rule 15 — Anti-stupidity rules (from the Unified Brain doc)

The [[unified-brain-architecture|Unified Brain doc]] hard-codes these into the research loop. They are promoted here as canonical rules for any future strategy research:

- **NEVER test multi-timeframe candle alignment** — known look-ahead trap
- **NEVER test break-even stops** on any signal — confirmed dead across 11 configurations
- **NEVER claim a strategy is validated based on backtesting alone** — backtest results are hypotheses, not facts
- **ALWAYS use conservative simulation** (worst-case OHLC ordering) for any trailing stop, AND cross-validate at tick resolution
- **ALWAYS run walk-forward validation** (minimum 4 quarters, all positive OOS)
- **ALWAYS apply realistic costs**: 0.5pt slippage + 0.5pt commission per side
- **NEVER re-test the 7 dead signals** (london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion) without a fundamentally new thesis
- **ALWAYS show RT (R target)** in every results table

## How to use this page

1. **Before proposing a new test**, check if any rule applies. If yes, the test needs a plan to avoid the contamination.
2. **Before citing a number from a research doc**, check if any of these bugs could have affected it. The doc should have a banner if so — if not, add one.
3. **Before committing a change to a research script**, check if the change re-enables a dead pattern (BE stops, bar-sim trailing, unclosed HTF bars). If so, stop.
4. **When in doubt, prefer the older tick-level result over the newer bar-level result.** Rule 12.

## Audits that produced these rules

- [[look-ahead-audit]] — Rule 2 (V10i)
- V8 Full Audit Report — Rule 4 (phantom B/E)
- V9 Full Audit Report — extends V8 findings
- BACKTEST_INTEGRITY_AUDIT_V2 — various simulator cleanups
- [[bos-fvg-10pt-audit|BOS_FVG_AUDIT_FINDINGS.md]] — early BOS_FVG analysis, historical
- [[bos-fvg-10pt-audit]] — Rules 5, 6, 7, 8 (P/D look-ahead, WF limitation, multiple comparison, outlier months)
- [[bos-fvg-failure-consolidated]] — Rule 1, Rule 12, Rule 14 (bar-sim trailing, tick supersedes bar, Nautilus cross-check)
- [[v5-strategy-bug-audit]] — Rule 9 (NT OnEachTick bar gating)
- [[comprehensive-mining-report-v2]] — Rules 10, 11 (min risk floor, idx+1)
- [[unified-brain-architecture]] — Rule 15 (hard-coded anti-stupidity list)

See [[maps/audit-history-moc|Audit History MOC]] for the chronological view.

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT|IFVG OPTIMIZATION REPORT]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_summary|tempo rules summary]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/README_QUANTCONNECT_IMPLEMENTATION|README QUANTCONNECT IMPLEMENTATION]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/TEMPO_RULES_IMPLEMENTATION|TEMPO RULES IMPLEMENTATION]]
