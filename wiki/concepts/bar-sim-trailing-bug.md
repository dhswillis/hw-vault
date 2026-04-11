---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md
  - ~/Documents/strategies/03-sf-portfolio/backtests/clean_backtest.py
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
tags: [trading, backtesting, bug, methodology, nq]
---

# Bar-Sim Trailing Stop Bug

The structural reason you cannot backtest a trailing stop on 1-minute (or any) OHLC bars and get a trustworthy average-R. Every trailing-stop result in the Tempo research corpus that used a bar-level simulator is affected by this, to a degree proportional to how much intra-bar range the strategy sees.

## Summary

A trailing stop's exit price depends on the **sequence** in which the underlying touches a bar's extremes, not just the extremes themselves. A 1-minute bar collapses roughly 1,000 ticks into four numbers (O/H/L/C) and discards the sequence. Any simulator that drives a trailing stop from OHLC has to guess the sequence, and the most natural guess — "check stop first, then update max-favorable from bar_high, then tighten trail for the next bar" — systematically favors the winner by locking in high-water marks that, in the real tick path, would have been unwound within the same bar.

On BOS_FVG trailing (0.5R trigger, 0.1R buffer) run on a 60-day Databento NQ sample, the bar sim reports **+0.36 avgR** and the tick sim reports **+0.001 avgR** on the identical signal set. The delta is +0.29 R per trade. The entire reported "edge" is the bug.

## Mechanism

`clean_backtest.py` and its descendants process each post-fill bar in this order:

1. Check if `bar_low <= current_stop` (for longs). If so, exit at current_stop − slippage.
2. Update `max_favorable_r` using `bar_high`.
3. If `max_favorable_r >= TRAIL_TRIGGER_R`, tighten `current_stop` to `entry + (max_favorable_r − TRAIL_BUFFER_R) * risk`.

The stop is checked against the **old** trail level, but the trail is tightened using the **new** max-favorable from `bar_high`. If the bar's real sequence was `open → high → low → close` (high first), the tightened trail would have been in place by the time price dropped to `bar_low`, and the bar would have stopped out at the tighter trail. The sim doesn't see that; it carries the tightened trail forward to the next bar, where it may or may not be hit.

This converts "trade that would have stopped at 1.9R within the same bar" into "trade that stays alive at the 1.9R trail and captures more." Over thousands of trades on a signal where 0.5–1.0R intra-bar ranges are common, the inflation compounds.

## Two variants, two magnitudes

There are actually two related bugs at different scales.

**Variant A — strict intra-bar re-check.** Reorder only: after step 3 above, re-check the stop against `bar_low` with the newly-tightened trail. This is a conservative fix that catches the "high-first-then-low" cases within a single bar.

Measured impact on BOS_FVG 1-minute trailing (60-day sample, 858 trades):
- Mode A (original ordering): +0.3418 avgR
- Mode B (re-check after tighten): +0.3372 avgR
- Delta: **+0.0046 avgR**, only 4 of 858 trades affected

The strict ordering bug is mostly inert on 1-minute NQ bars because intra-bar ranges rarely exceed 0.5R on BOS_FVG-sized FVGs. Reordering is not enough.

**Variant B — full tick replay.** Throw out OHLC entirely for trade management. Step through the actual tick sequence, updating max-favorable, trail, and stop checks in real time.

Measured impact on the same sample (842 trades filled in both sims):
- Bar avgR: +0.2936
- Tick avgR: +0.0010
- **Delta: +0.29 R per trade**
- 36.1% of trades had bar > tick (inflation cases)
- 28.5% of trades had bar < tick (deflation cases)
- Net favors the bar sim by ~0.29 R

Variant B is **~60x larger** than Variant A. The conclusion is that the real damage isn't the intra-bar ordering of a single logic step — it's the act of reconstructing a path from four numbers when the strategy's P&L lives in the path.

## Why this isn't caught by "conservative" checks

`clean_backtest.py` does add a conservative fill-bar check: if the fill bar's low breaks through the initial stop, the trade is recorded as an immediate −1R loss. That catches initial-stop-on-fill-bar cases. It does not help with later bars, where the trail is moving and the sim has to guess the intra-bar sequence.

Similarly, the file has a comment block at the top that reads "LOOK-AHEAD PREVENTION: Swing points require confirmation, BOS only fires on confirmed swings, FVG fill checks start the bar after the FVG forms, etc." Those are all correct, and the signal side is clean. The bug is not look-ahead in the conventional sense — it's path reconstruction during trade management. The author was looking in the right place for the wrong bug.

## What this invalidates in Tempo research

- `clean_backtest.py` outputs (BOS_FVG + pure trail, +0.895 avgR / +13.37 R/day / Calmar 237)
- `BOS_FVG_RESEARCH.md` headline numbers (+0.354 avgR / Calmar 151)
- V10y/V10z 15S trailing results (+1.141 avgR / Calmar 263.6) — 15s bars compress less path but the bug applies
- V10v/V10u "trailing" results already noted in `MEMORY.md` as "optimistic intra-bar ordering" but partially rehabilitated in V10z; all now suspect
- Any `/tmp/mine_*.py` or `/tmp/ifvg_optimize_v*.py` script that trails on OHLC (IFVG's "soft stop" exit is similar in spirit but exits on close-through rather than wick-touch, which makes it bar-sim-safe — those results survive this audit)
- Any composite portfolio that includes a bar-trailed leg

## What this DOES NOT invalidate

- Fixed-R-target backtests (static T3R, T5R, etc.). Fixed targets don't depend on intra-bar sequence — a bar either reaches the target or it doesn't. The 10+pt static audit's baseline of +0.015 avgR is safe and was already tick-validated.
- **Close-through stops** (IFVG "soft stops" — exit on candle CLOSE past the stop, not wick touch). These are bar-sim-safe by construction because the close is unambiguous regardless of intra-bar path. This is why IFVG's tempo audit numbers remain defensible.
- NinjaTrader engine-native execution. The platform runs against tick-by-tick data, not bar OHLC. `SweepBreakv17.cs` and v24 IFVG MTF cascade are not affected.
- The signal-detection logic itself. BOS, FVG, sweep, swing detection all run on completed bars and are unaffected.

## Rule of thumb

**Do not trail on bars.** If a strategy's exit is a trailing stop, either:
1. Run it at tick resolution (`sim_tick_trailing` style — step through each tick), OR
2. Replace the trailing exit with a fixed R target or a close-through soft stop, both of which are bar-sim-safe.

Do not try to fix the bug by reordering the logic within a bar. The reorder catches <1% of affected trades. The only correct fix is throwing away OHLC for trade management.

## Reproducers

Both in `/tmp/`, runnable with `~/Documents/trading-system/.venv/bin/python`:

- `bos_fvg_bug_reproducer.py` — isolates Variant A (intra-bar ordering). Produces the 0.5% / +0.005 avgR delta.
- `bos_fvg_bar_vs_tick.py` — paired bar-vs-tick sim on the same signals. Produces the +0.29 avgR delta and the +0.001 avgR tick result.
