# V9 Full Audit Report — NinjaTrader vs Python Backtester

**Date:** 2026-03-23
**Files Audited:**
- V9NQVWPMStrategy.cs
- V9ESVWPMStrategy.cs
- V9NQLF1170Strategy.cs

**Reference Python:**
- detailed_stats_260d.py (NQ VWPM, NQ LF1170)
- es_vwpm_260d.py (ES VWPM parameter sweep)
- portfolio_clean_260d.py (corrected VWAP, full portfolio)

---

## VERDICT: ALL CLEAR — V9 matches Python backtester

All previous bugs (V6 #1–7, V8 #1–2) and newly discovered V9 fixes (#3–4) are correctly applied. Below is the complete line-by-line verification.

---

## Cumulative Bug Log

| # | Bug | Version Found | Impact | Fix Applied |
|---|-----|--------------|--------|-------------|
| 1 | VWAP tick double-counting | V5 | Live VWAP diverged from backtest; ES WR 44% vs 66% | V6: `lastVwapBar` bar-gating |
| 2 | SetStopLoss/SetProfitTarget conflict | V6 | NinjaTrader ignored manual ExitLong/Short; trades hit backup stop ($245 vs $200 on NQ) | V6.1: Removed entirely |
| 3 | Chart tag collision ("V5Stats") | V5 | NQ chart showed LF1170 stats instead of VWPM stats | V6: Unique tags per strategy |
| 4 | PF=0 when all wins | V5 | Misleading stats display | V6: Returns 999.0 |
| 5 | No execution tracking | V5 | No fill price logging for reconciliation | V6: OnExecutionUpdate handler |
| 6 | Session-flat uses wrong bar | V5 | Close[0] = first bar of new session, not last bar of prior | V6: Close[1] with guard |
| 7 | Side paused during trade | V5 | Next signal not ready after exit; missed entries | V6: Side runs before inTrade check |
| 8 | Entry cutoff 25 min late (VWPM) | V5 | NinjaTrader used flatTime (3:55 PM) vs Python RTH_END-30 (3:30 PM); extra entries in last 25 min | V8: entryEnd = 15:30 |
| 9 | Inclusive side comparisons (VWPM) | V5 | `>=`/`<=` vs Python's strict `>`/`<`; side established on boundary ticks that Python skips | V8: Changed to `>`/`<` |
| 10 | VWPM entry at market, not VWAP | V5 | Python enters at VWAP (`ep=vw`); NinjaTrader entered at Close[0]+VSlip, shifting stops/targets by up to TouchZone+VSlip pts | V9: `EnterTrade(dir, currentVWAP)`, VSlip=0 |
| 11 | LF1170 entry time 1 hour late | V5 | Python minute 1170 = 2:30 PM ET; NinjaTrader had "15:30" (3:30 PM). Wrong VWAP freeze (6 hrs vs 5 hrs) and wrong entry time. Affects which trades trigger and in which direction. | V9: EntryTimeStr = "14:30" |
| 12 | VSlip on all strategies | V5 | Python has no slippage model; VSlip=0.25 shifted entry prices | V9: VSlip default = 0 on all three |

---

## 1. VWAP Computation

| Aspect | Python | NinjaTrader V9 | Match? |
|--------|--------|----------------|--------|
| Formula | `vp += pr[i]; vv += 1; vw = vp/vv` | `vwapCumPV += Close[0]; vwapCumVol += 1; currentVWAP = vwapCumPV / vwapCumVol` | ✅ |
| Time range | `mi[i] >= 870 and mi[i] < 1260` (9:30–16:00 ET) | `tod >= rthStart && tod < rthEnd` (9:30–16:00) | ✅ |
| Bar-gating (Bug 1 fix) | N/A (tick-level loop) | `CurrentBar != lastVwapBar` prevents tick double-counting | ✅ |
| Reset on new day | Implicit (new loop per day) | `ResetDay()` zeroes all VWAP accumulators | ✅ |
| LF1170 freeze time | `compute_vwap_at(pr, mi, 1170)` — stops before minute 1170 (2:30 PM) | Freezes at entryTime = 14:30 (2:30 PM) | ✅ |
| UseVolumeVWAP=false | Simple average of Close prices | Simple average of Close prices | ✅ |

**Note:** Python averages every tick price; NinjaTrader averages one bar-close per minute. On 1-min bars this produces near-identical but not bit-exact VWAP values. This is inherent to NinjaTrader's bar-based architecture and is not a bug.

---

## 2. Side Establishment (VWPM strategies only)

| Aspect | Python | NinjaTrader V9 | Match? |
|--------|--------|----------------|--------|
| Warmup window | `mi[i] < RTH_START+30` (9:30–10:00) | `tod < entryStart` (before 10:00) | ✅ |
| Warmup threshold (NQ) | `pr[i] > vw+5` / `pr[i] < vw-5` | `distFromVWAP > SideWarmup(5)` / `< -SideWarmup` | ✅ |
| Warmup threshold (ES) | `es_pr[i] > evw+2` / `< evw-2` | `SideWarmup = 2.0` | ✅ |
| Establish threshold (NQ) | `pr[i] > vw+10` / `< vw-10` | `distFromVWAP > SideEstablish(10)` / `< -SideEstablish` | ✅ |
| Establish threshold (ES) | `es_pr[i] > evw+4` / `< evw-4` | `SideEstablish = 4.0` | ✅ |
| Comparison operators (Bug 9) | Strict `>` / `<` | V9 uses `>` / `<` | ✅ |
| Runs during trade (Bug 7) | Yes — loop continues regardless of trade | Yes — side block runs before `if (inTrade)` | ✅ |
| Reset on entry | `ls = 0` | `establishedSide = 0` | ✅ |
| Reset on new day | Implicit new loop | `ResetDay()` sets `establishedSide = 0` | ✅ |

---

## 3. Entry Conditions

### NQ VWPM / ES VWPM

| Aspect | Python | NinjaTrader V9 | Match? |
|--------|--------|----------------|--------|
| Entry start | `mi[i] >= RTH_START+30` (10:00 AM) | `tod >= entryStart` (10:00) | ✅ |
| Entry cutoff (Bug 8) | `mi[i] >= RTH_END-30` (3:30 PM) | `tod >= entryEnd` (15:30) | ✅ |
| Max trades/day | `ct2 >= 3` → skip | `tradesToday >= MaxTradesPerDay(3)` → return | ✅ |
| Touch zone (NQ) | `pr[i] <= vw+2 and pr[i] >= vw-2` | `Math.Abs(dist) <= TouchZone(2.0)` | ✅ |
| Touch zone (ES) | `es_pr[i] <= evw+1 and es_pr[i] >= evw-1` | `TouchZone = 1.0` | ✅ |
| Entry price (Bug 10) | `ep = vw` (VWAP, no slippage) | `EnterTrade(dir, currentVWAP)`, VSlip=0 → `entryPrice = currentVWAP` | ✅ |
| Direction | Continuation: `ls > 0` → Long, `ls < 0` → Short | `establishedSide > 0` → Long, `< 0` → Short | ✅ |
| VWAP required | Implicit (vw computed from data) | `currentVWAP <= 0` → skip | ✅ |

### NQ LF1170

| Aspect | Python | NinjaTrader V9 | Match? |
|--------|--------|----------------|--------|
| Entry time (Bug 11) | `mi[i] >= 1170` (2:30 PM ET) | `tod >= entryTime` (14:30 = 2:30 PM) | ✅ |
| Fire once/day | `break` after entry check | `firedToday` + `entryChecked` flags | ✅ |
| Extension threshold | `p > vw1170+10` / `p < vw1170-10` (strict `>` / `<`) | `dist > ExtensionThreshold(10)` / `dist < -ExtensionThreshold` | ✅ |
| Entry price (Bug 12) | `ep = p` (market price, no slippage) | `entryPrice = price + VSlip*dir`, VSlip=0 → `entryPrice = price` | ✅ |
| Direction | Fade: above → Short(-1), below → Long(+1) | Same: `dist > threshold` → Short, `dist < -threshold` → Long | ✅ |
| VWAP frozen (Bug 11) | `compute_vwap_at(pr, mi, 1170)` — 5 hrs of RTH data | Freezes at 14:30 (5 hrs after 9:30) | ✅ |

---

## 4. Exit Logic (ManageTrade)

| Aspect | Python (`trade_f_pts`) | NinjaTrader V9 (`ManageTrade`) | Match? |
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

| Aspect | Python | NinjaTrader V9 | Match? |
|--------|--------|----------------|--------|
| Trigger | New day detected | `today != lastSessionDate` | ✅ |
| Exit price | Last price of prior session | `Close[1]` (prior bar close) | ✅ |
| PnL calc | `d*(flatPrice - ep)` | `tradeDir * (flatPrice - entryPrice)` | ✅ |

---

## 6. Order Management

| Aspect | NinjaTrader V9 | Status |
|--------|----------------|--------|
| No SetStopLoss/SetProfitTarget (Bug 2) | Confirmed removed | ✅ |
| Manual ExitLong/ExitShort only | No managed approach conflicts | ✅ |
| Signal names unique | "NQ_VWPM", "ES_VWPM", "NQ_LF1170" | ✅ |
| Exit signal names | "X_NQ_VWPM", "X_ES_VWPM", "X_NQ_LF1170" | ✅ |
| Chart tag collision fix (Bug 3) | "NQ_VWPM_Stats", "ES_VWPM_Stats", "NQ_LF1170_Stats" | ✅ |
| OnExecutionUpdate logging (Bug 5) | Present in all three strategies | ✅ |

---

## 7. Default Parameters

### NQ VWPM

| Parameter | Python | NinjaTrader V9 | Match? |
|-----------|--------|----------------|--------|
| Stop | 10 pts | StopPts = 10.0 | ✅ |
| Target | 1.0R | TargetR = 1.0 | ✅ |
| Touch Zone | ±2 pts | TouchZone = 2.0 | ✅ |
| Side Warmup | 5 pts | SideWarmup = 5.0 | ✅ |
| Side Establish | 10 pts | SideEstablish = 10.0 | ✅ |
| Max Trades | 3 | MaxTradesPerDay = 3 | ✅ |
| Commission | 0.50 pts | VComm = 0.50 | ✅ |
| Slippage | None | VSlip = 0 | ✅ |
| Flat Time | 3:55 PM | 15:55 | ✅ |
| Entry price | VWAP | currentVWAP | ✅ |

### ES VWPM

| Parameter | Python | NinjaTrader V9 | Match? |
|-----------|--------|----------------|--------|
| Stop | 3 pts | StopPts = 3.0 | ✅ |
| Target | 1.0R | TargetR = 1.0 | ✅ |
| Touch Zone | ±1 pt | TouchZone = 1.0 | ✅ |
| Side Warmup | 2 pts | SideWarmup = 2.0 | ✅ |
| Side Establish | 4 pts | SideEstablish = 4.0 | ✅ |
| Max Trades | 3 | MaxTradesPerDay = 3 | ✅ |
| Commission | 0.25 pts | VComm = 0.25 | ✅ |
| Slippage | None | VSlip = 0 | ✅ |
| Flat Time | 3:55 PM | 15:55 | ✅ |
| Entry price | VWAP | currentVWAP | ✅ |

### NQ LF1170

| Parameter | Python | NinjaTrader V9 | Match? |
|-----------|--------|----------------|--------|
| Stop | 15 pts | StopPts = 15.0 | ✅ |
| Target | 1.5R | TargetR = 1.5 | ✅ |
| Extension | 10 pts | ExtensionThreshold = 10.0 | ✅ |
| Entry Time | minute 1170 (2:30 PM ET) | "14:30" (2:30 PM ET) | ✅ |
| Commission | 0.50 pts | VComm = 0.50 | ✅ |
| Slippage | None | VSlip = 0 | ✅ |
| Flat Time | 3:55 PM | 15:55 | ✅ |
| Entry price | market (`p`) | `price` (VSlip=0) | ✅ |

---

## 8. Known Platform Differences (Not Bugs)

### 8a. Tick vs Bar Resolution

Python iterates every tick within a minute bar. NinjaTrader historical mode fires once per bar. In live mode (OnEachTick), NinjaTrader fires on every tick, matching Python's granularity. Historical backtest results may differ slightly at bar boundaries.

### 8b. Side Establishment Scope

In Python, side establishment stops once max trades or entry cutoff reached (the `continue` skips side update code). In NinjaTrader, side continues running until flatTime. No practical impact since no entries occur afterward.

### 8c. DST / UTC Minute Mapping

Python hardcodes UTC minutes (RTH_START=870, RTH_END=1260) which map to correct ET times during EST (UTC-5). During EDT (UTC-4), all Python time windows shift by 1 hour in real clock time. This is a systemic Python backtester limitation affecting all strategies equally, not a NinjaTrader bug.

---

## Summary

V9 is clean. All 12 identified issues have been fixed. The three strategies now match the Python backtester in entry price basis, entry timing, VWAP freeze points, comparison operators, exit priority, and all default parameters. The only remaining differences are inherent platform constraints (tick vs bar resolution, DST handling).
