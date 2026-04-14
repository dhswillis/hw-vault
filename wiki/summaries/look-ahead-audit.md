---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18.md
related:
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/research-journal.md
  - wiki/summaries/nq-playbook.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, audit, methodology, look-ahead, foundation]
---

# Look-Ahead Audit 2026-02-18 — Summary

## What the source is

`~/HW/raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18.md` — **143 lines**, dated 2026-02-18. A foundation-level audit of the early signal-detection and footprint-analysis code in `context-engine/` and `micro_structure.py` that caught several structural look-ahead bugs the early V7/V8 mining was silently carrying. Predates both the V10o alignment look-ahead discovery (documented in the research journal) and the 2026-04-10 bar-sim trailing audit.

## The bugs found

### Bug 1 — Swing point confirmation timestamp (CRITICAL)

The early swing-detector returned swings as soon as the lookback window was filled, using the swing's own timestamp as its "usable from" time. But a 3-bar-confirmed swing high isn't really confirmed until **lookback bars AFTER** the swing prints. For a lookback of 3, a swing at bar 100 is only confirmed at bar 103.

**Impact:** Strategies that used swings for BOS detection could react to a swing before it was actually confirmed, effectively using future bars. Not a trailing-stop bug but a signal-layer bug.

**Fix:** Every SwingPoint carries a `confirmed_idx = idx + lookback`, and downstream code only consumes swings whose `confirmed_idx < current_bar`. The `clean_backtest.py` implementation and `bos_fvg_full_audit.py` both follow this rule correctly; earlier V7/V8 code did not.

### Bug 2 — Footprint absorption detection (CRITICAL)

Footprint absorption logic in `context-engine/features/footprint.py` was reading ahead in the delta array to confirm absorption patterns. Specifically, the absorption detector had a window that extended past the current bar's completion time.

**Impact:** Any V8-era signal using the "absorption" filter was contaminated by future delta readings. This later interacted with the V10i alignment bug: the absorption filter's apparent edge was a mix of look-ahead + alignment look-ahead + bar-sim bugs, and couldn't be cleanly isolated until V10o.

**Fix:** Absorption detector constrained to the currently-closed bar's footprint. Re-run of V8 data after this fix shows absorption as "too rare for meaningful signal (28 trades at V8g ≥ 3)" — confirming that the pre-fix absorption edge was mostly look-ahead.

### Bug 3 — Swing filtering false positives

A secondary issue investigated in the audit: some swings were being filtered out for "false positive" reasons that the audit concluded were actually valid swings. The code had an overly aggressive filter that caught some real structural swings along with noise.

**Impact:** Smaller than Bugs 1 and 2 — resulted in a reduced swing population rather than contamination of existing swings.

**Fix:** Relaxed filter tolerances. Trade-off: slightly more swings, some of which are noise, but no genuine swings being filtered out.

## What this audit does NOT catch

This audit is foundation-level and pre-dates the two later bugs that ultimately killed BOS_FVG's results:

1. **[[v10i-look-ahead-bug|V10i alignment look-ahead]]** — the `tf_c.index ≤ candles_1m.index[idx]` pattern that included uncompleted higher-TF bars. This wasn't discovered until V10o, several months after this audit.
2. **[[bar-sim-trailing-bug|Bar-sim trailing bug]]** — the structural inability to simulate trailing stops on OHLC bars. This wasn't discovered until 2026-04-10, a year and a half after this audit.

The audit's scope was signal-layer look-ahead (is the signal using future data?). The V10i bug is filter-layer look-ahead (is the confluence filter using future data?). The bar-sim bug is exit-layer (is the trade management reconstructing intra-bar paths correctly?). All three are "look-ahead" in the broad sense, but they're in different layers of the backtester and require different fixes.

## Why this doc is still worth reading

1. **It's a worked example of what an honest look-ahead audit looks like.** The author goes through each suspect component, identifies the specific code that was wrong, explains the mechanism, and reports the re-run results. This is the template for [[bos-fvg-failure-consolidated]] and should be the template for any future audit.
2. **It establishes the fix patterns** — `confirmed_idx = idx + lookback`, closed-bar-only filters, explicit timestamp comparisons — that later code relies on.
3. **It's evidence that look-ahead bugs come in waves.** Bug 1 + Bug 2 in February, V10i in roughly May/June (research journal timing), bar-sim in April 2026. Each wave invalidated the previous era's headline numbers. The project has not had a "clean" backtesting epoch yet, only successively-less-dirty ones.

## Methodology rules established by this audit

- **Swing confirmation is a property of the swing**, not of the consumer. Every swing object carries the bar index at which it became usable.
- **Footprint and delta features are closed-bar-only**. If the bar hasn't closed yet, the feature is undefined for downstream consumers.
- **Re-running old results on fixed infrastructure is non-optional** whenever a new look-ahead bug is found. This rule held for the V10o discovery (which triggered the V10p/V10r/V10z cleanup) and for the 2026-04-10 discovery (which triggered [[bos-fvg-failure-consolidated]]).

## Cross-references

- [[v10i-look-ahead-bug]] — the next major look-ahead bug after this audit
- [[bar-sim-trailing-bug]] — the third wave, the one that killed BOS_FVG
- [[research-journal]] — narrative context for how these bugs fit into the V7 → V11 arc
- [[bos-fvg-failure-consolidated]] — the 2026-04-10 audit that follows this one's template
- [[tempo-three-layers]] — Layer 2 methodology rules

## Source documents

- [[raw-sources/trading/audits/LOOK_AHEAD_AUDIT_2026-02-18]]
