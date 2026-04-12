---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/V5_Strategy_Bug_Audit_2026-03-23.md
related:
  - wiki/concepts/v10i-look-ahead-bug
  - wiki/entities/ninjatrader-v5.md
  - wiki/summaries/v5-strategy-bug-audit.md
tags: [trading, bug, ninjatrader, vwap, live-divergence, critical]
---

# VWAP Double-Counting Bug (NinjaTrader OnEachTick)

A critical bug pattern in [[ninjatrader-v5|NinjaTrader strategies]] when VWAP is computed inside an `OnEachTick` strategy without bar-gating. Identified during the [[v5-strategy-bug-audit|V5 strategy bug audit]] on 2026-03-23.

## The pattern

```csharp
// WRONG — runs on every tick in live, once per bar in backtest
vwapCumPV  += Close[0];
vwapCumVol += 1;
currentVWAP = vwapCumPV / vwapCumVol;
```

With `Calculate = Calculate.OnEachTick`:

- **Backtest (historical replay):** ~390 updates per day (once per 1-min bar, 9:30–16:00)
- **Live:** thousands per day (once per tick, ~50–200× the backtest rate)

The `vwapCumVol` accumulator grows at completely different rates. Live VWAP becomes a rolling average of ticks, backtest VWAP is a rolling average of bar closes. They are not the same value.

## Impact measured on V5 strategies

| Strategy | Backtest WR | Live WR (Sim101, 02/27–03/23) |
|---|---|---|
| ES VWPM | 66% | **44.4%** |
| R/Day (ES VWPM) | +0.66 | **−0.292** |

The audit identifies Bug 1 (this one) as the primary cause. The Python reference backtester iterates `for i in range(len(pr))` — once per minute bar — confirming the intended model is bar-close VWAP, not tick VWAP.

## The fix

Track the last bar index and only accumulate on new bars:

```csharp
private int lastVwapBar = -1;

// Inside OnBarUpdate, replace the VWAP block:
if (tod >= rthStart && tod < rthEnd && CurrentBar != lastVwapBar)
{
    lastVwapBar = CurrentBar;
    vwapCumPV  += Close[0];
    vwapCumVol += 1;
    currentVWAP = vwapCumPV / vwapCumVol;
}
```

Same fix applies to the `UseVolumeVWAP = true` branch.

## Why this class of bug is easy to miss

- Backtest and live use the same code path
- Unit tests on backtest data produce "correct" numbers
- Live divergence is slow to diagnose — requires weeks of live data to notice WR drift
- `OnEachTick` is the correct mode for managing in-flight orders, so switching back to `OnBarClose` isn't an option
- The fix is a 2-line gating check that's trivial to forget

## Detection heuristic

If you use `Calculate = Calculate.OnEachTick` in a NinjaTrader strategy, any accumulator that should update once per bar (VWAP, running delta, candle-close indicators) **must** be guarded with `CurrentBar != lastXBar`. Grep the codebase for `Calculate.OnEachTick` and audit every `+=` / accumulator operation that follows.

## Blast radius in Tempo

All three V5 strategies used the same VWAP block without gating: `V5NQVWPMStrategy`, `V5ESVWPMStrategy`, `V5NQLF1170Strategy`. The audit recommends fixing Bug 1 **first** in the priority order because it's the root cause of live/backtest divergence across the whole SF portfolio.
