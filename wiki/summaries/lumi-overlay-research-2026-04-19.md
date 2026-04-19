---
created: 2026-04-19
updated: 2026-04-19
type: summary
sources:
  - strategies/tempo/scripts/lumi_overlay_research.py
  - strategies/tempo/results/OVERLAY_RESEARCH_RESULTS.md
related:
  - wiki/concepts/leppyrd.md
  - wiki/concepts/smt.md
  - wiki/maps/strategies-moc.md
tags: [trading, lumi, overlay, research, nq, es]
---

# Lumi Overlay Research — Apr 19 2026

> Tested 6 market condition overlays on Lumi (NQ sweep-reversal strategy) using tick-level simulation across 280 days of NQ + ES tick data.

## Key Result

**ES momentum alignment** is the only overlay that materially improves Lumi. Trading only when ES intraday direction matches the Lumi trade direction lifts WR from 56.5% to 63.9% and is walk-forward validated (63% train → 65% test).

## Overlays Tested

| Overlay | Result | Status |
|---|---|---|
| **ES momentum alignment** | +7.4% WR, Calmar 39.7 | **CONFIRMED — implement** |
| Skip NY_AM longs | +1.6% WR, free improvement | **CONFIRMED — implement** |
| Session filtering | NY_Lunch + NY_PM best | Informational |
| Weekday (Mon best, Fri worst) | +10.5% WR spread | Informational |
| [[leppyrd]] bias alignment | No edge (54.7% aligned vs 57.5% against) | **REJECTED** |
| PDClose position | Overfit (78% on 30d → 49.7% on 280d) | **REJECTED** |
| [[smt]] divergence (15m) | Too few signals, 1H produced zero | **REJECTED** |
| Prev day trend alignment | Counter-trend slightly better | Informational |

## ES Momentum Filter Detail

- **Rule**: At Lumi entry time, check if ES is trending up (open > prior close proxy) or down. Only take trades in the ES direction.
- **574 trades** (vs 1102 baseline) — roughly half pass the filter
- **63.9% WR**, +0.615 avgR, +1.26 R/d, 8.0R MDD
- **Every month positive** over 12 months (Mar 2025 – Feb 2026)
- **Walk-forward**: 63% train → 65% test (improves OOS)
- **Avg winner +1.391R, avg loser -0.760R** (1.83 ratio)
- **73% of trading days profitable**

## Why Leppyrd Fails on Lumi

[[leppyrd]] bias works for strategies that trade WITH the daily trend (like SF engulfing). Lumi is inherently a sweep-reversal strategy — it enters when price sweeps a level and reverses. Aligning with the daily bias actually filters out the exact reversal setups Lumi is designed to catch. Against-bias trades (57.5% WR) outperformed aligned trades (54.7% WR).

## Why SMT Failed

The [[smt]] engine detected zero divergences on the 1H timeframe across 280 days — the swing lookback of 3 bars is too strict for intraday data where swings form and resolve within the same session. The 15m SMT produced some signals (222 across 280 days) with mild directional value (aligned 55.3% vs against 50.5%) but not enough to filter on.

## Implementation Notes

The ES momentum check is simple: at Lumi entry time, compare ES session open to current ES price. If ES is up, only take longs. If ES is down, only take shorts. This can be implemented in the NinjaTrader C# strategy by adding an ES data series and checking intraday direction.

## Data

- Engine: `strategies/tempo/scripts/lumi_overlay_research.py`
- Full results: `strategies/tempo/results/OVERLAY_RESEARCH_RESULTS.md`
- Trade log: `strategies/tempo/results/lumi_overlay_20250306_20260211_trades.csv`
