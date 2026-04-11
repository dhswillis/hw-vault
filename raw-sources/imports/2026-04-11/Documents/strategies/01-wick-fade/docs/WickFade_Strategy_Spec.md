# WickFade Strategy — Master Specification

**Last Updated:** 2026-03-02 (v11)
**Purpose:** Persistent reference for all Claude sessions working on this strategy. READ THIS FIRST before modifying any WickFade code.

---

## 1. Strategy Concept

Fade the breakout of the previous bar's high or low. When price breaks above the prior bar's high, short at that high (expecting it to reverse back down). When price breaks below the prior bar's low, go long at that low (expecting it to bounce). If the fade is stopped out, FLIP direction (the stop-out confirms momentum, so ride it).

**There is NO wick size filter.** Every bar during the entry window generates both a short fade (at prev high) and a long fade (at prev low). This is by design — the backtests validated this approach without any wick filter.

---

## 2. Tick-Validated Backtest Results (Databento, 235 days)

These are the CONFIRMED results from tick-level backtesting with real slippage/commission modeling.

### Best Configs — 1M Timeframe (from `wick_fade_multi_tf_results.json`)

| Config | Trades | WR | Total R | R/Day | Net $/Day | Fade WR | Flip WR |
|--------|--------|----|---------|-------|-----------|---------|---------|
| **S=1, T=5 (5R)** | 10,763 | 47.7% | 20,035R | 85.3 | $1,539 | 37.3% | 64.3% |
| S=1, T=4 (4R) | 10,649 | 50.7% | 16,366R | 69.6 | $1,227 | 39.3% | 69.6% |
| **S=2, T=10 (5R)** | 10,845 | 37.2% | 13,359R | 56.9 | $2,109 | 31.2% | 45.9% |
| S=1, T=3 (3R) | 10,514 | 54.2% | 12,274R | 52.2 | $958 | 42.0% | 74.2% |

### Deep Backtest — S=2, T=10 (from `wick_fade_1m_deep_summary.json`)

- **235 days, 10,664 trades, 45.4 trades/day**
- Win rate: 35.7% (fade: 29.2%, flip: 44.8%)
- Total R: +9,726.2 | Net: $341,062 ($1,451/day)
- 94% positive days | Max DD: -$1,641 | Worst day: -$1,281
- **Slippage modeled:** 0.25 base + 0.02 * vol_mult, 30% ambiguity stop bias
- **Commission:** $4.50 RT
- **Entry miss rate:** 5% (orders that wouldn't fill in real life)
- Fade accounts for 32% of net profit, Flip accounts for 68%

### Stop Sweep Results (`wick_fade_1m_stop_sweep.json`)

Best net $/day by stop size (all with T = 5x stop):
- S=2, T=10: $1,451/day (the validated config)
- S=2.5, T=10: $1,213/day
- S=3, T=10: $974/day

### 15S Timeframe Results (from `wick_fade_wide_stop_results.json`)

| Config | Trades/Day | WR | R/Day | Net $/Day |
|--------|-----------|-----|-------|-----------|
| S=2, T=10 | 159 | 26.1% | 89.7 | $3,029 |
| S=2, T=8 | 166 | 30.0% | 83.2 | $2,741 |
| S=2, T=6 | 171 | 35.0% | 68.7 | $2,134 |

---

## 3. Current NinjaTrader Implementation (WickFadeV11.cs)

### Version: v11 (as of 2026-03-02)

### **CRITICAL: SINGLE POSITION MODEL**

v11 matches the Databento tick backtest EXACTLY. The key architectural difference:
- **Databento backtest:** `if not t_on` — must be FLAT before entering. Single position at a time.
- **Pine Script v75:** `array.push(positions, ...)` — MULTI-position. Was NOT tick-validated.
- **v11 follows Databento.** One position (`_currentPos`), must be null (flat) before new entry.

### Parameters (current defaults = tick-validated)

| Parameter | Value | Notes |
|-----------|-------|-------|
| StopPts | 2 | Matches Databento |
| TargetPts | 10 | 5R — matches Databento validated config |
| TrailActivatePts | 5 | Trail on flip leg |
| TrailDeltaPts | 3 | Trail distance behind MFE |
| EntryStartTime | 830 | 8:30 AM CT |
| EntryEndTime | 930 | Backtest used full RTH; 830-930 is forward-test window |
| FlattenTime | 1455 | 2:55 PM CT |
| StopLimitBuffer | 0.5 | NT-specific: stop-limit gap detection |
| Qty | 1 | Contracts per trade |

### Entry Logic (NO wick filter, SINGLE POSITION)

On each tick during the entry window:
1. If NOT flat (`_currentPos != null`), skip — **single position only**
2. If already entered this bar (`_barBreakoutDone`), skip — **one entry per bar**
3. Check breakout:
   - If `price > prevBarHigh` → **SHORT Market order** (fade the breakout)
   - Else if `price < prevBarLow` → **LONG Market order** (fade the breakout)
4. Stop/target calculated from **actual fill price** (not from the level) in OnOrderUpdate

On each new bar:
- Cancel unfilled entry from previous bar (only one can exist)
- Record `High[1]` and `Low[1]` as new breakout levels
- Reset `_barBreakoutDone` flag

### Databento vs Pine Script vs NT comparison

| Aspect | Databento (validated) | Pine v75 | NT v11 |
|--------|----------------------|----------|--------|
| Positions | SINGLE (`if not t_on`) | MULTI (array.push) | SINGLE (`_currentPos`) |
| Entry type | At market price + slippage | Limit at level | Market order |
| Stop/target from | Actual entry price | The level (prev_h/prev_l) | Actual fill price |
| Entry per bar | One (else-if, short priority) | One (else-if) | One (`_barBreakoutDone`) |
| Max flips | 1 | 1 | 1 (fade→flip, flip stops = flat) |

### Exit Logic

- **Stop:** Stop-Limit order with 0.5pt buffer
  - LONG stop (Sell StopLimit): limit = stop - 0.5 (accept lower fill)
  - SHORT stop (BuyToCover StopLimit): limit = stop + 0.5 (accept higher fill)
- **Target:** Limit order at target price
- **No OCO:** Manual cancel — when one side fills, cancel the other
- **Flip:** On FADE stop-out only, spawn flip in opposite direction (Market order). Max 1 flip.
- **Flip stop-out:** Back to FLAT. No further flips.
- **Trail:** On flip legs only, when MFE >= 5pts, trail stop 3pts behind best price
- **Flatten:** Position closed at 14:55 CT. Back to flat.

### Safety Mechanisms (NT-specific)

1. **Naked position detection:** If position has no working stop or target, re-submit brackets
2. **Blown stop detection:** If stop level is already past market, market exit
3. **Stop-limit gap detection:** If price blows past the 0.5pt buffer, market exit
4. **Bracket rejection counter:** Max 3 rejects, then market exit
5. **Double-fill handler:** If both stop and target fill (race condition), flatten the overshoot
6. **Late-fill guard:** If cancelled entry gets a late fill, close immediately
7. **Config validation:** Blocks trading on non-1M charts, warns on bad parameters

---

## 4. Architecture (WickFadeV11.cs)

### Position Lifecycle (SINGLE POSITION)

```
FLAT (_currentPos == null)
  ↓ breakout detected
PendingEntry → [Market fill] → Live → [stop fills] → Closed → FLIP (new _currentPos)
                                    → [target fills] → Closed → FLAT
                                    → [flatten/safety] → Closed → FLAT
```

Only ONE position exists at a time. `_currentPos` is either null (flat) or points to the active trade.

### Key Classes

- **WickPos:** Position tracker with Id, Dir, EntryPrice, StopPrice, TargetPrice, EntryOrder, StopOrder, TargetOrder, PosState, PosLeg (Fade/Flip), BestPrice (MFE), TrailActive, BracketRejectCount
- **PosState:** PendingEntry, Live, PendingExit, Closed
- **PosLeg:** Fade, Flip

### Key Methods

| Method | Purpose |
|--------|---------|
| `OnBarUpdate()` | Each tick: manage live pos, check breakout. First tick of bar: cancel stale, update levels |
| `CheckBreakoutEntry()` | Every tick: if flat + breakout → submit Market entry |
| `SubmitFadeEntry()` | Market entry + create WickPos, set `_currentPos` |
| `SubmitFlipEntry()` | Market entry for flip leg, replace `_currentPos` |
| `OnOrderUpdate()` | Handle fills: entry → calc stop/target from fill, submit brackets. Stop → close + flip. Target → close → flat |
| `SubmitBracket()` | Place StopLimit + Limit target |
| `ManageLive()` | Every tick: safety net, gap detection, MFE tracking, trail management |
| `MarketExitPosition()` | Cancel brackets, market exit, record PnL, optionally spawn flip |
| `CancelStaleEntry()` | Cancel unfilled entry from previous bar |
| `FlattenAll()` | EOD flatten |
| `FlattenDoubleFill()` | Handle both-sides-filled race condition |

### Unmanaged Order Specifics

- `IsUnmanaged = true` — full order control, no ATM
- `Calculate = Calculate.OnEachTick` — tick-level management
- `SubmitOrderUnmanaged(barsInProgress, OrderAction, OrderType, qty, limitPrice, stopPrice, oco, signalName)`
- `ChangeOrder(order, qty, limitPrice, stopPrice)` for stop-limit modifications
- Signal names: `"FadeS_7"`, `"FlipL_8"`, `"SL_7"`, `"Tgt_7"`

---

## 5. File Locations

| File | Purpose |
|------|---------|
| `Documents/WickFadeV11.cs` | **PRIMARY** — copy to NT Strategies folder |
| `Documents/WickFadeV10.cs` | Backup of v10 (multi-position, breakout-confirmed) |
| `Documents/WickFadeV9.cs` | Backup of v9 (config validation added) |
| `Documents/WickFadeV8.cs` | Backup of v8 (T=10 fix, class rename) |
| `Documents/wick_fade_1m_deep_summary.json` | Tick-validated S=2/T=10 results |
| `Documents/wick_fade_1m_stop_sweep.json` | Stop size sweep results |
| `Documents/wick_fade_multi_tf_results.json` | Multi-timeframe comparison |
| `Documents/wick_fade_wide_stop_results.json` | 15S wide-stop results |
| `Documents/trading-system/wickfade_tick_backtest.py` | Tick-level backtester (Databento) |
| `Documents/trading-system/results/wickfade_tick_best.json` | 15S tick best results |
| `Documents/wick_fade_1m_v75.txt` | Pine Script v75 (1M multi-position, trail) |
| `Documents/wick_fade_15s_v53.txt` | Pine Script v53 (15S, analysis-informed) |

**IMPORTANT:** NinjaTrader compiles ALL .cs files in its Strategies folder (`Documents\NinjaTrader 8\bin\Custom\Strategies\`). Only `WickFadeLive.cs` should be in that folder. Version backups stay in the Documents root.

---

## 6. Known Issues & Bug History

| Issue | Root Cause | Fix | Version |
|-------|-----------|-----|---------|
| Naked positions (no brackets) | Entry filled but OnOrderUpdate didn't submit brackets | Added safety net in ManageAllLive() + full order state logging | v5 |
| "Buy stop below market" error | Safety net re-submitted stale stop when price had moved past it | Added blown-stop detection → market exit | v5 |
| Infinite bracket re-submit loop | Bracket rejection → re-submit → rejection cycle | Added BracketRejectCount, max 3 → market exit | v5 |
| "Sell stop above market" for flip | Flip used StopMarket but price was already at the stop level | Changed flip entries from StopMarket to Market orders | v7 |
| CS0101 compilation error | Both WickFadeLive.cs and _v7.cs in NT Strategies folder | Remove _v7.cs from NT folder (backup only) | v7 |
| NT cached old settings (T=20) | NT caches strategy defaults by class name | Changed class name to WickFadeV8 to force fresh defaults | v8 |
| Wrong chart type + wide window | User ran on tick chart with 6hr window → order overload | Added config validation (block non-1M, warn on wide window) | v9 |
| Entry at level, not breakout | NT pre-placed limit orders at bar open (touching level = fill) | Rewrote to breakout-confirmed entry (tick-by-tick detection) | v10 |
| Multi-position vs Databento | NT v10 was multi-position; Databento backtest was single-position | Rewrote to single position (`_currentPos`), Market entry | v11 |
| Tgt cancellation error in v10 | Multi-position: cancelling brackets on one pos conflicted with others | Single position eliminates multi-pos order conflicts | v11 |

---

## 7. Deployment Checklist

1. Copy `WickFadeV11.cs` to `Documents\NinjaTrader 8\bin\Custom\Strategies\`
2. Ensure NO other WickFade*.cs files are in that folder
3. Open NinjaScript Editor → F5 to compile
4. Open NinjaScript Output window (New → NinjaScript Output) BEFORE enabling
5. Apply to NQ 1-minute chart (chart timezone = US Central)
6. Set account to sim account (Sim101)
7. Enable strategy
8. Monitor Output window for order state logging

---

## 8. Open Questions / Future Work

- [x] **T=20 vs T=10:** RESOLVED in v8. Changed to T=10 to match Databento validated config.
- [x] **Single vs multi-position:** RESOLVED in v11. Switched to single position to match Databento.
- [x] **Entry logic:** RESOLVED in v10/v11. Breakout-confirmed Market orders, not pre-placed limits.
- [ ] **Entry window:** Tick backtest used full RTH. Pine v53/v75 used 830-930 CT. Need to validate 830-930 window specifically in tick backtest.
- [ ] **15S timeframe on NT:** 15S showed higher R/day in backtest ($3,029 vs $2,109 at S=2/T=10). Would need secondary data series in NT.
- [ ] **Wick filter testing:** The "confirmed" backtest (`wickfade_confirmed_best.json`) tested min_wick 1-2pt with S=buffer+1, T=1R and was NEGATIVE (-521.9R). Wick filter HURTS this strategy.
- [ ] **Slippage reality check:** Tick backtest modeled 0.25pt base + vol adjustment. Real NT execution may differ. Monitor fills vs. expected.
- [ ] **Multi-position Pine v75 validation:** Pine v75 was multi-position but was NOT run through Databento tick validation. If multi-position is desired later, needs separate tick backtest.
