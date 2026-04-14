---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources: [raw-sources/V5_Strategy_Bug_Audit_2026-03-23.md]
related:
  - wiki/entities/ninjatrader-v5.md
  - wiki/concepts/vwap-double-counting-bug.md
tags: [trading, ninjatrader, audit, bugs, live-divergence]
---

# V5 Strategy Bug Audit — 2026-03-23

**Source:** `raw-sources/V5_Strategy_Bug_Audit_2026-03-23.md`
**Strategies audited:** V5NQVWPMStrategy, V5ESVWPMStrategy, V5NQLF1170Strategy
**Code path:** `/strategies/03-sf-portfolio/ninjatrader/`
**Live data:** NinjaTrader Sim101, 02/27–03/23/2026
**Python reference:** `es_vwpm_260d.py`, `detailed_stats_260d.py`, `portfolio_clean_260d.py`

Seven bugs found in [[ninjatrader-v5]] strategies during live-vs-backtest divergence investigation. Two are CRITICAL and collectively explain the ES VWPM live WR of 44.4% vs backtest 66%.

## Bug 1 — CRITICAL: VWAP double-counting in live mode

**Affects:** all 3 strategies
**Cause:** `Calculate = Calculate.OnEachTick`. The VWAP accumulator fires on every incoming tick in real-time but only once per bar in historical replay.

- Backtest: ~390 data points per day (one per 1-min bar, 9:30–4:00)
- Live: thousands per day (one per tick)

This produces a fundamentally different VWAP in live, shifting entry signals. The Python backtester iterates once per minute bar — confirming the intended model is bar-close VWAP. See [[vwap-double-counting-bug]] for the full pattern.

**Fix:** track `lastVwapBar` and only accumulate on new bars:
```csharp
if (tod >= rthStart && tod < rthEnd && CurrentBar != lastVwapBar) {
    lastVwapBar = CurrentBar;
    vwapCumPV  += Close[0];
    vwapCumVol += 1;
    currentVWAP = vwapCumPV / vwapCumVol;
}
```
Same fix applies to the `UseVolumeVWAP = true` branch.

## Bug 2 — CRITICAL: No protective stops on the exchange

**Affects:** all 3 strategies
Stops and targets are managed purely in code via `ManageTrade()`. No real stop-loss or take-profit orders exist on the exchange. If NinjaTrader disconnects, crashes, or the PC loses power mid-trade, the position sits in the market with **zero protection**.

**Fix:** after entry, submit `SetStopLoss` and `SetProfitTarget` so there's a real order backing every position.

## Bug 3 — HIGH: Chart stats tag collision

**Affects:** V5NQVWPMStrategy + V5NQLF1170Strategy (same NQ chart)
Both use `"V5Stats"` as the `DrawTextFixed` tag, so LF1170 overwrites the NQ_VWPM stats. Screenshot confirms the NQ chart shows LF1170's 3-trade stats rather than NQ_VWPM's.

**Fix:** use unique tag names (`NQ_VWPM_Stats`, `NQ_LF1170_Stats`).

**Impact:** user is flying blind on live NQ_VWPM performance until fixed.

## Bug 4 — MODERATE: Profit Factor displays 0.00 on all-winners

`double pf = grossLoss > 0 ? grossWin / grossLoss : 0;` — when all trades win, PF reads 0.00 instead of ∞/N/A. Confirmed on chart screenshot (`WR: 100.0% | PF: 0.00`).

**Fix:** return a sentinel like 999.0 or "N/A" when `grossLoss == 0`.

## Bug 5 — MODERATE: Virtual P&L diverges from actual fills

Strategy tracks P&L via `entryPrice = price + VSlip * dir` (virtual entry) but real orders fill at market. Stop exits record P&L as exactly `-StopPts` regardless of actual fill. No `OnExecutionUpdate` reconciliation.

**Fix:** add `OnExecutionUpdate` to capture real fill prices, or at minimum log the divergence.

## Bug 6 — MODERATE: Session-flat uses wrong bar price

Session-close flatten triggers on new-date detection and uses `Close[0]` — which is the first bar of the NEW session, including any overnight gap. Large gaps will be mispriced in the strategy's P&L.

**Fix:** use `Close[1]` or set `IsExitOnSessionCloseStrategy = true`.

## Bug 7 — LOW: Side establishment paused during trade (VWPM only)

`if (inTrade) return;` skips the side-establishment block during active trades, so after exit the strategy must wait for price to move ≥ threshold from VWAP again. Python backtest continues side tracking during trades — so Python re-enters sooner. Reduces live trade frequency.

## ES VWPM live vs backtest

- **Live:** 9 trades, 44.4% WR, R/Day -0.292, PF 0.68
- **Backtest:** 66% WR, +0.66 R/Day

Primary cause: Bug 1 (VWAP tick counting). Secondary: Bug 7 (paused side tracking).

## Fix priority

1. Bug 1 — VWAP bar-gating (fixes divergence root cause)
2. Bug 3 — chart tag collision (immediate visibility fix)
3. Bug 2 — protective stop orders (risk management)
4. Bug 6 — session-flat pricing
5. Bug 4 — PF display
6. Bug 5 — virtual/real fill reconciliation
7. Bug 7 — side tracking during trade

## Source documents

- [[raw-sources/V5_Strategy_Bug_Audit_2026-03-23]]
