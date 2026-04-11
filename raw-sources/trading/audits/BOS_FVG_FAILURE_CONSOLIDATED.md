# BOS_FVG — Consolidated Failure Report

**Date**: 2026-04-10
**Status**: DEAD. Do not trade. Do not cite as validated.
**Supersedes**: `BOS_FVG_RESEARCH.md`, `sf_portfolio_bug_and_fix.md` (BOS_FVG sections), HW vault `wiki/concepts/bos-fvg.md` (pre-2026-04-10 claims)

---

## TL;DR

Every previously reported "validated" BOS_FVG result — `+0.895 avgR`, `Calmar 263.6`, `+13.37 R/day`, `+1.141 avgR`, `Calmar 237` — was produced by **1-minute bar-level simulation**. When the identical signals are replayed against tick data with the same trailing stop rules, the edge collapses to **+0.001 avgR**. It is not tradeable, and it never was.

This file replaces any prior claim that BOS_FVG "survived" or is the "core signal" of the Tempo system. It did not survive. CLAUDE.md has had the correct position since at least March 2026 (`BOS_FVG DOES NOT WORK`); the research files and the HW vault entry were drifting behind it.

---

## The Canonical Number

**Fresh reproduction, 2026-04-10**, 60 days sampled across the 312-day Databento NQ tick dataset (Feb 2025 – Feb 2026), identical signal logic (3-bar swing → BOS ≥ 2pt displacement → FVG within 15 bars → limit at FVG far edge), identical trailing (trigger 0.5R, buffer 0.1R), identical session filter (13:00–20:00 UTC):

| Simulation | n | avgR | WR | Total R | R / day |
|---|---|---|---|---|---|
| Bar-level (1min OHLC, clean_backtest.py logic) | 875 | **+0.3590** | 41.0% | +314.2 | **+5.24** |
| **Tick-level (ground truth)** | **842** | **+0.0010** | **66.9%** | **+0.8** | **+0.01** |

**Paired comparison on the 842 trades filled in both sims**:
- Bar avgR: +0.2936
- Tick avgR: +0.0010
- **Delta (bar – tick): +0.2926 R per trade**
- 36.1% of trades had bar > tick (inflation cases)
- 28.5% of trades had bar < tick (deflation cases)
- 35.4% exactly equal

**Net: 1-minute bar simulation inflates BOS_FVG trailing results by ~+0.29 avgR per trade. Eliminating the inflation kills the edge.**

Script: `/tmp/bos_fvg_bar_vs_tick.py` (see Appendix A). Reproducer for the subtler intra-bar ordering bug at `/tmp/bos_fvg_bug_reproducer.py`.

---

## Why the bar-sim inflates

A 1-minute bar collapses ~1,000 ticks into four numbers (O/H/L/C). The trailing-stop simulator has to guess the intra-bar path. `clean_backtest.py` does this:

```python
# For each bar after fill:
#   1. Check stop against bar_low (uses OLD trail)
#   2. Update max_favorable_r using bar_high
#   3. Tighten trail from new max_favorable_r
```

This reconstruction systematically favors the trailer in two ways:

1. **Locks in high-water marks that never happened in that order.** If a bar's high comes BEFORE the low in real time, price hits +2R, then collapses back through the initial stop. Tick sim exits at the initial stop (–1R). Bar sim says "max fav was 2R, tighten trail to 1.9R, bar low didn't hit 1.9R so we're still alive" — because bar_low was checked against the OLD stop, not the newly-tightened one.
2. **Lets max_favorable_r ratchet on bar highs that were already reversed tick-by-tick.** The bar says `high=X`, but by the time any trailing logic would have reacted to X, price had already moved away. The trail still ratchets to X–buffer on the bar sim and carries that forward.

I wrote two patched variants to isolate the two bugs:

**Variant A — intra-bar re-check** (after tightening trail, re-check stop against the same bar's low). Tested on the 60-day sample:
- Mode A (original): +0.3418 avgR on 858 trades
- Mode B (conservative re-check): +0.3372 avgR
- **Delta: +0.0046 avgR — tiny.** Only 4 of 858 trades (0.5%) were affected.

**Variant B — full tick replay** (the comparison above):
- Bar: +0.359 avgR
- **Tick: +0.001 avgR — delta +0.358 avgR.**

The conclusion is that the intra-bar ordering "bug" in the strict sense is mostly inert on 1-minute bars, because intra-bar ranges rarely exceed 0.5R on BOS_FVG sized FVGs. **The real problem is path reconstruction itself** — using OHLC to simulate a trailing stop on a strategy whose edge lives entirely in how the trail ratchets is structurally wrong. You cannot fix it by reordering the logic within the bar; you need tick data.

---

## What this invalidates

Each of these results was headlined in `MEMORY.md` or in `BOS_FVG_RESEARCH.md` or in the HW vault:

| Claim | Source | Status |
|---|---|---|
| "BOS_FVG 1min 24H: +0.354 avgR, +13.37 R/day, Cal 151" | BOS_FVG_RESEARCH.md §Executive Summary | **Invalidated.** Bar-sim artifact. Tick-replay: +0.001 avgR. |
| "clean_backtest.py BOS_FVG + pure trail: +0.895 avgR, Cal 237" | MEMORY.md Full Audit section | **Invalidated.** Same script, same bug. |
| "V10z conservative 15S trailing: +1.141 avgR, Cal 263.6" | MEMORY.md V10y/V10z | **Invalidated in practice.** 15S bars compress path less than 1min, but the same structural bug applies; the larger-than-1m number is a larger-than-1m artifact. CLAUDE.md has called this out since March 2026. |
| "V10z 4-signal portfolio: +0.880 avgR, Cal 94.4" | MEMORY.md | **Invalidated** for the BOS_FVG leg. Other signals (vol_spike_FVG, ORB_breakout, pre_gap_fill) inherit the same bar-sim bug and should be re-tested under tick replay before being trusted. |
| "BOS_FVG + MTF alignment: 97.9% WR, 98.8% WR, 90.6% WR" | HW vault bos-fvg.md V1-era section | **Already flagged suspect** in the vault for look-ahead on the alignment side. Now also invalidated for bar-sim inflation. |
| BOS_FVG "survived" as top performer at 63.5% WR / +1.354 R/T from V2 mining | HW vault bos-fvg.md V2-corrected section | **Invalidated.** V2 fixed the alignment look-ahead but still ran on bar-level sim. |
| `bos_disp_*.csv`, `fvg_fill*_bos*.csv` mining grids | `results/` | **All suspect.** Generated from bar-level sim. The relative rankings across the grid may still carry information about the SIGNAL distribution, but the R numbers are not trustworthy. |
| 10+pt static T3R audit: +0.015 avgR baseline | BOS_FVG_10PT_AUDIT.md | **Still valid as a tick-based result** — it was the one piece of analysis that ran tick-level. It correctly identified the edge as absent. It is the tick result that the rest of this file is now backfilling for the trailing-stop case. |

---

## What is actually true about BOS_FVG

These are the findings that remain defensible after the tick audit:

1. **The signal generator (3-bar swing → BOS ≥ 2pt → FVG → limit at far edge) produces a population of roughly break-even trades.** Tick sim says +0.001 avgR on 842 trades. That is indistinguishable from zero given bootstrap noise — the 10+pt audit's 95% CIs confirm this for the static-target variant.
2. **The 47–67% WR reported in various places is real**, in the sense that it's what the sim (any sim) reports — but on tick replay the WR rises (66.9%) specifically because the trailing stop closes most trades near break-even. It is a WR without a paycheck.
3. **The signal is not structurally broken.** It fires where you'd expect, in the right direction, at the right level. It is simply that once you account for intra-bar path, the trailing stop's average R converges to zero. This is consistent with what CLAUDE.md has said since March 2026.
4. **There is no subset (session, FVG size, direction, day-of-week, ATR regime) that survives multiple-comparison correction** at tick resolution. The 10+pt audit enumerated 25 filter combos; every 95% bootstrap CI contained zero.

---

## What to do about it

### Immediate
1. Treat any Python script in `~/Documents/` or `~/Documents/trading-system/` that simulates trailing stops on OHLC bars as **suspect by default**. This includes:
   - `clean_backtest.py`
   - `strategies/03-sf-portfolio/backtests/clean_backtest.py`
   - `bos_fvg_full_audit.py` static sections (the trailing section already runs tick-level — that one is fine)
   - All `/tmp/mine_*.py`, `/tmp/ifvg_optimize_v*.py` files that trail on bars
2. Do not cite any `+X.XXX avgR` / `Calmar` number from a bar-level trailing simulation as evidence of edge in `START_HERE.md`, NQ Test.xlsx, or the research journal. If a number is needed, re-run it against tick data.
3. The NinjaTrader `SweepBreakv17.cs` and the live-sim v24 IFVG MTF cascade do NOT have this problem (they run on the platform's engine with real tick-level execution). Those are still the only trustworthy sources of forward performance.

### Structural
1. Delete or quarantine the BOS_FVG result CSVs in `results/` that are bar-sim products. Suggested: move them to `results/archive/bar-sim-pre-2026-04-10/` so git history stays intact but they're not accidentally cited.
2. Update `HW/wiki/concepts/bos-fvg.md` with the `suspect-results` tag and a link forward to this file.
3. Update `MEMORY.md` "What Actually Works" section to reflect that BOS_FVG is in the same category as V10a-V10n alignment and doji-break — a dead lead with contaminated prior evidence.
4. Any future backtester intended for serious conclusions should either:
   - Run at tick resolution, OR
   - Use a target-based exit (fixed R) rather than a trailing stop, since fixed targets are robust to intra-bar path.

### For the NinjaTrader port
The `SweepBreakv17.cs` implementation is fine — it runs on the engine, not on bar OHLC. But if anyone is tempted to read the bar-sim numbers in this folder and expect them in sim/live: don't. Expect the tick numbers. The tick numbers for BOS_FVG are zero.

---

## Appendix A — Reproducer

Two scripts in `/tmp/`:

**`/tmp/bos_fvg_bug_reproducer.py`** — isolates the intra-bar ordering variant. Mode A is `clean_backtest.py`'s order of operations; Mode B re-checks the stop against bar_low after the trail has tightened on the same bar. Result: +0.005 avgR delta, only 0.5% of trades affected. The ordering bug is real but small.

**`/tmp/bos_fvg_bar_vs_tick.py`** — generates the same signals from 1-min OHLC and runs two sims: bar-level trailing (identical to `clean_backtest.py`'s `simulate_trade` function) and tick-level trailing (step through each tick, updating max_fav, trail, stop check in real time). On 60 sampled days and 842 paired trades: bar = +0.293 avgR, tick = +0.001 avgR. This is the canonical number for the file.

Both scripts load data from `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/` using the project venv `~/Documents/trading-system/.venv/bin/python`. They are self-contained and do not depend on the `intraday_scanner` / `micro_structure` machinery (which has its own known phantom B/E bug).

---

## Appendix B — The bar-sim bug in one paragraph, for future reviewers

A trailing stop exits a trade at a price that depends on the path of the underlying, not just the range. When you simulate a trail on 1-minute OHLC, you are choosing an ordering of the bar's high and low. `clean_backtest.py` chooses "check stop at bar_low using the stop level from the PREVIOUS bar's close; then ratchet trail from bar_high." This implicitly assumes low-then-high, i.e. the winner gets to catch the bar's extreme without first being stopped by a tighter trail. In reality, on about a third of BOS_FVG trades, the sequence is high-first-then-low, and the tighter trail would have triggered within the same bar. A conservative fix that re-checks against bar_low after tightening only recovers ~0.5% of trades because most 1-minute BOS_FVG bar ranges are smaller than the trail's motion for that bar. The only correct fix is to replay the ticks. When you do, the edge disappears. **Do not trail on bars.**
