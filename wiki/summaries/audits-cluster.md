---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/audits/V8_Full_Audit_Report.md
  - raw-sources/trading/audits/V9_Full_Audit_Report.md
  - raw-sources/trading/audits/BACKTEST_INTEGRITY_AUDIT_V2.docx
  - raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18.md
  - raw-sources/trading/audits/BOS_FVG_AUDIT_FINDINGS.md
  - raw-sources/trading/audits/BOS_FVG_FAILURE_CONSOLIDATED.md
  - raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md
related:
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/bos-fvg-10pt-audit.md
  - wiki/syntheses/mining-reports-v1-v2-reconciliation.md
tags: [trading, audits, cluster-summary, look-ahead, bar-sim]
---

# Audits Cluster — 7 Sources

The definitive collection of look-ahead and simulator audits on the Tempo research corpus. This is the cluster that contains the **ground truth** about what's real vs what was contaminated.

**Critical:** the two most recent audits ([[bos-fvg-failure-consolidated|BOS_FVG Failure Consolidated 2026-04-10]] and [[bos-fvg-10pt-audit|BOS_FVG 10pt Audit 2026-03-08]]) are the authoritative position. The older audits documented earlier contamination that was fixed, but the bar-sim trailing bug survived those fixes and wasn't caught until April 10.

## File catalog (chronological)

### `LOOK_AHEAD_AUDIT_2026-02-18.md` (7KB)
First systematic look-ahead audit — February 2026. This is the audit that identified the [[v10i-look-ahead-bug|V10i look-ahead bug]] in `get_alignment()`. The fix: use completed bars only (`mask = tf_c.index + TF_DURATION[tf_label] <= entry_time`). Historical but still useful as the chronological anchor — V10a–V10n alignment results are invalidated by this audit.

### `BACKTEST_INTEGRITY_AUDIT_V2.docx` (20KB)
V2 backtest integrity audit. Broader sweep of the simulation pipeline after the V10i fix. Looked for additional bugs in the simulator — but **did not catch the bar-sim trailing bug**, which required paired bar-vs-tick replay to identify. This audit's "clean" numbers are therefore suspect for any trailing-stop strategy.

### `V8_Full_Audit_Report.md` (10KB)
Full audit of the V8 mining era. Confirms the V8 phantom B/E bug in `micro_structure.py` (JSON trades have `rej_delta_aligned`/`delta_divergent` as string "True"/"False"; numpy bool bug). **V8 anything with B/E is contaminated.** See memory entry `v8_bugs`.

### `V9_Full_Audit_Report.md` (11KB)
Audit of the V9 generation — after V8 fixes. V9 improvements over V8 are documented here. Still predates the bar-sim trailing finding.

### `BOS_FVG_AUDIT_FINDINGS.md` (6KB)
Early BOS_FVG audit findings. Historical — superseded by the two 2026 audits below.

### `BOS_FVG_10PT_AUDIT.md` — [[bos-fvg-10pt-audit|see dedicated summary]]
**2026-03-08.** Tick-level audit of the 10+pt static T3R variant. Baseline avgR +0.015, all filters fail multiple-comparison correction. **Still valid** — was tick-level.

### `BOS_FVG_FAILURE_CONSOLIDATED.md` — [[bos-fvg-failure-consolidated|see dedicated summary]]
**2026-04-10.** The canonical failure report. Paired bar-vs-tick trailing replay on 60 days of Databento NQ data. Bar: +0.359 avgR. **Tick: +0.001 avgR.** Delta: +0.29 R per trade inflation. Generalizes the 10pt audit's finding to the full BOS_FVG population and to the trailing-stop configuration. This is the authoritative BOS_FVG position. **Supersedes everything else in this cluster.**

## Chronology of contamination findings

| Date | Audit | What it found | What it missed |
|---|---|---|---|
| 2026-02-18 | [[v10i-look-ahead-bug|V10i look-ahead audit]] | Multi-TF alignment used unclosed HTF bars — future close data leaked in | Bar-sim trailing inflation |
| 2026-02-XX | Backtest Integrity V2 | Various simulator bugs fixed | Same |
| 2026-03-08 | [[bos-fvg-10pt-audit|10pt tick audit]] | 10pt static T3R has no tradeable edge (+0.015 avgR). Every 95% CI spans zero. | (nothing — this audit was thorough for its scope) |
| 2026-03-XX | V8, V9 full audits | V8 phantom B/E bug, V9 improvements over V8 | Bar-sim trailing inflation |
| **2026-04-10** | [[bos-fvg-failure-consolidated|BOS_FVG consolidated failure]] | **Bar-sim trailing inflates by +0.29 avgR/trade** across the BOS_FVG population; full tick replay is +0.001 avgR | nothing — this is the current authoritative position |

## The pattern across audits

Each audit catches a different contamination source:
1. **Look-ahead** (V10i) — future data in feature computation
2. **B/E phantom fills** (V8) — simulator fills break-even stops that never actually executed
3. **Bar-sim path reconstruction** (2026-04-10) — trailing stops simulated on OHLC are structurally wrong

**Rule:** any research that ran before a specific audit is suspect for the contamination that audit identified — EVEN IF the research appeared to survive earlier audits. The V2 comprehensive mining report survived V10i but not bar-sim.

## Cross-references

- [[v10i-look-ahead-bug]] — the specific alignment contamination
- [[bar-sim-trailing-bug]] — the structural OHLC path reconstruction bug
- [[bos-fvg]] — the signal concept, now tagged `dead-strategy`
- [[mining-reports-v1-v2-reconciliation]] — synthesis that tracks what survived what
- [[unified-brain-architecture|Unified Brain doc]] — the current authoritative position on validated vs unvalidated strategies (all V7-V11 research unvalidated)

## Source documents

- [[raw-sources/trading/audits/BOS_FVG_AUDIT_FINDINGS]]
