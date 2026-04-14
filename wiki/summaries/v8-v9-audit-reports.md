---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/audits/V8_Full_Audit_Report.md
  - raw-sources/trading/audits/V9_Full_Audit_Report.md
related:
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/research-journal.md
  - wiki/summaries/look-ahead-audit.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, audit, v8, v9, suspect-results]
---

# V8 and V9 Full Audit Reports — Summary

## What the sources are

Two audit reports from the V8/V9 era of the BOS_FVG mining research:

- `~/HW/raw-sources/trading/audits/V8_Full_Audit_Report.md` — **199 lines**, dated February 2026. Full audit of the V8-phase mining results (MTF alignment confluence scoring).
- `~/HW/raw-sources/trading/audits/V9_Full_Audit_Report.md` — **202 lines**, dated February 2026. Full audit of the V9-phase mining results (signal composition / hybrid entries built on top of V8).

Both pre-date the V10o discovery of the MTF alignment look-ahead bug, and therefore both "validated" results they describe were later invalidated when V10o/V10p re-ran with completed-bar-only alignment. See [[research-journal]] for the full narrative.

## Role in the research arc

These reports exist because after each mining phase, Harrison ran a formal audit to check whether the phase's results should be trusted. Each audit caught some bugs but missed the structural alignment look-ahead — it was hiding in the shared `tf_c.index ≤ candles_1m.index[idx]` pattern that ran across every V8/V9 script and wasn't recognized as a bug until V10o directly compared "bars whose start ≤ entry time" vs "bars whose close ≤ entry time" and noticed the gap.

**V8 audit claims (all now invalidated):**
- V8g ≥ 4 (4+ timeframes confirming): 61.7% WR OOS, +3.501 avgR
- V8g ≥ 5 (5+ timeframes confirming): 79.6% WR OOS, +4.801 avgR
- V8h with candle pattern filter: "sniper" tier at 90%+ WR, low trade count
- Walk-forward validation: positive across the 70/30 split

**V9 audit claims (all now invalidated):**
- V9 hybrid signal composition on top of V8 alignment
- Confluence-scored signals at 80%+ WR
- Monte Carlo simulation showed "robust" edge

**What the V10o audit later found:**
- The `tf_c.index ≤ candles_1m.index[idx]` pattern in the alignment checker was including bars whose *start* time was before the entry timestamp, but whose *close* was in the future. For a 1H bar starting at 10:00 with entry at 10:03, the sim was effectively reading the bar's close price at 11:00 — 57 minutes of look-ahead per entry.
- After fixing with `tf_c.index + TF_DURATION[tf_label] ≤ entry_time`, all V8 and V9 signals collapsed to ~50% WR across the board. The "massive edge" was entirely the look-ahead.
- V10p tested 5 clean alignment methods (completed direction, open vs price, previous close vs price, EMA trend, multi-bar) — **ALL 5 DEAD**. Every combo negative avgR.

## What the audits did catch

The V8 and V9 audits were not useless — they caught several real bugs that made it into `micro_structure.py`:

1. **V8g numpy bool bug** — JSON trades had `rej_delta_aligned` / `delta_divergent` as string "True"/"False" rather than booleans. Fixed with explicit conversion: `t[key] = t[key] == 'True'`.
2. **V8u trade truncation** — The original V8u script saved only the first 200 tagged trades (`tagged_trades[:200]`), losing 1,490 of 1,690. Fixed to save all trades.
3. **Direction enum mismatches** — `Direction.LONG` vs `Direction.BULLISH` inconsistency across the V8 code.
4. **SwingPoint attribute naming** — `.type` vs `.swing_type` mismatch.

These are genuine bugs that the audit process identified and fixed. The bar-level alignment bug was not in the audit's scope because nobody was looking for it yet — the discovery required cross-comparing two different alignment implementations (which V10o did).

## Why both reports should still be tagged suspect

Even setting aside the alignment bug, these audits did not:
- Apply **Bonferroni correction** for the multiple-comparison environment (100+ filter combos swept)
- **Cross-validate at tick level** (all results are 1-minute or 3-minute bar sim)
- **Pressure-test against NautilusTrader or another engine** (NautilusTrader cross-val didn't happen until later and showed a very different number)
- **Test for look-ahead in the confluence filter layer** (the audits checked signal layer, not filter layer)

The audits are **Layer 2 methodology documentation** — what the project was doing before V10o and how it was being checked. They are not evidence of edge, and their "validated" claims carry forward into `V8_MINING_SYNTHESIS.md` and `MINING_ANALYSIS.md` which are also tagged suspect.

## What to cite from these

- **The bug list** — V8g numpy bool, V8u truncation, direction enum, SwingPoint attribute. Real fixes, useful for understanding why certain old JSON files need post-processing.
- **The walk-forward methodology** (70/30 train/test split, 174 train / 75 test days, Monte Carlo 10,000 runs) — still the right pattern, just ran on contaminated data.
- **The signal taxonomy** — V8g through V8z variants, each with a specific hypothesis tested. Useful as inventory of ideas even if none worked.

## What NOT to cite

- Any V8g ≥ N WR number
- Any V9 "validated" hybrid signal composition
- Any "sniper tier" 90%+ WR claim from this era
- The walk-forward "positive across split" result (positive with look-ahead, not positive without)

## Cross-references

- [[v10i-look-ahead-bug]] — the bug that invalidated every V8/V9 alignment-based result
- [[research-journal]] — narrative context for the V7 → V11 arc
- [[look-ahead-audit]] — the earlier signal-layer audit from Feb 2026
- [[mining-analysis]] — the cross-tab companion to V8_MINING_SYNTHESIS
- [[tempo-three-layers]] — Layer 2 framework that places these audits in context

## Source documents

- [[raw-sources/trading/audits/V8_Full_Audit_Report]]
- [[raw-sources/trading/audits/V9_Full_Audit_Report]]
