# TempoPortfoliov1.cs — NinjaTrader Build Spec

**Two independent signal engines in one strategy. Both run simultaneously.**

**Reference file**: `SweepBreakv17.cs` in this same directory — use its unmanaged order pattern, stats display, and lifecycle exactly. Create `TempoPortfoliov1.cs` as a NEW file.

---

## ARCHITECTURE

- **Primary series**: 1-minute bars (BarsInProgress = 0) — used by IFVG engine
- **Secondary series**: 3-minute bars (BarsInProgress = 1) — used by Lumi LTF
- **Tertiary series**: 60-minute bars (BarsInProgress = 2) — used by Lumi HTF
- **IsUnmanaged = true** — all orders submitted via SubmitOrderUnmanaged
- **Calculate = Calculate.OnBarClose** — both engines work on bar close, not tick
- **Two simultaneous positions allowed**: IFVG and Lumi can each have one open trade. Track separately with `_ifvgTrade` and `_lumiTrade`.

---

## ENGINE 1: IFVG (Inverse Fair Value Gap)

### Signal Flow
1. Build key levels once per day before entry window
2. On each 1-min bar close: check for sweep at any key level
3. After sweep: look for Inverse FVG within 20 bars
4. After IFVG: wait for candle to CLOSE through FVG → market order entry
5. Manage with soft stops + session-aware TPs

### Key Levels (rebuild at 3 points during the day)
| Level | Source |
|---|---|
| PDH / PDL | Yesterday's RTH (9:30-16:00 ET) high/low |
| Asia H/L | 6PM-1AM ET (EST) or 5PM-12AM ET (EDT) |
| London H/L | 3AM-8:30AM ET (EST) or 2AM-7:30AM ET (EDT) |
| AM H/L | 9:30-10:30 AM ET high/low |
| 1H Swing H/L | Swing highs/lows on 60-min bars, lookback=3, last 5 days |

De-duplicate: if two levels within 2 points, keep one.

**Level rebuild schedule**: (a) at EntryStartTime (3 AM) — PDH/PDL, Asia, 1H Swings; (b) at LondonEndTime (8:30 AM) — adds London H/L; (c) at 10:30 AM — adds AM H/L. Each rebuild clears and rebuilds all levels from scratch.

**Swept-level memory**: Once a level is swept on a given day, do not allow re-sweeping it (within 2 pts tolerance). Clear the memory on new day. Applies to both IFVG and Lumi engines independently.

### Sweep Detection
- Bar high pierces above a "high" level AND bar closes below it → bearish sweep (SHORT setup)
- Bar low pierces below a "low" level AND bar closes above it → bullish sweep (LONG setup)
- Min sweep: 1.0 points beyond level

### IFVG Detection (within MaxBarsToIFVG bars after sweep)
**For SHORT** (after sweep of HIGH):
- Look for bullish FVG: `Bar[i].Low > Bar[i-2].High` (gap up)
- Gap = `Bar[i].Low - Bar[i-2].High`
- Must be GapMin ≤ gap ≤ GapMax

**For LONG** (after sweep of LOW):
- Look for bearish FVG: `Bar[i-2].Low > Bar[i].High` (gap down)
- Gap = `Bar[i-2].Low - Bar[i].High`

### Entry (within MaxBarsToFill bars after IFVG)
- **LONG**: candle closes ABOVE FVG high → market order BUY
- **SHORT**: candle closes BELOW FVG low → market order SELLSHORT
- Stop = sweep wick ± 1 point
- Risk filter: skip if risk < MinRisk or > MaxRisk

### Soft Stop (CRITICAL)
The stop triggers on **candle CLOSE** past stop level, NOT wick touch. On each bar close:
- LONG: if `Close[0] <= stopPrice` → exit at market
- SHORT: if `Close[0] >= stopPrice` → exit at market
- Keep a disaster hard stop at `stop ± DisasterStopPts` as flash-crash safety only

### Session-Aware TP Management
Determine session from entry time:

**London (EntryStartTime to 83000 ET)**:
- 100% runner, no partials
- B/E: move stop to entry + 0.25pt after +BEThresholdPts favorable
- Ride to flatten time

**NY AM (93000 to NYAMEndTime)**:
- TP1: close 30% at entry ± TP1Pts
- Move stop to B/E (entry price, now HARD stop)
- 70% runner rides to flatten

**NY Lunch + PM (NYAMEndTime to EntryEndTime)**:
- TP1: close 55% at entry ± TP1Pts
- Move stop to B/E (HARD)
- TP2: close 35% at entry ± TP2Pts
- 10% runner rides to flatten

### Optional Mean60 Filter
If `UseMean60Filter = true`:
- Compute SMA(Close, 60) on 1-min bars
- Only take SHORTS when Close > SMA (premium)
- Only take LONGS when Close < SMA (discount)

---

## ENGINE 2: LUMI (HTF Sweep → LTF OB+FVG)

### Signal Flow
1. Build HTF levels from 60-min bars (FVGs, swing points, order blocks)
2. Detect sweep on 60-min bars during entry window
3. **STRICT MSS**: After sweep, confirm market structure shift on 3-min bars BEFORE entry
4. Find LTF order block + FVG on 3-min bars
5. Enter on FVG retracement → hard stop at swing, TP at 3R

### HTF Level Building (from 60-min bars, last 3 days)

**FVGs on 60-min**:
- Bullish: `Bar[i].Low > Bar[i-2].High` and gap ≥ 3.0 pts → level at Bar[i-2].High, type="low"
- Bearish: `Bar[i-2].Low > Bar[i].High` and gap ≥ 3.0 pts → level at Bar[i-2].Low, type="high"

**Swing points on 60-min** (lookback=2):
- Swing high: `High[i]` is max of High[i-2] through High[i+2]
- Swing low: `Low[i]` is min of Low[i-2] through Low[i+2]

**Order blocks on 60-min**:
- Bullish OB: bearish candle followed by bullish engulf (Close > prev High), body ≥ 5pt → level at prev candle Low, type="low"
- Bearish OB: bullish candle followed by bearish engulf (Close < prev Low), body ≥ 5pt → level at prev candle High, type="high"

De-duplicate: levels within 3 points → keep one.

### HTF Sweep Detection (on 60-min bar close)
- High sweeps: `High > level.Price` AND `Close < level.Price` AND `High - level.Price ≥ 1.0` → bearish sweep (SHORT setup)
- Low sweeps: `Low < level.Price` AND `Close > level.Price` AND `level.Price - Low ≥ 1.0` → bullish sweep (LONG setup)

### STRICT MSS (on 3-min bars, MUST happen BEFORE entry fill)
After sweep detected, on subsequent 3-min bars:
- For SHORT setup (swept high): find bar where `Low < recent_swing_low` (swing low = min of 5 bars before sweep on 3-min)
- For LONG setup (swept low): find bar where `High > recent_swing_high` (swing high = max of 5 bars before sweep on 3-min)
- MSS must occur within 20 bars of sweep
- **CRITICAL**: Track the MSS bar index. Entry fill bar must be AFTER MSS bar. If OB+FVG fill happens before MSS, skip the trade.

### LTF Order Block + FVG (on 3-min bars, within 30 bars of sweep)

**Order Block**:
- LONG: bullish candle (Close > Open) with body ≥ 3.0 pts
- SHORT: bearish candle (Close < Open) with body ≥ 3.0 pts

**FVG after OB** (within 30 bars of OB):
- LONG: `Bar[j+1].Low > Bar[j-1].High` (bullish FVG), gap 2-40 pts
- SHORT: `Bar[j-1].Low > Bar[j+1].High` (bearish FVG), gap 2-40 pts

**Entry**:
- LONG: price retraces into FVG (Low ≤ FVG upper edge) → limit entry at FVG upper
- SHORT: price retraces into FVG (High ≥ FVG lower edge) → limit entry at FVG lower

### Lumi Stop & Target
- **Stop**: swing extreme before sweep ± 1.0 pt
  - LONG: min(Low) of 5 bars before sweep start on 3-min, minus 1.0
  - SHORT: max(High) of 5 bars before sweep start on 3-min, plus 1.0
- **Risk filter**: 5 ≤ risk ≤ 50 pts
- **Target**: 3R (entry ± 3.0 × risk)
- **Hard stop** (not soft) — Lumi uses standard stop-market order
- **No partials** — full exit at 3R or stop

### Lumi Session End
Flatten any open Lumi trade at FlattenTime, same as IFVG.

---

## PARAMETERS

| Parameter | Default | Group | Description |
|---|---|---|---|
| **General** | | | |
| IFVGQuantity | 1 | General | Contracts per IFVG trade |
| LumiQuantity | 1 | General | Contracts per Lumi trade |
| **Session** | | | |
| EntryStartTime | 30000 | Session | Entry window start (3:00 AM ET) |
| EntryEndTime | 153000 | Session | Entry window end |
| FlattenTime | 155500 | Session | Flatten all positions |
| **IFVG Setup** | | | |
| GapMin | 5 | IFVG | Min IFVG gap (pts) |
| GapMax | 20 | IFVG | Max IFVG gap (pts) |
| MinRisk | 10 | IFVG | Min risk distance |
| MaxRisk | 45 | IFVG | Max risk distance |
| SweepMinPts | 1.0 | IFVG | Min sweep wick |
| MaxBarsToIFVG | 20 | IFVG | Bars after sweep to find IFVG |
| MaxBarsToFill | 20 | IFVG | Bars after IFVG to get fill |
| DisasterStopPts | 10 | IFVG | Hard stop buffer beyond soft stop |
| UseMean60Filter | false | IFVG | Enable SMA(60) bias filter |
| Mean60Period | 60 | IFVG | SMA lookback |
| EnableIFVG | true | IFVG | Master toggle for IFVG engine |
| **IFVG Management** | | | |
| TP1Pts | 17 | IFVG Mgmt | TP1 distance |
| TP2Pts | 35 | IFVG Mgmt | TP2 distance |
| BEThresholdPts | 17 | IFVG Mgmt | B/E trigger distance |
| SessionAwareTPs | true | IFVG Mgmt | Use session-specific TP configs |
| LondonEndTime | 83000 | IFVG Mgmt | London session end for TP logic |
| NYAMEndTime | 113000 | IFVG Mgmt | NY AM end for TP logic |
| **Lumi Setup** | | | |
| LumiMaxBarsOBFVG | 30 | Lumi | Max bars to find OB+FVG after sweep |
| LumiMinFVG | 2 | Lumi | Min LTF FVG gap |
| LumiMaxFVG | 40 | Lumi | Max LTF FVG gap |
| LumiMinRisk | 5 | Lumi | Min risk |
| LumiMaxRisk | 50 | Lumi | Max risk |
| LumiRTarget | 3.0 | Lumi | R-multiple target |
| EnableLumi | true | Lumi | Master toggle for Lumi engine |
| **Risk** | | | |
| MaxTradesPerDay | 50 | Risk | Combined max trades/day |

---

## STATS TABLE

Same Consolas 11pt monospace box as SweepBreakv17.cs:

```
+--------------------------+
| TempoPortfolio v1        |
+--------------------------+
  COMBINED
  Trades          1234
  Win Rate        55.2%
  Total P/L       +8765 pts
  Avg/Trade       +7.1 pts
  Max DD          432 pts
+--------------------------+
| IFVG ENGINE              |
+--------------------------+
  Trades          890
  Win Rate        68.5%
  Avg/Trade       +9.3 pts
  P/L             +8277 pts
+--------------------------+
| LUMI ENGINE              |
+--------------------------+
  Trades          344
  Win Rate        38.1%
  Avg/Trade       +1.4 pts
  P/L             +488 pts
+--------------------------+
| TODAY MM/DD              |
+--------------------------+
  IFVG            3t +24.5 pts
  Lumi            1t -12.3 pts
  Combined        4t +12.2 pts
+--------------------------+
| OPEN TRADES              |
+--------------------------+
  IFVG: SHORT @ 21450 (SOFT stp)
  Lumi: LONG @ 21380 (3R tgt)
+--------------------------+
```

---

## IMPLEMENTATION NOTES

1. **Two trades can be open at once.** Track `_ifvgTrade` and `_lumiTrade` independently. They have different stop logic (soft vs hard) and different management (partials vs 3R).

2. **OnBarUpdate dispatching**: Check `BarsInProgress` to route to the correct engine:
   - `BarsInProgress == 0` (1-min): run IFVG scan + manage IFVG trade
   - `BarsInProgress == 1` (3-min): run Lumi LTF scan (OB+FVG+MSS) + manage Lumi trade
   - `BarsInProgress == 2` (60-min): run Lumi HTF scan (sweep detection)

3. **Lumi MSS strict ordering**: When a sweep is detected on 60-min, start tracking on 3-min bars. Record `mssBarIndex` when structure breaks. Record `fillBarIndex` when OB+FVG fill happens. Only allow entry if `mssBarIndex < fillBarIndex`. This is the critical audit fix.

4. **IFVG soft stop**: Check on each 1-min bar close. Do NOT place a stop-market order for the initial stop. Only place the disaster stop (hard stop at stop ± DisasterStopPts).

5. **Lumi hard stop**: Place a normal stop-market order immediately after fill. Also place a limit order at the 3R target.

6. **Flatten time**: Both engines flatten at FlattenTime. Cancel all pending orders and exit all positions.

7. **AddDataSeries in State.Configure**:
```csharp
AddDataSeries(BarsPeriodType.Minute, 3);   // BarsInProgress 1
AddDataSeries(BarsPeriodType.Minute, 60);  // BarsInProgress 2
```

8. **IFVG partial positions**: For multi-contract, submit initial entry for full IFVGQuantity. On TP1/TP2, submit partial exit orders. Track remaining quantity.

9. **Level struct**: Use a simple struct/class with `double Price`, `string Type` ("high"/"low"), `string Source`.

10. **Daily reset**: On new day, clear all levels, pending setups, swept-level memory, rebuild levels.

11. **B/E stop move with instant trigger** (v8): When B/E activates and the disaster stop needs to move, try immediately. If the stop order isn't in Working/Accepted state, store as PendingStopMove. In OnOrderUpdate, when the stop transitions to Working, execute the ChangeOrder instantly. Add close-based B/E safety net for when the hard stop move is still pending.

12. **Lumi re-entry protection** (v8): Three layers — (a) swept-level memory (5 pts tolerance), (b) daily trade cap (LumiMaxTradesPerDay, default 3), (c) exit cooldown (2 × 60-min bars after last exit).

13. **Enriched order names** (v10): All orders encode sweep source, session, direction in the name for NinjaTrader Trade Performance analysis. IFVG format: `IFVG_{action}_{source}_{session}_{dir}_{posId}`. Lumi format: `LUMI_{action}_{posId}`.

---

## CHANGELOG

### v13 (current)
- Fixed DrawStats version string (was stuck on "v9")
- Full line-by-line audit — no logic bugs found
- v12: Historical-to-live order transition crash fix (guards + GetRealtimeOrder cleanup)
- v11: Range(0,20) fix for IFVGMaxConsecLoss validation

### v10
- Enriched order/trade names with sweep source, session, direction for Trade Performance filtering
- All v8/v9 fixes included

### v9
- London H/L level rebuild at 8:30 AM
- IFVG swept-level memory (prevents re-sweeping same level same day)
- IFVGMaxConsecLoss: default 2, consecutive logic, win resets to 0
- Session classifier: NYAM >= 93000 (reverted to spec)

### v8
- Lumi re-entry protection (3 layers)
- B/E stop move: Working OR Accepted, PendingStopMove retry, OnOrderUpdate instant trigger
- B/E close-based safety net
- DrawStats: always show BT engine breakdown

---

## BACKTEST EXPECTATIONS

| Engine | Trades/d | WR | Avg R | R/Day | Calmar |
|---|---|---|---|---|---|
| IFVG (baseline, no Mean60) | ~26 | 68.5% | +0.680 | +17.8 | 87.6 |
| Lumi (strict MSS, 3R) | ~14 | 38.1% | +0.398 | +5.67 | 27.5 |
| **Combined** | ~40 | ~55% | - | **+8.18** | **53.9** |
| Correlation | 0.130 | | | | |

The combined Calmar (53.9) exceeds either engine alone because the 0.130 correlation means drawdowns rarely overlap.

---

## FILES TO REFERENCE

- `SweepBreakv17.cs` — unmanaged order pattern, stats display, B/E logic
- `TEMPO_IFVG_BUILD_SPEC.md` — detailed IFVG spec (this file is the portfolio version)
- `~/Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT.md` — full audit results
