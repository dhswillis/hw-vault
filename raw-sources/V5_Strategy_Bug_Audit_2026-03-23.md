# V5 Strategy Bug Audit — 2026-03-23

**Strategies audited:** V5NQVWPMStrategy, V5ESVWPMStrategy, V5NQLF1170Strategy
**Source:** `/strategies/03-sf-portfolio/ninjatrader/`
**Python reference:** `es_vwpm_260d.py`, `detailed_stats_260d.py`, `portfolio_clean_260d.py`
**Live data:** NinjaTrader Sim101, 02/27–03/23/2026

---

## BUG 1 — CRITICAL: VWAP double-counting in live mode

**Affects:** All 3 strategies
**Severity:** CRITICAL — causes live/backtest divergence

With `Calculate = Calculate.OnEachTick`, the VWAP accumulation fires on **every incoming tick** in real-time, but only **once per bar** during historical processing:

```csharp
// This runs once/bar in backtest, ~50-200x/bar in live
vwapCumPV  += Close[0];
vwapCumVol += 1;
currentVWAP = vwapCumPV / vwapCumVol;
```

- **Backtest:** ~390 data points per day (one per 1-min bar, 9:30–4:00)
- **Live:** Thousands of data points per day (every tick)

The VWAP value will be fundamentally different in live. This shifts entry signals, and is the most likely cause of the **ES VWPM live WR of 44.4% vs backtest 66%**.

The Python backtester iterates once per minute bar (`for i in range(len(pr))`), confirming the intent is bar-close VWAP.

**Fix:** Track bar index and only accumulate VWAP on new bars:

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

**Note:** This also affects the volume-weighted branch (`UseVolumeVWAP = true`). Same fix applies.

---

## BUG 2 — CRITICAL: No protective stop orders in the market

**Affects:** All 3 strategies
**Severity:** CRITICAL — no crash protection

Stops and targets are managed purely in code via `ManageTrade()`. No actual stop-loss or take-profit orders are submitted to the exchange. If NinjaTrader disconnects, crashes, or the PC loses power mid-trade, the position sits in the market with **zero protection**.

```csharp
// Current: code-level stop only
if (-fav >= StopPts)
    CloseTrade(-StopPts, p, "STOP");  // submits market exit
```

**Fix:** After entry, submit a protective stop and target order:

```csharp
if (dir == 1)
{
    EnterLong(Qty, "NQ_VWPM");
    SetStopLoss("NQ_VWPM", CalculationMode.Ticks, StopPts / TickSize, false);
    SetProfitTarget("NQ_VWPM", CalculationMode.Ticks, (StopPts * TargetR) / TickSize);
}
```

Or at minimum, submit a protective stop-market order as backup.

---

## BUG 3 — HIGH: Chart stats tag collision (NQ chart)

**Affects:** V5NQVWPMStrategy + V5NQLF1170Strategy
**Severity:** HIGH — misleading live dashboard

Both strategies use `"V5Stats"` as the `DrawTextFixed` tag:

```csharp
// In BOTH strategies:
Draw.TextFixed(this, "V5Stats", text, TextPosition.TopRight, ...);
```

When both run on the same NQ chart, the **LF1170 overwrites the NQ_VWPM stats**. This is confirmed by your screenshot: the NQ chart shows `Trades: 3 | WR: 100.0% | PF: 0.00` which is the LF1170's cumulative stats (3 total trades), not the NQ_VWPM stats.

**You are flying blind on NQ_VWPM performance in live.**

**Fix:** Use unique tag names:

```csharp
// V5NQVWPMStrategy:
Draw.TextFixed(this, "NQ_VWPM_Stats", text, ...);

// V5NQLF1170Strategy:
Draw.TextFixed(this, "NQ_LF1170_Stats", text, ...);
```

---

## BUG 4 — MODERATE: Profit Factor shows 0.00 when all trades win

**Affects:** All 3 strategies
**Severity:** MODERATE — confirmed on NQ chart display

```csharp
double pf = grossLoss > 0 ? grossWin / grossLoss : 0;
```

When grossLoss = 0 (all winners), PF displays 0.00 instead of ∞ or N/A. Confirmed in screenshot: `WR: 100.0% | PF: 0.00`.

**Fix:**

```csharp
double pf = grossLoss > 0 ? grossWin / grossLoss : (grossWin > 0 ? 999.0 : 0);
// Or display "N/A" when grossLoss == 0
```

---

## BUG 5 — MODERATE: Virtual P&L diverges from actual fills

**Affects:** All 3 strategies
**Severity:** MODERATE — stats tracking inaccurate

The strategy tracks P&L using virtual entry prices:

```csharp
entryPrice = price + VSlip * dir;  // virtual entry
```

But real orders fill at market. No `OnExecutionUpdate` handler reconciles the actual fill price. The virtual stop/target checks run against `entryPrice`, while the real position's P&L differs by slippage.

Additionally, when `ManageTrade` triggers a stop exit, it records P&L as exactly `-StopPts`:

```csharp
CloseTrade(-StopPts, p, "STOP");  // always records exact stop loss
```

But the real market exit via `ExitLong`/`ExitShort` may fill at a different (worse) price.

**Fix:** Add `OnExecutionUpdate` to track real fills, or at minimum log the divergence.

---

## BUG 6 — MODERATE: Session-flat uses wrong bar price

**Affects:** All 3 strategies
**Severity:** MODERATE

Session-close flattening triggers when a **new date is detected**:

```csharp
DateTime today = barTime.Date;
if (today != lastSessionDate)
{
    if (inTrade)
    {
        double fav = tradeDir * (p - entryPrice);
        CloseTrade(fav, p, "SESSION_FLAT");
    }
    ResetDay();
}
```

The problem: `p = Close[0]` at this point is the **first bar of the NEW session**, not the last bar of the prior session. If the overnight gap is large, the SESSION_FLAT P&L will include the gap. The real position would have been closed at the prior session's actual close price (if using NinjaTrader's session-end handling) or held overnight.

**Fix:** Use `Close[1]` (prior bar's close) for the session-flat P&L, or set `IsExitOnSessionCloseStrategy = true` and handle the actual fill.

---

## BUG 7 — LOW: Side establishment lost during trade (VWPM only)

**Affects:** V5NQVWPMStrategy, V5ESVWPMStrategy
**Severity:** LOW — by design, but diverges from Python

When in a trade, `if (inTrade) return;` (line 258) skips the side-establishment block. After the trade closes (with `establishedSide = 0`), the strategy must wait for price to move ≥ threshold from VWAP again before another side is established.

In the Python backtest, the side tracking continues during a trade because the loop iterates every bar regardless of trade state. This means the Python can enter a new trade sooner after an exit.

**Impact:** Fewer trades in live than backtest expected. May explain some of the signal divergence.

---

## OBSERVATION: ES VWPM live performance degradation

From the ES chart: **9 trades, 44.4% WR, R/Day: -0.292, PF: 0.68**
Backtest reference: **66% WR, +0.66 R/Day**

The most likely cause is Bug 1 (VWAP tick-counting). The live VWAP is computed from every tick, producing a smoother/different average than the bar-close VWAP used in backtesting. This shifts both the side-establishment signals and the touch-zone triggers.

Secondary contributor: Bug 7 (side establishment paused during trades) could reduce trade frequency and miss favorable re-entries.

---

## Priority fix order

1. **Bug 1** — VWAP bar-gating (fixes backtest/live divergence)
2. **Bug 3** — Chart tag collision (immediate visibility fix)
3. **Bug 2** — Protective stop orders (risk management)
4. **Bug 6** — Session-flat pricing
5. **Bug 4** — PF display
6. **Bug 5** — Virtual/real fill reconciliation
7. **Bug 7** — Side tracking during trade

---

*Audit performed against V5 source code and live NinjaTrader Sim101 trades from 02/27–03/23/2026.*
