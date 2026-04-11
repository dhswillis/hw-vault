---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources:
  - ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md
  - ~/Documents/trading-system/results/BOS_FVG_10PT_AUDIT.md
  - ~/Documents/trading-system/results/BOS_FVG_RESEARCH.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/comprehensive-mining-report-v2.md
tags: [trading, nq, suspect-results, superseded, dead-strategy]
---

# BOS_FVG Failure Report (2026-04-10)

Canonical summary of `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md`. Consolidates every prior claim that [[bos-fvg|BOS_FVG]] has tradeable edge and replaces them with the tick-level result: **+0.001 avgR**, flat zero, no edge.

## The core finding

On 2026-04-10, a fresh paired bar-vs-tick replay of BOS_FVG on a 60-day Databento NQ sample produced:

| Simulation | n | avgR | WR | Total R | R / day |
|---|---|---|---|---|---|
| Bar-level (1min OHLC, `clean_backtest.py` logic) | 875 | **+0.3590** | 41.0% | +314.2 | +5.24 |
| **Tick-level (ground truth)** | **842** | **+0.0010** | **66.9%** | **+0.8** | **+0.01** |

On the 842 trades filled in both sims: **bar – tick = +0.29 avgR per trade**. The entire reported edge is a bar-reconstruction artifact, not a signal property.

This reproduces and generalizes the earlier finding in `BOS_FVG_10PT_AUDIT.md` (tick-level static T3R on 10+pt FVGs: +0.015 avgR baseline) to the full-population, pure-trailing configuration that the `BOS_FVG_RESEARCH.md` headline numbers relied on. CLAUDE.md has been correct on this since March 2026 — the research files and the HW vault entry were drifting behind it.

## Why it fails: [[bar-sim-trailing-bug]]

The `clean_backtest.py` simulator processes each post-fill bar in this order:
1. Check stop against `bar_low` (using OLD trail level)
2. Update `max_favorable_r` from `bar_high`
3. Tighten trail for next bar

When a bar's real tick sequence is "open → high → low → close," the tighter trail that would have been in place by the time price dropped to `bar_low` never gets checked. Winners that should have exited at the tighter intra-bar trail get carried forward intact. Over thousands of trades, this systematically inflates the trailing stop's apparent P&L.

Two variants of the fix were tested:
- **Intra-bar re-check** (tighten trail, then re-check stop against bar_low within the same bar): recovers only 0.5% of trades, +0.005 avgR delta. The ordering bug is mostly inert on 1-minute bars.
- **Full tick replay** (step through each tick, update trail and stop check in real time): recovers the full +0.29 avgR delta and produces the +0.001 avgR flat-zero ground truth.

The conclusion: you cannot reorder logic around the bug. The only correct fix is tick-level trade management. See [[bar-sim-trailing-bug]] for the structural explanation.

## What was invalidated

| Claim | Source | Status |
|---|---|---|
| +0.354 avgR / +13.37 R/day / Calmar 151 / 47.7% WR on 1min 24H | `BOS_FVG_RESEARCH.md` | **Invalidated.** Bar-sim artifact. |
| +0.895 avgR / Calmar 237 (clean_backtest.py + pure trail) | MEMORY.md Full Audit | **Invalidated.** Same script, same bug. |
| +1.141 avgR / Calmar 263.6 (V10z conservative 15S) | MEMORY.md V10y/V10z | **Invalidated.** 15s bars compress path less than 1m, same structural bug. |
| +0.880 avgR / Calmar 94.4 (V10z 4-signal portfolio) | MEMORY.md | **Invalidated** for the BOS_FVG leg. Other legs inherit the bug and need re-test. |
| 63.5% WR / +1.354 R/T / +15,368R (V2-corrected from [[comprehensive-mining-report-v2|V2 mining]]) | HW vault bos-fvg.md pre-2026-04-10 | **Invalidated.** V2 fixed [[v10i-look-ahead-bug|V10i alignment look-ahead]] but still ran on bar-level sim. |
| 97.9% / 98.8% / 90.6% WR V1-era alignment tiers | `STRATEGY_MINING_REPORT.md` | Previously flagged for V10i look-ahead. **Also invalidated** now for bar-sim inflation. |
| `bos_disp_*.csv`, `fvg_fill*_bos*.csv` grids, `full_year_all_trades.csv` | `results/` | **All suspect.** Bar-level sim outputs. Relative rankings may still carry signal information, but absolute R numbers are not trustworthy. |
| 10+pt static T3R baseline (+0.015 avgR) | `BOS_FVG_10PT_AUDIT.md` | **Still valid.** Was tick-level. It is the tick result that the rest of this file backfills for the trailing configuration. |

## What remains true

1. **The BOS_FVG signal generator itself is clean.** 3-bar swing detection with confirmation delay, BOS with ≥2pt displacement on first close-through of a confirmed level, FVG within 15 bars, limit at the far edge — all of this runs on completed bars and has no look-ahead.
2. **The signal produces a roughly break-even trade population.** Tick sim: +0.001 avgR on 842 trades, which is indistinguishable from zero given bootstrap noise.
3. **No subset (session, FVG size, direction, time-of-day, ATR regime) survives multiple-comparison correction at tick resolution.** The 10+pt audit already established this for the static variant; the trailing variant is no better.
4. **The NinjaTrader engine-native implementations are not affected** — they run on the platform's tick-level execution, not on bar OHLC. SweepBreakv17.cs and the v24 IFVG MTF cascade remain the only trustworthy forward-performance sources in the Tempo system.

## Scripts

Both reproducers live at `/tmp/` (session-local) and are runnable with `~/Documents/trading-system/.venv/bin/python`:

- `bos_fvg_bug_reproducer.py` — isolates the intra-bar ordering variant (Mode A vs Mode B)
- `bos_fvg_bar_vs_tick.py` — paired bar-vs-tick trailing sim on the same signals (the canonical experiment)

They depend on Databento and numpy/pandas from the trading-system venv. Data directory: `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/`.

## Cross-reference

This summary is the vault-side pointer to `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md`, which is the authoritative project-side file. The project CLAUDE.md's "Critical Corrections" section has been consistent with the failure position since March 2026; this doc and the concept-page updates bring the research journal and the vault in line with it. The auto-memory (`~/.claude/projects/-Users-harrisonwillis/memory/MEMORY.md`) has also been updated; see `bos_fvg_failure_20260410.md` in that directory.
