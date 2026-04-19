---
created: 2026-04-19
updated: 2026-04-19
type: concept
sources:
  - strategies/03-sf-portfolio/python/leppyrd_v4.py
related:
  - wiki/concepts/ifvg.md
  - wiki/maps/strategies-moc.md
tags: [trading, bias, filter]
---

# Leppyrd Bias

A daily directional bias framework based on the relationship between the current day's close and the previous day's high/low (PDH/PDL).

## Logic

Given yesterday's session range (PDH, PDL) and today's price action:

| Condition | Bias |
|---|---|
| Close > PDH | Bullish |
| Close < PDL | Bearish |
| Swept PDH, closed inside range | Bearish (reversal) |
| Swept PDL, closed inside range | Bullish (reversal) |
| Close inside range, no sweep | Neutral |

The sweep-reversal cases are the most interesting — price took out a key level and failed, implying institutional reversal.

## H4 Swing Enhancement

Secondary confirmation via H4 candle structure:
- H4 sweep of prior H4 high → bearish
- H4 sweep of prior H4 low → bullish

## Implementation

Canonical implementation: `leppyrd_v4.py` (611 lines). Contains a **known look-ahead bug** in the `sf_scan_window()` function — uses current candle's final close data to check engulfing, but entry happens during the candle. See file header for details.

The bias computation itself (PDH/PDL comparison) is **not affected** by the look-ahead bug — only the SF engulfing scan is contaminated.

## Tested On

### SF Portfolio — WORKS
Leppyrd bias alignment improves SF engulfing strategies. The filter selects trades where the sweep-fail direction matches the daily bias. See [[sf-portfolio-cluster]].

### Lumi — DOES NOT WORK
Tested April 2026 on 280 days (1,102 Lumi trades):
- Aligned trades: 54.7% WR, +0.390 avgR, 25.7R MDD
- Against trades: 57.5% WR, +0.492 avgR (actually better)
- Baseline: 56.5% WR

Lumi is a sweep-reversal strategy — it inherently trades against the prior move. Leppyrd alignment (which favors trend continuation) conflicts with this. See [[lumi-overlay-research-2026-04-19]].

## Key Takeaway

Leppyrd works for strategies that trade WITH the daily bias (like SF engulfing). It does NOT work for sweep-reversal strategies (like [[lumi-strategy-spec|Lumi]]) where the entry is inherently counter-trend at the intraday level.
