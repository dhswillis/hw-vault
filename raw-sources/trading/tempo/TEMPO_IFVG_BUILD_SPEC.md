# Tempo IFVG NinjaTrader Build Spec

**Strategy Name**: TempoIFVGv1
**Instrument**: NQ (MNQ compatible)
**Timeframe**: 1-minute chart
**Style**: Unmanaged orders (like SweepBreakv17.cs)

---

## STRATEGY OVERVIEW

Sweep at key level → Inverse FVG forms → Candle CLOSES through FVG → Market order reversal entry → Partial profit at TP1 + B/E + runner to session end.

This is a **reversal** strategy: price sweeps a key level (liquidity grab), then reverses. The IFVG confirms the reversal.

---

## ENTRY LOGIC (step by step)

### Step 1: Build Key Levels (once per day, before entry window)

Compute these levels from prior price data:

| Level | Source | How to Calculate |
|---|---|---|
| PDH / PDL | Previous day high/low | Yesterday's RTH (9:30-16:00 ET) high and low |
| Asia H/L | Overnight session | 6:00 PM - 1:00 AM ET high/low (EST) or 5:00 PM - 12:00 AM ET (EDT) |
| London H/L | London session | 3:00 AM - 8:30 AM ET (EST) or 2:00 AM - 7:30 AM ET (EDT) |
| AM H/L | First hour of NY | 9:30 - 10:30 AM ET high/low |
| 1H Swing H/L | 1-hour swing points | Swing highs/lows on 60-min bars with lookback=3 (last 5 days of 1H bars) |

**DST Note**: NinjaTrader handles ET automatically if you use the instrument's exchange timezone. Times above are in ET (Eastern Time). No manual DST conversion needed — NinjaTrader's `ToTime()` uses the chart's configured timezone.

**De-duplicate**: If two levels are within 2 points of each other, keep only one.

### Step 2: Detect Sweep (during entry window)

A **sweep** occurs when:
- Current bar's **high** pierces above a key level marked as "high" type, AND current bar **closes below** that level → bearish sweep (sets up SHORT)
- Current bar's **low** pierces below a key level marked as "low" type, AND current bar **closes above** that level → bullish sweep (sets up LONG)
- Minimum sweep size: 1.0 points (wick beyond level)

After a sweep is detected, watch for IFVG on subsequent bars.

### Step 3: Detect Inverse FVG (within 20 bars of sweep)

An **Inverse FVG** (IFVG) is a 3-candle pattern. The FVG forms WITH the sweep momentum, then price breaks back through it (the "inverse fill").

**For SHORT setup** (after sweep of a HIGH):
- Sweep runs price UP past the level
- Look for a BULLISH FVG (gap up): Bar[i].Low > Bar[i-2].High
- Gap size = Bar[i].Low - Bar[i-2].High
- Then wait for candle to close BELOW FVG low → SHORT entry

**For LONG setup** (after sweep of a LOW):
- Sweep runs price DOWN past the level
- Look for a BEARISH FVG (gap down): Bar[i-2].Low > Bar[i].High
- Gap size = Bar[i-2].Low - Bar[i].High
- Then wait for candle to close ABOVE FVG high → LONG entry

Gap size must be between **5-30 points** (configurable).

### Step 4: Entry Signal (candle CLOSE through FVG)

After IFVG is detected, wait (up to 20 bars) for a candle to **close** through it:

**LONG entry**: A candle closes ABOVE the FVG high → market order LONG
**SHORT entry**: A candle closes BELOW the FVG low → market order SHORT

**Stop loss** = sweep wick + 1 point:
- LONG stop = sweep low wick - 1.0
- SHORT stop = sweep high wick + 1.0

**Risk filter**: If |stop - entry| < 10 or > 45 points, skip the trade.

### Step 5: Entry Execution

- Submit **market order** on the close of the confirmation candle
- Set up soft stop monitoring (see management section)
- Place disaster stop (hard stop at stop + DisasterStopPts)

---

## TRADE MANAGEMENT

### Soft Stops (CRITICAL — the entire edge depends on this)

The stop is a **soft stop** — it triggers on **candle CLOSE past the stop level**, NOT on a wick touch. Hard stops kill the strategy (drops to 50% WR).

Implementation: On each bar close (`IsFirstTickOfBar` on the NEXT bar, or `Calculate.OnBarClose`), check if `Close[0]` has crossed the stop level:
- LONG: if `Close[0] <= stopPrice` → exit at market
- SHORT: if `Close[0] >= stopPrice` → exit at market

**Do NOT use a regular stop-market order for the initial stop.** Instead, check on each bar close. However, keep a **disaster stop** (hard stop) at stop + 10 points as a safety net for flash crashes only.

### Partial Profits — Session-Aware

The optimal TP config depends on which session the trade is in:

#### London Session (3:00 AM - 8:30 AM ET entry)
- **No TPs — 100% runner to session end**
- Winners average 4.0R max favorable excursion — don't cap them
- After entry, only exit on soft stop or session flatten
- B/E: move stop to **entry + 0.25 pts** (long) or **entry - 0.25 pts** (short) after trade reaches +17 pts favorable
- The 0.25pt buffer locks in a tiny profit on B/E exits → WR jumps from 20% to 69.5%

#### NY AM Session (9:30 AM - 11:30 AM ET entry)
| Action | Trigger | Size |
|---|---|---|
| TP1 | Price reaches entry ± 17 pts favorable | Close **30%** of position |
| Move to B/E | Same time as TP1 | Move stop to entry price (now HARD) |
| Runner | Session end (flatten time) | Remaining **70%** rides |

#### NY Lunch (11:30 AM - 1:00 PM ET entry)
| Action | Trigger | Size |
|---|---|---|
| TP1 | Price reaches entry ± 17 pts favorable | Close **55%** of position |
| Move to B/E | Same time as TP1 | Move stop to entry price (now HARD) |
| TP2 | Price reaches entry ± 35 pts favorable | Close **35%** of position |
| Runner | Session end (flatten time) | Remaining **10%** rides |

#### NY PM Session (1:00 PM - 3:30 PM ET entry)
Same as NY Lunch — baseline TPs (55/35/10). Not enough time to run before close.

**TP triggers**: TP1 and TP2 trigger on bar high/low reaching the level (not close-based).
**After TP1**: Stop becomes **hard** at breakeven (entry price) — no longer soft.

### Session End Flatten
- Close any open position at flatten time (default 3:55 PM ET / 155500)
- All runner portions ride to this time regardless of entry session

---

## RISK MANAGEMENT

### Max Consecutive Losses Per Day
- Default: **2** — limits cascading losses during choppy sessions
- Configurable: 0 = unlimited, 1-5 = max consecutive losses per day
- A **win resets** the counter to 0 (not decrement — full reset)
- A **B/E trade** (pnl = 0) does NOT count as a loss and does NOT reset the counter
- After hitting max, stop taking new trades for the rest of the day
- Note: backtesting shows 0 (unlimited) produces the most trades/day but 2 provides better risk-adjusted live performance

### Position Sizing
- Use fixed quantity (configurable, default 1 contract)
- For partials with multiple contracts, the quantity input IS the total contracts
- Session determines the split (30/70 for NY AM, 55/35/10 for PM, 100% runner for London)
- If Quantity = 1, no partials — B/E after +17 pts, full exit at session end

---

## CONFIGURABLE PARAMETERS

| Parameter | Default | Group | Description |
|---|---|---|---|
| Quantity | 1 | General | Total contracts per trade |
| EntryStartTime | 30000 | Session | Entry window start (HHMMSS ET) — 3:00 AM for London |
| EntryEndTime | 153000 | Session | Entry window end (HHMMSS ET) — 3:30 PM |
| FlattenTime | 155500 | Session | Flatten all positions |
| GapMin | 5 | Setup | Min IFVG gap size (pts) |
| GapMax | 30 | Setup | Max IFVG gap size (pts) |
| MinRisk | 10 | Setup | Min risk/stop distance (pts) |
| MaxRisk | 45 | Setup | Max risk/stop distance (pts) |
| SweepMinPts | 1.0 | Setup | Min sweep wick beyond level |
| MaxBarsToIFVG | 20 | Setup | Max bars after sweep to find IFVG |
| MaxBarsToFill | 20 | Setup | Max bars after IFVG to get fill |
| TP1Pts | 17 | Management | TP1 distance in points |
| BEThresholdPts | 17 | Management | Move to B/E after this many pts favorable |
| MaxConsecLosses | 2 | Risk | Max consecutive losses/day (0=unlimited) |
| DisasterStopPts | 10 | Risk | Hard stop buffer beyond soft stop |
| Use1HSW | true | Levels | Enable 1H swing levels |
| UseLondon | true | Levels | Enable London H/L |
| UseAsia | true | Levels | Enable Asia H/L |
| UsePDHL | true | Levels | Enable PDH/PDL |
| UseAMHL | true | Levels | Enable AM H/L |
| SessionAwareTPs | true | Management | Use session-specific TP configs |
| LondonStartTime | 30000 | Session | London session start for TP logic |
| LondonEndTime | 83000 | Session | London session end for TP logic |
| NYAMEndTime | 113000 | Session | NY AM end for TP logic |
| NYPMStartTime | 130000 | Session | NY PM start for TP logic |
| UseMean60Filter | false | Filter | Enable Mean60 bias filter (Cal 207 vs 88 without) |
| Mean60Period | 60 | Filter | Lookback period for SMA bias filter |
| GapMax | 20 | Setup | Max IFVG gap size — optimized down from 30 |

---

## SESSION-AWARE TP LOGIC (pseudocode)

```
int entryTimeInt = entryTime.Hour * 10000 + entryTime.Minute * 100;

if (SessionAwareTPs)
{
    if (entryTimeInt >= LondonStartTime && entryTimeInt < LondonEndTime)
    {
        // LONDON: 100% runner, B/E only at threshold
        tp1Pct = 0;  tp2Pct = 0;  // no partials
        // B/E at BEThresholdPts, then ride to flatten
    }
    else if (entryTimeInt >= 93000 && entryTimeInt < NYAMEndTime)
    {
        // NY AM: 30% at TP1, 70% runner
        tp1Pct = 30;  tp2Pct = 0;
    }
    else
    {
        // NY LUNCH + PM: 55% at TP1, 35% at TP2, 10% runner
        tp1Pct = 55;  tp2Pct = 35;
    }
}
```

---

## STATS TABLE (on-chart display)

Same pattern as SweepBreakv17.cs — Consolas 11pt, top-left, monospace box sections:

```
+--------------------------+
| TempoIFVG v1             |
+--------------------------+
  Trades          3665
  Win Rate        72.6%
  Profit Factor   3.41
  Total P/L       +30792 pts
  Avg/Trade       +8.4 pts
  Pts/Day         +99.2
  Max Drawdown    572 pts
  Day WR          78%
+--------------------------+
| SESSION BREAKDOWN        |
+--------------------------+
  London          600t  +14.2/d
  NY AM           1374t +44.1/d
  NY Lunch        441t  +7.1/d
  NY PM           674t  +16.6/d
+--------------------------+
| DIRECTION                |
+--------------------------+
  Long            XXXt  XX% WR
  Short           XXXt  XX% WR
+--------------------------+
| TODAY 03/09              |
+--------------------------+
  Trades          8
  Consec Losses   0
  P/L             +86.3 pts
+--------------------------+
| OPEN TRADE               |
+--------------------------+
  Direction       SHORT
  Session         NY_PM
  Entry           21450.25
  Stop            21476.00 (SOFT)
  TP Config       55/35/10
  Risk            25.8 pts
  Open P/L        +18.5 pts
  Partials        1/2 (TP1 hit)
```

---

## BACKTEST RESULTS (what to expect)

### Production Config: LON + NY Full, 1min, Session-Aware TPs, Unlimited
- **3665 trades** over 312 days (**11.7/day**)
- **72.6% WR**, +8.4-9.1 avg pts, **+99-107 pts/day**
- **Calmar 51-54**, 78% daily WR
- Walk-forward: +6.8 → +10.2 (OOS improves)
- Bootstrap 95% CI: [+7.8, +9.1] — well above zero

### Session Breakdown
| Session | Trades | WR | Avg Pts | Pts/Day | Best TP Config |
|---|---|---|---|---|---|
| London 3-8:30AM | 600 | 69.5% | +7.4-16.0 | +14-31 | 100% runner |
| NY AM 9:30-11:30 | 1374 | 76.0% | +10.0-12.1 | +44-54 | 30/70 split |
| NY Lunch 11:30-1 | 441 | 66.7% | +5.0 | +7.1 | Baseline 55/35/10 |
| NY PM 1-3:30 | 674 | 70.8% | +7.7 | +16.6 | Baseline 55/35/10 |

### Validated Against Tick Data
Bar sim with soft stops matches tick sim within 0.7 points. Soft stops are critical — hard stops kill the strategy (50% WR, -0.5 avg pts).

---

## REFERENCE FILES

- **Python backtests**: `/tmp/ifvg_optimize.py`, `/tmp/ifvg_max_losses.py`, `/tmp/ifvg_sessions_nocap.py`
- **Tick vs bar validation**: `/tmp/ifvg_tick_compare.py`
- **SMT audit**: `/tmp/ifvg_smt_audit.py`
- **Full audit report**: `~/Documents/trading-system/results/IFVG_AUDIT_RESULTS.md`
- **SweepBreak pattern to follow**: `SweepBreakv17.cs` (same directory)
- **Tempo rules source**: `~/Downloads/tempo_extracted/tempo_rules_v3_complete.md`

---

## CRITICAL IMPLEMENTATION NOTES

1. **Soft stops are non-negotiable** — the entire edge depends on NOT stopping out on wicks. Check close only. Keep a disaster hard stop 10pts beyond as flash-crash safety net only.

2. **Market order entry at candle close** — no limit orders, no stop entries. When the confirmation candle closes through the FVG, enter at market on the next tick.

3. **1H swing detection**: Use 60-min bars, lookback=3 (a swing high must be the highest high in 3 bars on each side). Use last 5 days of 1H bars to build swing list.

4. **Session times are ET** — NinjaTrader handles DST automatically when using exchange timezone. Entry window 3:00 AM - 3:30 PM ET covers London through NY PM.

5. **Partial position management**: Session-aware. London = 100% runner. NY AM = 30% TP1 / 70% runner. Lunch+PM = 55% TP1 / 35% TP2 / 10% runner. If Quantity = 1, no partials — just B/E + ride to session end.

6. **Max consecutive losses**: Default 0 (unlimited). Unlimited outperforms every cap in backtesting. Configurable for risk comfort.

7. **De-duplicate levels**: Two levels within 2 points = keep one. Prevents double-counting.

8. **FVG direction**: The FVG forms WITH the sweep momentum (not against it). After a sweep HIGH, look for a BULLISH FVG (gap up on the way up). Then price reverses and closes BELOW the FVG → SHORT entry. The "inverse" is that price fills back through the FVG in the reversal direction.

9. **London runners are the biggest edge**: London winners average 4.0R max favorable excursion. Do NOT take partial profits in London — let the full position ride to session end or B/E stop.

10. **No trade cap**: Unlimited trades per day produces the best results. The strategy generates 11.7 trades/day across all sessions. Do not artificially limit.

11. **Mean60 bias filter (optional, default OFF)**: If `UseMean60Filter=true`, only take shorts when price is above SMA(Close,60) (premium) and longs when price is below SMA(Close,60) (discount). This triples Calmar (207 vs 88) by filtering to 75.7% WR, +13.6 avg, but reduces trade count to ~12/day. Toggle for risk preference.

12. **FVG gap cap**: Optimized from 30pt to 20pt. Gaps 20-30pt have Cal 2.9 — no edge. Use `GapMax=20`.

13. **Level rebuild timing** (v9 fix): Levels must rebuild at 3 points during the day: (a) at EntryStartTime (3:00 AM) — gets PDH/PDL, Asia H/L, 1H Swings; (b) at LondonEndTime (8:30 AM) — adds London H/L for NY AM trades; (c) at 10:30 AM — adds AM H/L. Without the 8:30 AM rebuild, London H/L are missing for the critical NY AM open window when London extremes are most commonly swept.

14. **Swept-level memory** (v9 fix): Track swept levels per day. Once a level is swept, do not re-sweep it on the same day (within 2 pts tolerance). Clear the list on new day. Without this, the same level (e.g. AsiaL or 1HSL) can be swept 3-5+ times in a day, producing duplicate losing setups.

15. **B/E stop move with instant trigger** (v8 fix): When B/E activates, the disaster stop must be moved to the B/E price. If the stop order is not in Working/Accepted state, store the pending move price. In `OnOrderUpdate`, when the stop transitions to Working and there's a pending move, execute the `ChangeOrder` immediately — don't wait for the next bar. This mirrors how a live trader would act the instant they see the order on screen. Also add a close-based B/E safety net: if B/E is active but the hard stop hasn't been moved yet, check Close[0] against the B/E price and exit if hit.

16. **Enriched order names** (v10): All order names encode sweep source, session, and direction for NinjaTrader Trade Performance analysis. Format: `IFVG_{action}_{source}_{session}_{dir}_{posId}` (e.g. `IFVG_Entry_PDH_NYAM_S_42`). Exit/partial names follow the same pattern with the exit reason.

---

## CHANGELOG

### v13 (current)
- Fixed DrawStats version string (was stuck on "v9")
- Full line-by-line audit of all 1867 lines — no logic bugs found
- v12: Historical-to-live order transition crash fix (OnOrderUpdate/OnExecutionUpdate guards, GetRealtimeOrder cleanup)
- v11: Range(0,20) fix for IFVGMaxConsecLoss property validation

### v10
- Enriched order/trade names: all IFVG orders now include sweep source (PDH/PDL/AsiaH/1HSH/etc), session (LON/NYAM/LUNCH/NYPM), and direction (L/S) in the NinjaTrader order name for Trade Performance filtering
- All v9 fixes included

### v9
- London H/L level rebuild at 8:30 AM — fixes missing London levels for NY AM open trades
- IFVG swept-level memory — prevents re-sweeping same price level on the same day
- MaxConsecLoss renamed, default changed to 2, win resets to 0 (not decrement)
- Session classifier reverted to spec: NYAM starts at >= 93000 (not LondonEndTime)

### v8
- Lumi re-entry protection: swept-level memory, daily cap (3), exit cooldown (2 × 60-min bars)
- B/E stop move: accepts Working OR Accepted state, PendingStopMove retry, OnOrderUpdate instant trigger
- B/E safety net: close-based exit when hard stop move is still pending
- DrawStats: always shows BT engine breakdown + LIVE sections after Realtime transition
