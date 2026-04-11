# Tempo Strategy Audit Results — March 29, 2026

## 3 Bugs Found and Fixed

### Bug 1: CRITICAL — Lumi 1H-to-3M Look-Ahead
**File**: `lumi_v19_backtest.py` line 113
**Bug**: `h1_to_3m = np.searchsorted(bars_3m.index.values, bars_1h_full.index.values, side='left')` maps each 1H bar to the 3M bar at the same START time. For a 1H bar starting at 10:00, this points to the 3M bar at 10:00 — but the 1H bar doesn't close until 11:00. The MSS/OB/FVG scan starting at 10:03 uses the 1H bar's close (which is 57 minutes in the future).

**Fix**: Offset by 1H duration: `h1_end_times = bars_1h_full.index.values + np.timedelta64(60, 'm')` then searchsorted on end times. Now the LTF scan starts AFTER the 1H bar closes.

**Impact**: Lumi WR dropped from inflated levels to 41%. R/Day dropped to +0.130.

### Bug 2: TP Touch-Fill Instead of Tick-Through
**Files**:
- `lumi_v19_backtest.py` lines 293-295 (soft-stop mode)
- `market_vs_limit_compare.py` lines 374-375 (`sim_trade_limit`)

**Bug**: TP check used `>=`/`<=` (touch fills) instead of strict `>`/`<` (tick-through). Since TP is a limit order, it requires price to trade PAST the level.

**Fix**: Changed to strict `>` for long TP, `<` for short TP in both files.

### Bug 3: Missing Flatten Fallback in Lumi Soft-Stop Mode
**File**: `lumi_v19_backtest.py` lines 287-301
**Bug**: If the 3M bar loop exhausts without hitting target, stop, or flatten time, PnL defaults to 0.0 instead of mark-to-market.

**Fix**: Added `else` clause with mark-to-market at last bar close.

## Post-Fix Results

### Individual Components (244 trading days)
| Component | RT | Fills | T/day | WR | R/trade | R/Day | MDD(R) | Calmar |
|-----------|---:|------:|------:|---:|--------:|------:|-------:|-------:|
| 15S_short | 3.0 | 323 | 1.3 | 66.3% | +1.580 | +2.067 | 16.0 | 0.129 |
| 15S_long | 3.0 | 196 | 0.8 | 56.1% | +1.162 | +0.922 | 23.3 | 0.040 |
| 30S_short | 2.0 | 317 | 1.3 | 60.9% | +0.777 | +0.997 | 17.6 | 0.057 |
| 30S_long | 2.0 | 395 | 1.6 | 61.0% | +0.773 | +1.235 | 10.7 | 0.115 |
| 1M_short | 3.0 | 198 | 0.8 | 61.1% | +1.407 | +1.128 | 7.4 | 0.152 |
| 1M_long | 3.0 | 278 | 1.1 | 52.5% | +1.065 | +1.198 | 13.7 | 0.088 |
| **lumi** | **3.0** | **139** | **0.6** | **41.0%** | **+0.231** | **+0.130** | **12.4** | **0.011** |

### Best Portfolio (post-fix)
**15SS_s1 + 30SS_s1 + 1MS_s1 + 1ML_s1**: +4.20 R/Day, 62% WR, MDD 11.8R, Calmar 0.356

No combo achieves MDD < 10R after the Lumi look-ahead fix. The 10R target is no longer achievable with this signal set.

### IFVG Engine Status
The IFVG engine (composite_strategy.py) passed all audit checks. No changes needed. Its results are unchanged.

### Cache
`component_trades.pkl` was deleted and regenerated with all 3 fixes applied.

---

# TempoPortfolio NinjaTrader Strategy Audit — March 30-31, 2026

## Summary

Built TempoPortfoliov20.cs (NinjaTrader 8, unmanaged order mode, 6 independent IFVG components). Audited across 5 passes over 4 versions (v20→v23). All bugs fixed. Final v23 compiles and runs.

## Version History

- **v20**: Initial build. 12+ bugs found in 5-pass audit.
- **v21**: Fixed v20 bugs (stray #endregion, position overwrite, orphaned positions, repeated flatten, ResetWeek guards). Hit 3 compile errors.
- **v22**: Fixed compile errors (Display attribute, OnOrderUpdate signature, [Parameter] attribute). Full re-audit found 3 more issues.
- **v23**: Final. Fixed `ExitOnSessionCloseStrategy` (not available in user's NT8 version → `IsExitOnSessionCloseStrategy`). Compiles and runs.

## All Bugs Found and Fixed (v20→v23)

### Compile Errors Fixed

| # | Error | Code | Fix |
|---|-------|------|-----|
| 1 | `using NinjaTrader.Instrument` — namespace doesn't exist | CS0234 | Changed to `NinjaTrader.Gui.Tools` |
| 2 | `OnOrderUpdate` — wrong 13-param signature | CS0115 | Corrected to NT8's 10-param signature |
| 3 | `[Parameter]` — not an attribute class in NT8 | CS0616 | Changed to `[NinjaScriptProperty]` |
| 4 | `Display` / `DisplayAttribute` — missing using | CS0246 | Added `System.ComponentModel.DataAnnotations` |
| 5 | `trade.BIP` — ComponentTrade had no BIP field | CS0103 | Added `BIP` property, populated in SubmitEntry |
| 6 | `execution.ExecutionType` — doesn't exist on NT8 Execution | CS0103 | Removed check (OnExecutionUpdate only fires on fills) |
| 7 | `FlatToday` — not a valid NT8 Strategy property | CS0103 | Removed |
| 8 | `ExitOnSessionCloseStrategy` enum — not in user's NT8 version | CS0103 | Changed to `IsExitOnSessionCloseStrategy = true` |

### Logic Bugs Fixed

| # | Bug | Severity | Fix |
|---|-----|----------|-----|
| 1 | BIP 0 never called `ProcessLTFComponent(0)` — 1M components dead | CRITICAL | Added `ProcessLTFComponent(0)` to BIP 0 block |
| 2 | Exit order directions INVERTED — short trades Sell to close | CRITICAL | Swapped to `(short) ? Buy : Sell` |
| 3 | Stop PnL used raw points instead of R | HIGH | Compute from fill price: `pnlPts / trade.Risk` |
| 4 | Long FVG edge set to barLow (wrong — should be bar2High) | HIGH | Fixed to `Highs[bip][2]` (bottom of FVG) |
| 5 | SubmitOrderUnmanaged was placeholder returning null | HIGH | Replaced with real NT8 API call |
| 6 | Orphaned positions on error paths (entry filled, stop/target fails) | HIGH | Added market exit orders on all 4 error paths |
| 7 | Stray `#endregion` — compile error | MEDIUM | Removed 2 orphaned endregions |
| 8 | Position check allowed overwrite of pending entry | MEDIUM | Changed from `ContainsKey && IsActive` to just `ContainsKey` |
| 9 | FlattenAllPositions called every bar after flatten time | MEDIUM | Added `flattenedToday` flag |
| 10 | BarsSinceSweep double-counted (two increment locations) | MEDIUM | Removed UpdatePendingSetups |
| 11 | DrawStats called on every BIP (5x per 1M bar) | LOW | Moved into BIP 0 block only |
| 12 | `components.First()` could throw | LOW | Changed to `FirstOrDefault()` + null checks |
| 13 | Division by zero when `trade.Risk == 0` | LOW | Added `(trade.Risk > 0)` guard |
| 14 | SubmitOrderHelper null not checked after stop/target submit | LOW | Added null checks with cleanup |
| 15 | Weekly reset used Sunday start (not ISO Monday) | LOW | Fixed to ISO Monday via `GetWeekStart()` |
| 16 | ResetWeek missing ContainsKey guard | LOW | Added guard |
| 17 | Bare `High[n]`/`Low[n]` instead of multi-series `Highs[bip][n]` | LOW | Converted all to multi-series syntax |
| 18 | `Time[0]` in SetDefaults (no bars exist) | LOW | Moved to State.DataLoaded |
| 19 | Swing detection negative barsAgo | LOW | Rewrote with confirmed-bar approach |
| 20 | Race condition: OnExecutionUpdate before trade dict populated | LOW | Populate trade BEFORE SubmitOrderHelper |
| 21 | DetectSweeps15M guard too weak (`< 1` → `< 5`) | LOW | Increased minimum bar count |
| 22 | HandleEntryFill risk < 0.25 guard missing | LOW | Added |
| 23 | RecordTrade missing ContainsKey check | LOW | Added |

## Python ↔ C# Cross-Reference (Verified)

All 6 component configs match `OPTIMIZED_RULES` exactly: RT, RiskCap, GapCap, Buffer, MinRisk, DailyRLimit, StreakLimit, WeeklyRLimit, AllowedSessions.

FVG detection, hard stop, target, session classification boundaries, entry/flatten windows, risk filters, streak logic, weekly R reset — all verified to match Python composite_strategy.py.

### Expected Minor Differences (Not Bugs)

1. **Target fills**: Python uses strict `>` / `<` (tick-through). NT8 limit orders fill at exact price. NT8 may show slightly more target hits.
2. **Entry price reference**: Python uses next bar open. C# uses current bar close for initial risk check, but recalculates from actual fill in HandleEntryFill.
3. **Slippage**: Python applies 0.5pt to market entries. NT8 unmanaged mode fills at actual market price. Set slippage to 2 ticks in NT8 strategy settings.

## File Reference

- **TempoPortfoliov23.cs** — Final version, compiles and runs in NT8
- **composite_strategy.py** — Python IFVG engine (reference for logic verification)
- **optimized_composite.py** — OPTIMIZED_RULES configs (reference for parameter verification)
