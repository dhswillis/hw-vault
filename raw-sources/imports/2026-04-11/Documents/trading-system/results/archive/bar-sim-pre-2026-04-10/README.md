# Archived Bar-Sim Outputs (pre-2026-04-10)

These 39 files are **suspect** and should not be cited as evidence of edge.

They were produced by backtesters that ran the BOS_FVG trailing-stop logic on 1-minute (or 15-second) bar OHLC data. On 2026-04-10, a paired bar-vs-tick replay of the same signal population showed that 1-minute bar simulation inflates trailing-stop results by **+0.29 R per trade** relative to tick-level ground truth. With the inflation removed, BOS_FVG has **+0.001 avgR** — no edge.

**Canonical explanation**: `../BOS_FVG_FAILURE_CONSOLIDATED.md`
**Structural bug**: `~/HW/wiki/concepts/bar-sim-trailing-bug.md`

## What you can still use these for

- The **relative ranking** of grid cells (e.g. which `bos_disp` value produces more signals) may still carry information about signal density.
- The **count** and **direction distribution** of trades are not affected — only the R-result column is inflated.
- The **signal-generation side** (swing detection, BOS detection, FVG detection) was clean. Any work that re-uses the timestamps or entry/stop levels without re-using the R-results is fine.

## What you cannot use these for

- Any absolute R/avgR/Calmar/WR number from these files.
- Any "portfolio" composite that weights these files' trades against others.
- Any filter-ranking based on `r_result` averages — the ranking itself may flip under tick replay.

## If you need the real numbers

Re-run the signals with tick-level trade management. A minimal reproducer is at `/tmp/bos_fvg_bar_vs_tick.py` (session-local; it will need to be re-created if not present). The pattern is:

1. Build 1-minute OHLCV from tick data — fine for signal detection
2. Detect swings → BOS → FVG → fill — all on completed bars, no issue
3. For trade management, step through the actual ticks, not the bar OHLC
4. Apply trailing stop rules tick-by-tick, updating max_favorable_r and current_stop in real time

Alternatively, use fixed-R targets (static T3R, T5R) or close-through soft stops instead of a trailing exit — both are bar-sim-safe and robust to intra-bar path ordering.
