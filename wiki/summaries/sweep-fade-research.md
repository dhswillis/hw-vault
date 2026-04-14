---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md
related:
  - wiki/summaries/sweep-cluster.md
  - wiki/maps/strategies-moc.md
  - wiki/concepts/wick-fade.md
tags: [summary, sweep-fade, tick-level, portfolio]
---

# Key Level Sweep Fade Research — Summary

> +10.43 R/day, Calmar 361.8, 100% winning weeks, 13/13 months positive across 299 days of tick data. Four-strategy portfolio (1H wick, round 50pt, PDH/PDL, VWAP sweeps).

## Key findings

Complete tick-by-tick sweep fade research across 299 trading days, 320+ configurations tested in 18 phases. Final production portfolio: 1H Wick Sweep (trail 1R/0.5R), Round 50pt Sweep (trail 1R/0.5R), Prev Day H/L Sweep (trail 1R/0.5R), and VWAP Sweep (2R fixed target). 3,910 total trades.

Robust under stress: survives 2pt entry slippage at +8.42 R/day (Calmar 200.7, still 100% WWR). Walk-forward H2 OOS (Calmar 211) beats H1 IS (Calmar 160) — performance improves in later data.

Entry is limit-order style at the key level price when price sweeps through and fails back. Stop at wick tip + 5pt buffer.

## Status / caveats

The trailing stop exit on three of four legs raises the [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] question. However, this research uses tick-by-tick simulation (not OHLC bars), so the bug may not apply. Verify whether the simulation iterates actual ticks or bar OHLC. See [[wiki/summaries/sweep-cluster|Sweep Cluster]] for the consolidated assessment.

## Source documents

- [[raw-sources/trading/sweep/SWEEP_FADE_RESEARCH]]
