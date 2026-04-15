---
created: 2026-04-14
updated: 2026-04-14
type: concept
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE.md
related:
  - wiki/summaries/trading-system-start-here.md
  - wiki/entities/tempo-trading-system.md
  - wiki/concepts/ifvg.md
  - wiki/maps/strategies-moc.md
tags: [trading, signal, nq, or-fail, production, v13]
---

# OR_FAIL (Opening Range Failure)

A 5-minute bar opening range failure strategy that is the backbone of the V13 DUAL OR production system. Independently audited (V13h) and walk-forward validated across all 12 months.

## The core bet

After the opening range (9:30-9:45 AM ET, first 15 minutes), if price breaks above the OR high then closes back below, SHORT at the next bar open targeting the OR midpoint. The inverse for longs, but only when OR delta is bearish (delta filter).

## Why this works

The opening range captures the initial supply/demand balance. A failure (break then reversal) signals that the breakout was a liquidity grab — trapped longs/shorts provide fuel for the reversal. The 5M timeframe produces natural 6-15pt stops (vs 1M's 2-4pt stops), putting commission at 5-17% of risk instead of 25-50%.

## Production spec (V13g)

- Trades/year: 164
- Win rate: 89.0%
- Total R: +65.6R
- R/day: +0.26
- Calmar: 22.9
- Max drawdown: 2.9R
- Max consecutive losses: 2
- SHORT: 109 trades, 90.8% WR
- LONG (delta-filtered): 55 trades, 85.5% WR
- Trail: 0.3R trigger, 30% of max favorable

## Extended variants (V13k-V13t)

**SHARPSHOOTER (OR_FAIL 3x re-entry):** 737 trades, 83.9% WR, 0.94 R/day, Calmar 59.5. Third entries at 85.2% WR — better than second entries.

**DUAL_OR (OR_FAIL + OR_BREAK_CONTINUE):** 1,048 trades, 83.1% WR, 1.20 R/day, Calmar 50.3. Every month green. Walk-forward H1 +180.8R, H2 +151.6R.

## Complementary concepts found

- IB_FAIL (30-min IB failure): 174 trades, 86.8% WR, +54.2R
- SESSION_EXTREME (PM reversal): 259 trades, 81.1% WR, +32.1R
- PREV_CLOSE_BOUNCE: 124 trades, 83.9% WR, +27.6R
- MICRO_SWEEP: DEAD after look-ahead fix (V13o)

## Bias audit (V13h)

8/10 checks passed. No look-ahead detected. Entry timing, stop/target priority, OR computation all correct. Edge degrades with delay (+1 bar: 82.4% WR vs 89.0%) — legitimate signal. Survives 3x commission (77.4% WR at 3pt). Random entries with same trail = 79% WR (trail asymmetry expected), but OR_FAIL adds +10pp WR AND 2.6x avg R per trade.

## Regime stability (V13i)

All 4 quarters profitable. All 5 days-of-week profitable. 100% of 1000 Monte Carlo subsamples profitable. Max 2 consecutive losses. Profit factor: 4.2.

## Relationship to IFVG + Lumi

OR_FAIL is a completely independent system from the [[ifvg|IFVG]] + [[lumi-strategy-spec|Lumi]] V15 portfolio. Different entry logic, different timeframes, different backtester codebase. The two systems are non-correlated and could potentially be combined, but this has not been tested.

## Cross-references

- [[trading-system-start-here]] — master context doc containing full V13 progression
- [[invalidation-rules]] — OR_FAIL is NOT contaminated by any known bugs (V13h audit)
- [[maps/strategies-moc]] — strategy status tracker
