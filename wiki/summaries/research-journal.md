---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/RESEARCH_JOURNAL.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/nq-playbook.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/look-ahead-audit.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, mining, research, suspect-results, narrative]
---

# Research Journal — Summary

> **Layer 2 — contaminated narrative.** The R-numbers in this doc are the same ones that `NQ_PLAYBOOK.md` carries and are invalidated by the 2026-04-10 tick audit. Unlike NQ_PLAYBOOK, this doc reads as a **chronological narrative** of the V7 → V11 research arc, which makes it valuable as a map of *what was tried* even if the performance numbers don't hold. Read it for the shape of the research, not for scores. See [[tempo-three-layers]] and [[bos-fvg-failure-consolidated]].

## What the source is

`~/HW/raw-sources/trading/RESEARCH_JOURNAL.md` — **359 lines**, narrative-style journal of 115+ backtesting experiments from V7 through V11. Written as a running account of hypotheses tested, results observed, and pivots made. More prose and less table-dense than [[nq-playbook]], which makes it the better doc for understanding *why* each experiment happened.

## The experimental arc (V7 → V11)

### V7 — Initial signal composition
- Tested raw signal candidates: BOS_FVG, sweep_reversal, VP_POC_retest, fib_retracement, failed_breakout, VWAP_mean_reversion, double_BOS_momentum, london_breakout
- Several apparent edges, most of which later failed cleanup
- Early framework: "find the signal, then find the filter stack"

### V8 — MTF alignment becomes the star
- Added multi-timeframe alignment (1m / 3m / 5m / 15m / 1h / 4h) as a confluence filter
- V8g (≥4 TFs aligned) apparently produces 61.7% OOS WR, +3.501 avgR
- V8g (≥5 TFs aligned) produces 79.6% OOS WR, +4.801 avgR
- **This was the peak of the mining "we have a massive edge" era.**

### V9 — Signal composition / hybrid entries
- Explored combinations of signals and confluence filters on top of the V8 alignment finding
- Variants: delta divergence, footprint absorption, candle body ratios, volume spikes
- Everything continues to look great — "the core signal + alignment + structure filters" is the winning formula

### V10 — The audit reset
- **V10o discovery (the plot twist):** the alignment filter had a look-ahead bug. `tf_c.index ≤ candles_1m.index[idx]` includes bars whose *start* time is before the entry but whose *close* is in the future. For a 1H bar starting at 10:00 with entry at 10:03, the sim was reading a close that was 57 minutes in the future.
- **V10p cleanup:** 5 clean alignment methods tested (completed_dir, open_vs_price, prev_close_vs_price, ema_trend, multi_bar). **All 5 DEAD.** Every combo is negative avgR at Calmar 1.0 (pure drawdown).
- **Everything V7/V8/V9 claimed disappears when alignment is done correctly.**
- **V10r:** native swing targets (no alignment). BOS_FVG is the only survivor: 361 trades, 62.6% WR, +0.460 avgR, Calmar 26.0, +0.53 R/day. Everything else dead.
- **V10t walk-forward:** fixed-R targets. Engulfing_key_level and ORB_breakout pass OOS without B/E; BOS_FVG variants using BE=1.0 are suspect due to phantom B/E bug in `simulate_outcomes()`.
- **V10v/V10u trailing:** "rescued" several signals with trailing exits. But these numbers are later admitted to use "optimistic intra-bar ordering" in the journal itself.
- **V10y/V10z:** 15S bars + "conservative" trailing sim (re-check stop after HWM update). Claims BOS_FVG is rehabilitated at +1.141 avgR, Calmar 263.6 on 685 trades. 4/4 walk-forward quarters positive.

### V11 (implied)
- Parameter sweeps on top of V10z
- Best BOS_FVG config: 6,091 trades, +0.924 avgR, Calmar 335.3 after swing lookback tuning (3 → 2), BOS displacement tuning (2.0 → 1.0), risk max cap (50 → 20pt)
- NautilusTrader cross-validation: 9,188 trades, 55.6% WR, +0.286 avgR — noticeably lower than Python sim, but the journal treats it as "different strategy surface" rather than a red flag

## The two signal cleanup rounds (what died at each gate)

**V10o round (look-ahead removed):** Everything that relied on MTF alignment collapsed. V8g ≥ 4 and ≥ 5 tiers, engulfing_key_level V10z (+0.117 → −0.105), pre_gap_fill V10z (+0.229 → −0.059). All MTF confluence results dead. The V10p clean methods also all dead — alignment as a concept does not produce edge on this dataset.

**2026-04-10 tick audit (not in this doc, post-dates it):** Everything that relied on trailing stops collapsed. BOS_FVG V10z +1.141 avgR becomes +0.001 avgR. Vol_spike_FVG and ORB_breakout with 15S trailing inherit proportional inflation. The "3-signal survivor portfolio" from the V10z era dissolves.

The tick audit is the second major reset the project has gone through, and this journal predates it. When you read a V10z number in here, remember it was state-of-the-art at the time the journal was written — and remember that "state of the art" in Layer 2 mining just means "most recent bug not yet found."

## Signal-by-signal verdict as of 2026-04-10

| Signal | V10z claim | Post-tick-audit |
|---|---|---|
| BOS_FVG 15S trailing | +1.141 avgR, Cal 263.6 | +0.001 avgR (dead) |
| BOS_FVG 10+pt static T3R | — | +0.015 avgR (dead, tick-validated in audit doc) |
| vol_spike_FVG 15S | +0.665 avgR | suspect, needs tick replay |
| ORB_breakout 15S | +0.471 avgR | suspect, needs tick replay |
| engulfing_key_level V10z | −0.105 avgR (already dead) | dead (not rehabilitated) |
| VA_fade V10z | −0.206 avgR (already dead) | dead |
| pre_gap_fill V10z | −0.059 avgR (already dead) | dead |
| sweep_reversal | dead across parameters | dead (robust) |
| fib_retracement | dead | dead |
| failed_breakout | dead | dead |
| double_BOS_momentum | dead | dead |
| VWAP_mean_reversion | dead | dead |
| london_breakout fixed target | dead | dead |
| london_breakout trailing | "rescued by V10v" | suspect (trailing bug) |
| VP_POC_retest | dead | dead |

## Methodology rules worth preserving

The journal documents several backtesting discipline rules that were learned the hard way:

1. **Walk-forward with 3-month train / 1-month test rolling windows, minimum 10 folds.** A real edge should be positive in >60% of folds.
2. **Bootstrap 95% CIs on all filter combos, 1000 resamples.** Flag if CI contains zero.
3. **Permutation tests on top candidates** — shuffle filter labels 1000 times, compare to random.
4. **Bonferroni correction for multiple comparisons.** 25-filter sweeps need p < 0.002, not p < 0.05.
5. **Always test tick-level, not bar-level**, for strategies whose edge depends on intra-bar paths (trailing stops especially).
6. **Cross-validate on a different engine** (NautilusTrader, NinjaTrader). Large disagreements are a red flag, not a surface difference.
7. **Beware of "fill-bar favoritism"** — any bar where the stop was inside the fill bar should be counted as −1R immediately, not passed through.

Most of these made it into the BOS_FVG_10PT_AUDIT but not into clean_backtest.py or the V10 series. The research journal documents both — what the rules *should* be and the fact that they weren't consistently applied.

## Cross-references

- [[nq-playbook]] — the table-heavy companion doc for the same research arc
- [[bos-fvg-failure-consolidated]] — the 2026-04-10 correction
- [[bar-sim-trailing-bug]] — the bug that invalidated V10y/V10z results post-audit
- [[v10i-look-ahead-bug]] — the earlier bug documented in this journal's V10o section
- [[look-ahead-audit]] — the foundation fixes from earlier in the project (swing confirmation, footprint)
- [[tempo-three-layers]] — this doc is the canonical Layer 2 narrative
