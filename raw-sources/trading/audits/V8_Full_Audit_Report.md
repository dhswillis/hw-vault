# V8 Full Audit Report — NinjaTrader vs Python Backtester

**Date:** 2026-03-23
**Files Audited:**
- V8NQVWPMStrategy.cs
- V8ESVWPMStrategy.cs
- V8NQLF1170Strategy.cs

**Reference Python:**
- detailed_stats_260d.py (NQ VWPM, NQ LF1170)
- es_vwpm_260d.py (ES VWPM parameter sweep)
- portfolio_clean_260d.py (corrected VWAP, full portfolio)

---

## VERDICT: ALL CLEAR — No new bugs found

All V6/V8 fixes are correctly implemented. The three V8 strategies faithfully reproduce the Python backtester logic for entry criteria, exit management, VWAP computation, side establishment, and parameter defaults. Below is the line-by-line verification.

---

## 1. VWAP Computation

| Aspect | Python | NinjaTrader V8 | Match? |
|--------|--------|----------------|--------|
| Formula | `vp += pr[i]; vv += 1; vw = vp/vv` | `vwapCumPV += Close[0]; vwapCumVol += 1; currentVWAP = vwapCumPV / vwapCumVol` | ✅ |
| Time range | `mi[i] >= 870 and mi[i] < 1260` (9:30–16:00 ET) | `tod >= rthStart && tod < rthEnd` (9:30–16:00) | ✅ |
| Bar-gating (Bug 1 fix) | N/A (tick-level loop) | `CurrentBar != lastVwapBar` prevents tick double-counting | ✅ |
| Reset on new day | Implicit (new loop per day) | `ResetDay()` zeroes all VWAP accumulators | ✅ |
| LF1170 freeze | `compute_vwap_at(pr, mi, 1170)` — stops accumulating before minute 1170 | `if (tod >= entryTime) { frozenVWAP = currentVWAP; }` — stops before 15:30 | ✅ |
| UseVolumeVWAP=false | Simple average of Close prices | Simple average of Close prices | ✅ |

**Note:** Python averages every tick price; NinjaTrader averages one bar-close per minute. On 1-min bars this produces near-identical but not bit-exact VWAP values. This is inherent to NinjaTrader's bar-based architecture and is not a bug.

---

## 2. Side Establishment (VWPM strategies only)

| Aspect | Python | NinjaTrader V8 | Match? |
|--------|--------|----------------|--------|
| Warmup window | `mi[i] < RTH_START+30` (first 30 min, 9:30–10:00) | `!warmupComplete` when `tod < entryStart` (before 10:00) | ✅ |
| Warmup threshold (NQ) | `pr[i] > vw+5` / `pr[i] < vw-5` | `distFromVWAP > SideWarmup(5)` / `< -SideWarmup` | ✅ |
| Warmup threshold (ES) | `es_pr[i] > evw+2` / `< evw-2` | `SideWarmup = 2.0` | ✅ |
| Establish threshold (NQ) | `pr[i] > vw+10` / `< vw-10` | `distFromVWAP > SideEstablish(10)` / `< -SideEstablish` | ✅ |
| Establish threshold (ES) | `es_pr[i] > evw+4` / `< evw-4` | `SideEstablish = 4.0` | ✅ |
| Comparison operators | Strict `>` / `<` (not `>=`/`<=`) | V8 uses `>` / `<` | ✅ |
| Runs during trade (Bug 7) | Yes — loop continues regardless of trade state | Yes — side block runs before `if (inTrade)` check | ✅ |
| Reset on entry | `ls = 0` | `establishedSide = 0` | ✅ |
| Reset on new day | Implicit new loop | `ResetDay()` sets `establishedSide = 0` | ✅ |

---

## 3. Entry Conditions

### NQ VWPM / ES VWPM

| Aspect | Python | NinjaTrader V8 | Match? |
|--------|--------|----------------|--------|
| Entry start | `mi[i] >= RTH_START+30` (10:00 AM) | `tod >= entryStart` (10:00) | ✅ |
| Entry cutoff | `mi[i] >= RTH_END-30` → `continue` (3:30 PM) | `tod >= entryEnd` (15:30) → `return` | ✅ |
| Max trades/day | `ct2 >= 3` → `continue` | `tradesToday >= MaxTradesPerDay(3)` → `return` | ✅ |
| Touch zone (NQ) | `pr[i] <= vw+2 and pr[i] >= vw-2` | `Math.Abs(dist) <= TouchZone(2.0)` | ✅ |
| Touch zone (ES) | `es_pr[i] <= evw+1 and es_pr[i] >= evw-1` | `TouchZone = 1.0` | ✅ |
| Direction | Continuation: `ls > 0` → Long, `ls < 0` → Short | `establishedSide > 0` → Long, `< 0` → Short | ✅ |
| VWAP required | Implicit (vw computed from data) | `currentVWAP <= 0` → skip | ✅ |

### NQ LF1170

| Aspect | Python | NinjaTrader V8 | Match? |
|--------|--------|----------------|--------|
| Entry time | `mi[i] >= 1170` (3:30 PM) | `tod >= entryTime` (15:30) | ✅ |
| Fire once/day | `break` after entry check | `firedToday` + `entryChecked` flags | ✅ |
| Extension threshold | `p > vw1170+10` / `p < vw1170-10` (strict `>` / `<`) | `dist > ExtensionThreshold(10)` / `dist < -ExtensionThreshold` | ✅ |
| Direction | Fade: extended above → Short, extended below → Long | Same: `dist > threshold` → Short(-1), `dist < -threshold` → Long(+1) | ✅ |
| VWAP frozen | `compute_vwap_at(pr, mi, 1170)` — ticks before 3:30 only | `frozenVWAP = currentVWAP` on first bar at/after 15:30 | ✅ |

---

## 4. Exit Logic (ManageTrade)

| Aspect | Python (`trade_f_pts`) | NinjaTrader V8 (`ManageTrade`) | Match? |
|--------|------------------------|-------------------------------|--------|
| Check order | 1) Flat, 2) Stop, 3) Target | 1) Flat, 2) Stop, 3) Target | ✅ |
| Flat time | `m >= 1255` (3:55 PM) | `tod >= flatTime` (15:55) | ✅ |
| Flat PnL | `pnl = d*(p-ep)` then subtract cm | `fav = tradeDir*(p-entryPrice)` then subtract VComm | ✅ |
| Stop trigger | Long: `p <= stop` / Short: `p >= stop` | `-fav >= StopPts` (equivalent) | ✅ |
| Stop PnL | `-risk` (then subtract cm) | `-StopPts` (then subtract VComm) | ✅ |
| Target trigger | `d*(p-ep)/risk >= tgt_r` | `fav >= targetPts` where `targetPts = StopPts * TargetR` | ✅ |
| Target PnL | `tgt_r * risk` (then subtract cm) | `targetPts` (then subtract VComm) | ✅ |

---

## 5. Session Flat (Bug 6 fix)

| Aspect | Python | NinjaTrader V8 | Match? |
|--------|--------|----------------|--------|
| Trigger | New day detected | `today != lastSessionDate` | ✅ |
| Exit price | Last price of prior session | `Close[1]` (prior bar close) | ✅ |
| PnL calc | `d*(flatPrice - ep)` | `tradeDir * (flatPrice - entryPrice)` | ✅ |

---

## 6. Order Management

| Aspect | NinjaTrader V8 | Status |
|--------|----------------|--------|
| No SetStopLoss/SetProfitTarget | Confirmed removed (Bug 2 fix) | ✅ |
| No managed approach conflicts | Manual ExitLong/ExitShort only | ✅ |
| Signal names unique | "NQ_VWPM", "ES_VWPM", "NQ_LF1170" | ✅ |
| Exit signal names | "X_NQ_VWPM", "X_ES_VWPM", "X_NQ_LF1170" | ✅ |
| Chart tag collision fix | "NQ_VWPM_Stats", "ES_VWPM_Stats", "NQ_LF1170_Stats" | ✅ |
| OnExecutionUpdate logging | Present in all three strategies | ✅ |

---

## 7. Default Parameters

### NQ VWPM

| Parameter | Python | NinjaTrader | Match? |
|-----------|--------|-------------|--------|
| Stop | 10 pts | StopPts = 10.0 | ✅ |
| Target | 1.0R | TargetR = 1.0 | ✅ |
| Touch Zone | ±2 pts | TouchZone = 2.0 | ✅ |
| Side Warmup | 5 pts | SideWarmup = 5.0 | ✅ |
| Side Establish | 10 pts | SideEstablish = 10.0 | ✅ |
| Max Trades | 3 | MaxTradesPerDay = 3 | ✅ |
| Commission | 0.50 pts | VComm = 0.50 | ✅ |
| Flat Time | 1255 (3:55 PM) | 15:55 | ✅ |

### ES VWPM

| Parameter | Python | NinjaTrader | Match? |
|-----------|--------|-------------|--------|
| Stop | 3 pts | StopPts = 3.0 | ✅ |
| Target | 1.0R | TargetR = 1.0 | ✅ |
| Touch Zone | ±1 pt | TouchZone = 1.0 | ✅ |
| Side Warmup | 2 pts | SideWarmup = 2.0 | ✅ |
| Side Establish | 4 pts | SideEstablish = 4.0 | ✅ |
| Max Trades | 3 | MaxTradesPerDay = 3 | ✅ |
| Commission | 0.25 pts | VComm = 0.25 | ✅ |
| Flat Time | 1255 (3:55 PM) | 15:55 | ✅ |

### NQ LF1170

| Parameter | Python | NinjaTrader | Match? |
|-----------|--------|-------------|--------|
| Stop | 15 pts | StopPts = 15.0 | ✅ |
| Target | 1.5R | TargetR = 1.5 | ✅ |
| Extension | 10 pts | ExtensionThreshold = 10.0 | ✅ |
| Entry Time | minute 1170 (3:30 PM) | "15:30" | ✅ |
| Commission | 0.50 pts | VComm = 0.50 | ✅ |
| Flat Time | 1255 (3:55 PM) | 15:55 | ✅ |

---

## 8. Known Differences (Not Bugs)

### 8a. VWPM Entry Price: VWAP vs Market

**Python** enters at VWAP (`ep = vw`), placing stops/targets relative to VWAP.
**NinjaTrader** enters at `Close[0] + VSlip * dir`, placing stops/targets relative to market fill.

Since the touch zone is ±2 (NQ) or ±1 (ES) from VWAP, the max entry price difference is **TouchZone + VSlip** pts (2.25 NQ, 1.25 ES). This shifts stop/target levels by the same amount vs Python. For example on NQ, if market is VWAP+1.5 on a long entry:

- Python stop: VWAP − 10
- NinjaTrader stop: (VWAP+1.5+0.25) − 10 = VWAP − 8.25

The NinjaTrader approach is more realistic for live trading (you can't fill at exactly VWAP), but will produce slightly different backtest results than Python. **LF1170 does NOT have this issue** — both Python and NinjaTrader enter at market price.

### 8b. Tick vs Bar Resolution

Python iterates every tick within a minute bar. NinjaTrader historical mode fires once per bar. This means NinjaTrader can't detect intra-bar stop/target hits in backtesting — a trade that hits stop 30 seconds into a bar in Python won't be detected until bar close in NinjaTrader. In live mode (OnEachTick), NinjaTrader fires on every tick, matching Python's granularity.

### 8c. Side Establishment Scope

In Python, side establishment stops once `ct >= MaxTrades` or time passes entry cutoff (both trigger `continue` which skips the side update). In NinjaTrader, side establishment continues running until flatTime regardless. **No practical impact** — no entries occur after cutoff, so the side value is unused.

---

## 9. Previous Bugs — All Confirmed Fixed

| # | Bug | Fix Verified |
|---|-----|-------------|
| 1 | VWAP tick double-counting | ✅ `lastVwapBar` gating in all 3 strategies |
| 2 | SetStopLoss/SetProfitTarget conflict | ✅ Completely removed from all 3 |
| 3 | Chart tag collision ("V5Stats") | ✅ Unique tags: NQ_VWPM_Stats, ES_VWPM_Stats, NQ_LF1170_Stats |
| 4 | PF=0 when no losses | ✅ Returns 999.0 when grossLoss==0 |
| 5 | No execution tracking | ✅ OnExecutionUpdate in all 3 |
| 6 | Session-flat uses wrong bar | ✅ Close[1] with CurrentBar>0 guard |
| 7 | Side paused during trade | ✅ Side runs before inTrade check |
| V8-1 | Entry cutoff 25 min late | ✅ entryEnd = 15:30 (was flatTime 15:55) |
| V8-2 | Inclusive side comparisons | ✅ Strict > / < (was >= / <=) |

---

## Summary

The V8 codebase is clean. All nine identified bugs have been correctly fixed. The logic flow, parameters, comparison operators, and exit priority exactly match the Python backtester. The only remaining differences (entry price basis, tick-vs-bar resolution, side scope) are inherent platform differences and intentional modeling choices, not bugs.
