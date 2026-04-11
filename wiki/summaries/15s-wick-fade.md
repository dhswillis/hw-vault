---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/sweep/15S_Wick_Fade_Findings.docx
related:
  - wiki/concepts/wick-fade.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/wickfade-complete.md
  - wiki/summaries/sweep-cluster.md
  - wiki/syntheses/research-arc-map.md
tags: [trading, wick-fade, tick-validated, canonical]
---

# 15S Wick Fade — Summary

> **TICK-LEVEL VALIDATED.** Unlike most of the Layer 2 mining corpus, this is a tick-level simulation on Databento tick data across 260 trading days. The strategy uses fixed stop + fixed target (no trailing), making it structurally immune to the [[bar-sim-trailing-bug|bar-sim trailing bug]]. Edge is real: 43% WR, $30.89/trade net of commissions. One of very few genuine Layer 2 wins.

## What the source is

`~/HW/raw-sources/trading/sweep/15S_Wick_Fade_Findings.docx` — **338 lines** (as converted via `textutil`), dated March 2026. A tick-level research report on a 15-second mean-reversion fade strategy for NQ futures, across 260 trading days (Feb 2025 – Feb 2026) using Databento tick data (312 files). This is a *different* strategy from [[wickfade-complete|WickFade 5m]] and should not be confused — both are "wick fade" in name but they operate on different timeframes, signal definitions, and parameters.

## Strategy definition

**15S Wick Fade:** enter at the **wick (high/low) of the previous 15-second candle** when the current candle breaks through it. This is a mean-reversion play — the bet is that a 15S bar that breaks the previous bar's extreme will reverse quickly.

**Recommended production configuration:**

| Parameter | Value |
|---|---|
| Timeframe | 15-second bars |
| Stop | 4.50 pt (or 6.75 pt for higher WR, 2.25 pt for smaller risk) |
| Target | 10 pt |
| Entry | At the wick (0 tick offset) |
| Window | 08:30–10:00 CT (core); 08:30–09:15 CT is the best sub-window |
| Flip on stop | **OFF** — destroys value |
| Direction | Both long and short |

## Headline results (260-day tick-level)

- **4.50pt stop:** 43% WR, **$30.89/trade net of commissions**, ~$1.27M gross P&L
- **6.75pt stop:** 52% WR, **$34.37/trade**, highest per-trade edge
- **2.25pt stop:** 31% WR, $26.37/trade (still profitable)
- **Best 15-min bucket:** 08:30–08:45 CT at $34.61/trade and 44.5% WR
- **Thursday outperformance:** 46.4% WR and $40.04/trade (vs ~42–43% WR other days)
- **92.4% snap-back rate** within the same 15-second candle
- **Median break past wick:** only 3.00 pt (12 ticks) — most breaks are small

## The bug history (important for methodology lessons)

The 15S wick fade went through three Pine Script versions that were each contaminated by different forms of OHLC ambiguity, showing the same pattern that killed BOS_FVG:

| Version | Flip Win Rate | Note |
|---|---|---|
| Pine v58 (original) | **84%** | Target-on-entry-bar bug: target check had no `is_entry_bar` guard, allowing phantom flip wins |
| Pine v59 (target fix) | 56% | Fixed the phantom-target bug |
| Pine v60 (flip fix) | 36% | Fixed the flip-entry-bar free-pass bug (flip had `is_entry_bar` protection but wasn't supposed to) |
| **Tick-level ground truth** | **20.4%** | Real result — the flip has no edge |

Even v60 (with both fixes) was still inflated by **structural OHLC bias** that only tick data could eliminate. The fade's entry-bar protection filters out weak breakouts, leaving only strong ones for the flip to inherit — a subtle form of selection bias that looks fine at the bar level.

**Lesson:** the 15S wick fade is the third strategy in the corpus (after [[bos-fvg|BOS_FVG]] and [[wickfade-complete|WickFade 5m]]) to document the same underlying pattern — **path-dependent exits cannot be backtested on OHLC bars, only on ticks**. The doc explicitly frames tick-level as "ground truth" and Pine Script as "systematically biased." Anyone reading only this doc would catch the [[bar-sim-trailing-bug]] pattern in March. It was sitting there for a month before the April BOS_FVG audit.

## Flip vs fade — the flip is dead

The flip (reversing the trade when the fade stops out) is the part that fails most badly in backtests:

| Metric | Fade | Flip |
|---|---|---|
| Trades (2.25pt stop, 10pt target) | 44,853 | 27,782 |
| Win rate | 31.7% | 20.4% |
| Avg R | +0.730 | +0.117 |
| Gross P&L | $1,350,000 | $147,000 |
| $/trade | $30.12 | $5.29 |

The 216-config flip meta-analysis tested 3 time windows × 2 fade stops × 3 flip stops × 3 flip targets × 4 max flip counts. **Zero of 216 configurations made the flip profitable.** Inverted R:R math kills it: at 1pt target / 5pt stop, you need > 83.3% WR just to break even, and no configuration achieves that.

**Production rule: run the fade alone, do not enable the flip.**

## Entry offset analysis

Entering at the wick exactly (0 tick offset) beats entering 2 or 4 ticks shy:

| Offset | Net P&L | $/Trade | WR |
|---|---|---|---|
| 0 ticks (at wick) | $1,434,000 | $31.98 | 43.2% |
| 2 ticks shy | $1,389,000 | $30.97 | 42.8% |
| 4 ticks shy | $1,336,000 | $29.79 | 42.5% |

The wick is the natural support/resistance level. Entering earlier gives up fills on trades that would have touched the wick but not gone further.

## Stop width comparison

Wider stops help — counterintuitive but consistent:

| Stop | WR | $/trade | Net P&L (260 days) |
|---|---|---|---|
| 2.25 pt | 31% | $26.37 | $1,185,000 |
| 4.50 pt | 43% | $30.89 | $1,268,000 |
| 6.75 pt | 52% | $34.37 | $1,296,000 |

**At 4.50pt and 6.75pt stops, the flip actively loses money** (−$26k and −$191k respectively). This confirms: wider is better for the fade, and the flip should be off entirely.

## Day-of-week breakdown (08:30–10:00 CT)

| Day | WR | $/trade |
|---|---|---|
| Monday | 42.2% | $27.90 |
| Tuesday | 42.8% | $29.72 |
| Wednesday | 42.6% | $29.02 |
| **Thursday** | **46.4%** | **$40.04** |
| Friday | 42.5% | $28.70 |

Thursday stands out by ~$10/trade. Working hypothesis: weekly economic data releases (jobless claims, GDP) create stronger mean-reversion conditions. Worth monitoring but not yet validated.

## Why this is NOT tagged suspect

1. **Tick-level throughout.** The doc's entire methodology is "run it against real Databento ticks, not OHLC bars." This is the only way to avoid the path-reconstruction bugs that killed BOS_FVG trailing.
2. **No trailing stops.** The recommended configuration is fixed stop + fixed target, which is structurally bar-sim-safe (though it doesn't need to be because it's tick-level anyway).
3. **Flip variant explicitly rejected.** The one part of the strategy that had bar-sim contamination (the flip) was identified, audited, and cut from the production spec.
4. **260-day sample with Thursday stratification.** Large enough to defeat most random-sampling effects.
5. **Cost-adjusted headlines.** The $30.89/trade number is net of commissions ($4.50 round-trip).

## Remaining caveats

1. **Slippage not stress-tested.** The doc notes results are gross of slippage; real fills may differ in fast markets.
2. **Commission assumption:** $4.50 RT. Actual Rithmic rates may be different.
3. **260-day window = one regime.** 2025–2026 was an active period for NQ. A prolonged low-vol regime could change the edge.
4. **Trade frequency is high:** 55–66 trades/day at the best window, 170+/day across the full 08:30–10:00 window. Real execution needs sub-second latency.
5. **Not cross-validated on a second engine.** Only Python/Databento — no NinjaTrader or Nautilus confirmation yet.

## Scripts referenced

- `tick_fade_flip_backtest.py` — core tick-level fade + flip comparison
- `tick_offset_backtest.py` — 0 / 2 / 4 tick entry offset comparison
- `tick_stop_width_backtest.py` — 2.25 / 4.50 / 6.75 pt stop comparison
- `tick_flip_meta3.py` — 216-config inverted R:R flip meta-analysis
- `tick_break_same_bar.py` — same-bar break excursion + snap-back analysis
- `tick_fade_time_of_day.py` — time-of-day and DOW analysis

## Cross-references

- [[wick-fade]] — concept page for the wick-fade family of strategies
- [[wickfade-complete]] — the parallel 5m WickFade research (different strategy, same family)
- [[bar-sim-trailing-bug]] — the bug class this doc identifies (and avoids) a month before BOS_FVG
- [[sweep-cluster]] — the cluster summary that pointed at this doc as TBD
- [[research-arc-map]] — where this fits in the Tempo research arc (Layer 2 clean)
