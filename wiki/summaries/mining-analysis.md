---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/context-engine/MINING_ANALYSIS.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/research-journal.md
  - wiki/summaries/nq-playbook.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, mining, suspect-results, matrix-analysis]
---

# Mining Analysis — Summary

> **Layer 2 — contaminated.** This is a V7aa_b cross-tabulation of 2,600 signal × pattern × session × DOW × alignment combinations. It pre-dates both the V10i alignment look-ahead discovery and the 2026-04-10 bar-sim audit. The MTF-alignment-based WR claims are invalidated by [[v10i-look-ahead-bug|V10i]]. Use this doc for the *shape* of the cross-tab (what combinations were even tested) rather than the WR numbers.

## What the source is

`~/HW/raw-sources/trading/context-engine/MINING_ANALYSIS.md` — **588 lines**. A matrix-style analysis from the V7aa_b mining phase that enumerated ~2,600 trade combinations across: primary signal, candle pattern, session, day-of-week, and MTF alignment level. The output is a three-tier recommendation: 31 combos classified for full size, 57 for half size, 193 for avoid.

## Claimed headline findings

(Framed as-written from the source; all suspect due to alignment look-ahead.)

- **Best combo overall:** doji_gravestone + BOS_FVG at 55.6% WR, +3.955 avgR
- **Runner-up:** doji_dragonfly + BOS_FVG at 67.4% WR
- **Alignment is critical:** confirming alignment = +1.28R, contradicting alignment = −0.35R
- **Thursday is strongest DOW;** Monday the weakest
- **NY AM + BOS_FVG + doji pattern + confirming alignment** is the canonical "full size" combo

## Why the headline is suspect

The MTF alignment numbers used throughout this doc rely on the same `tf_c.index ≤ candles_1m.index[idx]` pattern that [[v10i-look-ahead-bug|V10i]] identified as look-ahead. The +1.28R confirming vs −0.35R contradicting gap is almost certainly the look-ahead bug pattern — "confirming alignment" really means "future bar close matches trade direction," which is trivially true and matches the trade outcome rather than preceding it.

Additionally, most of the combos use BOS_FVG trailing as the base, and inherit the [[bar-sim-trailing-bug]] inflation. The "55.6% WR at +3.955 avgR" reading is not reachable by any tick-level measurement of the same signal.

## What this doc is still good for

1. **The cross-tab structure** — the matrix of (signal × pattern × session × DOW × alignment) is a reasonable taxonomy and could be re-run correctly. Future work that wants to test candle-pattern filters (dojis, engulfings) could start with this list of combos and run them tick-level with confluence filters that don't have look-ahead.
2. **The "avoid" list** (193 combos) — if something is negative across every parameter set, that's robust to most contamination. The avoid list carries more signal than the full-size list.
3. **DOW and session observations** (qualitatively) — Thursday better than Monday, NY AM better than NY PM are patterns observed in many other docs too. The *magnitude* is inflated, but the sign is probably right.

## What to strip when citing

- All R-numbers and Calmar values
- All WR numbers above ~50% that are attributed to alignment combos
- All "X + alignment = full size" recommendations

## What to keep when citing

- The signal × pattern taxonomy itself
- The avoid list (strategies that failed across all parameter sets)
- The methodology (cross-tab analysis as a discipline for generating hypotheses)
- The DOW / session qualitative patterns

## Cross-references

- [[v10i-look-ahead-bug]] — the bug that invalidates this doc's alignment-based claims
- [[bar-sim-trailing-bug]] — the bug that invalidates the trailing-stop R-numbers
- [[nq-playbook]] — the later consolidated version of the V10 research arc that succeeded this doc
- [[research-journal]] — narrative context for the V7 → V11 era
- [[tempo-three-layers]] — Layer 2 framework
