# NinjaTrader 8 Sweep Fade Strategy — Build Specification

**Purpose**: This document contains every detail needed to build the NinjaTrader 8 C# strategy.
Read this file, then build `SweepFade.cs`. No other context is needed.

**Source of truth**: `/tmp/phase18_robustness.py` lines 149-514 contain the tick-by-tick backtester.
All parameters and logic below are extracted directly from that code.

---

## ARCHITECTURE OVERVIEW

### NinjaTrader Approach
- **Unmanaged order mode** (`IsUnmanaged = true`) — needed for full control over stops, trails, and order prices
- **Primary data series**: 1-minute bars on NQ (MNQ) — used for sweep detection and trade management
- **Secondary data series**: 60-minute bars — used for 1H swing level generation
- **Calculate mode**: `Calculate.OnEachTick` — critical for sweep detection granularity
- **VWAP**: ~~Use NinjaTrader's `OrderFlowVWAP()` indicator, or~~ Compute manually from bar volume — `cumulative(TP * Vol) / cumulative(Vol)`, reset at RTH open. OrderFlowVWAP requires paid "Order Flow +" add-on. Manual calc used instead. `UseVWAP` defaults to `false`.
- **Prev Day H/L**: Track manually from prior session's 1-minute bar range (more reliable than PriorDayOHLC indicator which may not match RTH session exactly)

### Why Unmanaged Orders
The strategy needs to:
1. Submit market orders at exact sweep-return moments
2. Dynamically move stop loss prices (trail logic)
3. Cancel/replace stops without NinjaTrader's managed order constraints
4. Track multiple state machines (one per level type)

### File Output
Single file: `SweepFade.cs` in `Documents\NinjaTrader 8\bin\Custom\Strategies\`

---

## EXACT PARAMETERS (from backtester)

### Session Times (ALL in ET / exchange time)
```
RTH Open:       9:30 AM ET   (used for prev day H/L range, VWAP reset)
Entry Window:   9:45 AM ET - 12:00 PM ET  (all 4 strategies same window)
EOD Flat:       3:55 PM ET   (close any open position at market)
```
In the Python backtester, these are stored as UTC:
- Entry start: 14:45 UTC = 9:45 AM ET
- Entry end: 17:00 UTC = 12:00 PM ET
- EOD: 20:55 UTC = 3:55 PM ET (assuming EST, not EDT — NinjaTrader handles this via exchange timezone)

**NinjaTrader note**: Use `ToTime(Time[0])` comparisons. NinjaTrader bar times are in exchange timezone. Entry window check: `ToTime(Time[0]) >= 94500 && ToTime(Time[0]) < 120000`. EOD check: `ToTime(Time[0]) >= 155500`.

### Global Constraints
```
Max concurrent positions: 1  (one-trade-at-a-time)
Max risk per trade:       85 points (skip trade if risk > 85)
Commission:               Not modeled in strategy (set in NinjaTrader commission template)
```

### Per-Strategy Parameters

**Strategy 1: Previous Day H/L Sweep** (Priority 1 — highest)
```
Levels:          Yesterday's RTH session High and Low
Wick filter:     5 to 25 points (sweep distance past level)
Risk:            wick + 5 points (stop buffer = 5 pts above/below sweep extreme)
Risk filter:     3 to 60 points (skip if risk outside this range)
Max entries/day: 3 per side (3 for high, 3 for low, independently)
Exit:            Trail 1.0R trigger / 0.5R distance (NO fixed target)
Pre-session:     Track sweeps from midnight (globex open) — if price sweeps
                 past prev day H/L overnight, that counts as a sweep
```

**Strategy 2: 1H Wick Sweep** (Priority 2)
```
Levels:          1H bar swing highs and swing lows from last 5 calendar days
                 Swing high: bar where High[i] > High[i-1] AND High[i] > High[i+1]
                 Swing low:  bar where Low[i] < Low[i-1] AND Low[i] < Low[i+1]
                 Only bars that CLOSED before today's 9:30 AM open qualify
Wick filter:     10 to 80 points
Risk:            wick + 5 points
Max entries/day: 3 per level
Exit:            Trail 1.0R trigger / 0.5R distance (NO fixed target)
```

**Strategy 3: Round 50pt Numbers** (Priority 3)
```
Levels:          Every 50-point increment: ..., 21000, 21050, 21100, ...
                 Generated at session open: from (opening_price / 50) * 50 - 200
                 to that value + 500, in steps of 50
                 Each level has TWO entries: one for sweep-above, one for sweep-below
Wick filter:     5 to 30 points
Risk:            wick + 5 points
Max entries/day: 2 per level per direction (NOT 3 — different from other strategies)
Exit:            Trail 1.0R trigger / 0.5R distance (NO fixed target)
```

**Strategy 4: VWAP Deviation Sweep** (Priority 4 — lowest)
```
Level:           VWAP ± 12 points (deviation = 12)
                 VWAP = cumulative(price × volume) / cumulative(volume) from RTH open
                 Upper level = VWAP + 12, Lower level = VWAP - 12
                 These levels MOVE with VWAP throughout the day
Wick filter:     3 to 25 points (past the deviation level, not past VWAP itself)
Risk:            wick + 5 points
Max entries/day: 3 total (not per side — combined)
Exit:            2.0R FIXED TARGET (NO trail — trail_trigger=0, trail_distance=0)
```

---

## SWEEP DETECTION LOGIC (the core mechanic)

### Concept
For each level, track two states:
1. **Not swept**: Price hasn't gone past the level yet
2. **Swept**: Price has gone past the level (tracking the extreme)

When price is in state "Swept" and then crosses BACK through the level → **ENTRY TRIGGERED**.

### Directionality
Each level has a `sweep_direction`:
- **"above"**: We're watching for price to sweep ABOVE the level. When it does and comes back below → SHORT entry at level price.
- **"below"**: We're watching for price to sweep BELOW the level. When it does and comes back above → LONG entry at level price.

### For 1H wick levels:
- A swing HIGH generates an "above" level at the High price
- A swing LOW generates a "below" level at the Low price

### For round number levels:
- Each round number (e.g., 21050) generates TWO levels:
  - One "above" level at 21050 (watching for sweep above → short fade)
  - One "below" level at 21050 (watching for sweep below → long fade)

### For prev day H/L:
- prev_day_high → "above" direction (sweep above → short)
- prev_day_low → "below" direction (sweep below → long)

### For VWAP:
- VWAP + 12 → "above" direction
- VWAP - 12 → "below" direction

### State Machine Per Level (pseudocode)
```
On each tick/bar:
  if state == NOT_SWEPT:
    if sweep_dir == "above" and price > level_price:
      state = SWEPT
      sweep_extreme = price
    elif sweep_dir == "below" and price < level_price:
      state = SWEPT
      sweep_extreme = price

  elif state == SWEPT:
    // Track how far the sweep goes
    if sweep_dir == "above" and price > sweep_extreme:
      sweep_extreme = price
    elif sweep_dir == "below" and price < sweep_extreme:
      sweep_extreme = price

    // Check for return through level
    if sweep_dir == "above" and price <= level_price:
      // ENTRY CONDITION MET — short fade
      wick = sweep_extreme - level_price
      if wick >= min_wick and wick <= max_wick:
        risk = wick + 5.0
        entry_price = level_price
        stop_price = sweep_extreme + 5.0  // above the sweep tip
        direction = SHORT
        → Submit entry (if passes priority check and max entries check)
      // Reset state regardless of whether we entered
      state = NOT_SWEPT
      sweep_extreme = 0

    elif sweep_dir == "below" and price >= level_price:
      // ENTRY CONDITION MET — long fade
      wick = level_price - sweep_extreme
      if wick >= min_wick and wick <= max_wick:
        risk = wick + 5.0
        entry_price = level_price
        stop_price = sweep_extreme - 5.0  // below the sweep tip
        direction = LONG
        → Submit entry
      state = NOT_SWEPT
      sweep_extreme = initial_value  // 0 for above, 999999 for below
```

### CRITICAL NUANCE: Reset After Return
When price returns through a level (whether we entered or not), the level resets to NOT_SWEPT.
This means the SAME level can sweep and return multiple times per day (up to max_entries limit).

### CRITICAL NUANCE: Prev Day Sweep Resets on Return
For prev_day, when the swept_high flag triggers an entry (or the return happens but wick filter fails), the swept_high flag is reset to False AND extreme_high resets to 0. A new sweep must occur before a new entry can trigger. Same for swept_low.

### CRITICAL NUANCE: Round Number `below` Initial Extreme
For "below" sweep direction, the initial sweep_extreme must be set to a very high value (999999) or to the level_price itself, so that `price < sweep_extreme` properly tracks the furthest low point.

---

## TRAIL STOP LOGIC (for strategies 1, 2, 3)

### Parameters
```
trail_trigger  = 1.0   // Activate trail when MFE reaches 1.0 × risk
trail_distance = 0.5   // Trail 0.5 × risk behind the best price
```

### Logic (on every tick/bar while in position)
```
// Calculate how far price has moved in our favor
if direction == LONG:
  favorable = current_price - entry_price
else:  // SHORT
  favorable = entry_price - current_price

// Update max favorable excursion
if favorable > mfe:
  mfe = favorable

// Check if trail should activate
mfe_in_r = mfe / risk_points
if mfe_in_r >= trail_trigger:
  trail_active = true
  trail_dist_pts = trail_distance * risk_points

  if direction == LONG:
    new_trail_stop = entry_price + (mfe - trail_dist_pts)
    // Trail stop can only move UP (never down) for longs
    if new_trail_stop > current_trail_stop:
      current_trail_stop = new_trail_stop
  else:  // SHORT
    new_trail_stop = entry_price - (mfe - trail_dist_pts)
    // Trail stop can only move DOWN (never up) for shorts
    if current_trail_stop == 0 or new_trail_stop < current_trail_stop:
      current_trail_stop = new_trail_stop
```

### Exit Priority (checked in this order)
```
1. STOP HIT:   Long: price <= stop_price    Short: price >= stop_price
2. TRAIL HIT:  Long: price <= trail_stop    Short: price >= trail_stop  (only if trail_active)
3. TARGET HIT: Long: price >= target_price  Short: price <= target_price (only for VWAP strategy)
4. EOD:        Close at market at 3:55 PM ET
```

### Trail Stop Math Example
```
Entry SHORT at 21050, sweep extreme was 21065, buffer = 5
  risk = (21065 - 21050) + 5 = 20 points
  stop = 21065 + 5 = 21070

Price drops to 21030 (favorable = 20 pts = 1.0R) → trail activates
  trail_dist = 0.5 × 20 = 10 pts
  trail_stop = 21050 - (20 - 10) = 21040

Price drops to 21020 (mfe = 30 pts = 1.5R)
  trail_stop = 21050 - (30 - 10) = 21030  (tightened from 21040)

Price bounces to 21030 → trail stop hit → exit at 21030
  profit = 21050 - 21030 = 20 pts = 1.0R
```

---

## 1H SWING LEVEL GENERATION

### How the Backtester Does It
```python
# Build 1H bars from tick data (resample to 1H OHLC)
# Accumulate across days, keep only last 5 calendar days
# Only use bars that CLOSED before today's 9:30 AM open

# Swing detection (strength=1):
for i in range(1, len(bars) - 1):
    if highs[i] > highs[i-1] and highs[i] > highs[i+1]:
        → swing high level at highs[i], sweep_dir = "above"
    if lows[i] < lows[i-1] and lows[i] < lows[i+1]:
        → swing low level at lows[i], sweep_dir = "below"
```

### NinjaTrader Implementation
Use the `Swing` indicator on the 60-minute secondary series:
```csharp
AddDataSeries(BarsPeriodType.Minute, 60);  // BarsInProgress = 1

// In OnBarUpdate, when BarsInProgress == 1 (60-min bar closed):
// Use Swing(BarsArray[1], 1) to find swing points
// Or manually check: if (Highs[1][1] > Highs[1][0] && Highs[1][1] > Highs[1][2])
//   → that's a swing high 1 bar ago on the 60-min chart
```

**IMPORTANT**: NinjaTrader's `Swing` indicator with strength=1 matches the backtester's logic exactly (compare bar to immediate neighbors). Use `SwingHighBar(0, 1, lookbackBars)` to find the most recent swing high.

**5-day lookback**: Only keep 1H swing levels from the last 5 calendar days. In NinjaTrader, calculate how many 60-minute bars = 5 trading days. Approximately 5 × 7 RTH hours = 35 bars, but include globex: ~5 × 24 = 120 bars max lookback. Filter by time: `Time[barsAgo] >= DateTime.Today.AddDays(-5)`.

**Only CLOSED bars before session**: A 60-min bar that opened at 9:00 AM closes at 10:00 AM. Don't use it for level generation until AFTER it closes. The backtester does this: `recent_1h = all_1h_bars[all_1h_bars.index + pd.Timedelta(hours=1) <= today_open]`. This means: bar's open_time + 1 hour <= 9:30 AM → only bars that closed by 9:30 AM.

In NinjaTrader: regenerate 1H levels once at the start of each session (e.g., at 9:30 AM or on the first bar of the entry window). Use bars from `BarsArray[1]` that have already closed.

---

## PREV DAY H/L TRACKING

### How the Backtester Does It
```python
# After each day's simulation, compute prev_day_high/low from RTH ticks:
open_ns = 14:30 UTC (9:30 AM ET)
eod_ns  = 20:55 UTC (3:55 PM ET)
session_prices = prices[open_idx:eod_idx]
prev_day_high = max(session_prices)
prev_day_low  = min(session_prices)
```

### NinjaTrader Implementation
- Track the high and low of the current RTH session manually
- At session start (9:30 AM), save yesterday's values and reset
- OR use `CurrentDayOHL()` for today and track manually for yesterday

### Pre-Session Sweep Tracking (CRITICAL)
The backtester scans ALL ticks from midnight to 9:30 AM for sweeps of prev day H/L:
```python
for ti in range(0, open_idx):  # tick 0 to RTH open
    if price > prev_day_high:
        swept_high = True
        extreme_high = max(extreme_high, price)
    if price < prev_day_low:
        swept_low = True
        extreme_low = min(extreme_low, price)
```

**NinjaTrader implementation**: The primary 1-minute data series should include globex/overnight bars. On each bar from midnight to 9:30 AM, check if High/Low sweeps past prev day levels. Set the swept flags accordingly. This is essential — without it, prev_day trades drop from +1.86 avgR to +0.04.

---

## VWAP CALCULATION

### How the Backtester Does It
```python
# Reset at RTH open (9:30 AM), accumulate tick-by-tick:
cum_pv += price * volume
cum_v += volume
vwap = cum_pv / cum_v

# Levels:
upper = vwap + 12.0
lower = vwap - 12.0
```

### NinjaTrader Implementation
Use `OrderFlowVWAP()` indicator which resets at session open. Access via `OrderFlowVWAP().VWAP[0]`.
The deviation is a fixed 12 points, NOT a standard deviation band.

```csharp
double vwap = OrderFlowVWAP(VWAPResolution.Standard, Bars.TradingHours,
              VWAPStandardDeviations.Three, 1, 2, 3).VWAP[0];
double upper = vwap + VwapDeviation;  // VwapDeviation = 12.0
double lower = vwap - VwapDeviation;
```

**Note**: The VWAP levels move every tick. Sweep tracking must update the level position on each bar. When checking "has price swept above upper", use the upper level at the time of the sweep, but the entry triggers when price returns to the CURRENT upper level.

Actually, re-reading the backtester more carefully: the sweep is tracked against the CURRENT vwap+dev. The entry also triggers against the CURRENT vwap+dev. So on each tick:
1. Compute current VWAP
2. upper = vwap + 12, lower = vwap - 12
3. If price > upper → swept_above = true, track extreme
4. If swept_above and price <= upper → entry condition (short at upper)

This means the "level" is moving. The entry price is whatever VWAP+12 is at the moment price crosses back.

---

## ROUND NUMBER LEVEL GENERATION

### How the Backtester Does It
```python
first_price = prices[open_idx]  # price at 9:30 AM
low_rn = int(first_price / 50) * 50 - 200
high_rn = low_rn + 500
# Example: if first_price = 21037, low_rn = 20850, high_rn = 21350
# Levels: 20850, 20900, 20950, 21000, 21050, 21100, 21150, 21200, 21250, 21300, 21350
# Each gets TWO entries: one "above", one "below"
# Max 2 entries per level per direction (not 3)
```

### NinjaTrader Implementation
Generate once at session open. These are static for the day (unlike VWAP).

---

## ENTRY PRIORITY SYSTEM

When no position is open and the entry window is active:
```
1. Check Prev Day H/L  → if triggered, enter and skip rest
2. Check 1H Wick       → if triggered, enter and skip rest
3. Check Round Numbers  → if triggered, enter and skip rest
4. Check VWAP           → if triggered, enter
```

If already in a position → do NOT check for new entries. Wait until the position closes.

---

## ORDER FLOW IN NINJATRADER (Unmanaged)

### Entry
When sweep-return detected and priority check passes:
```csharp
// Enter at market (price is already at/past the level on this tick)
if (direction == 1) // LONG
    entryOrder = SubmitOrderUnmanaged(0, OrderAction.Buy, OrderType.Market,
                 Quantity, 0, 0, "", "SweepFadeEntry");
else // SHORT
    entryOrder = SubmitOrderUnmanaged(0, OrderAction.SellShort, OrderType.Market,
                 Quantity, 0, 0, "", "SweepFadeEntry");
```

**Why market order, not limit**: In the backtester, entry happens the instant price crosses back through the level. On 1-minute bars in NinjaTrader, by the time we detect the crossback (bar close), price is already at or past the level. A market order on the next tick is the closest approximation. A limit order at the level price would also work but might not fill if price has already moved through.

### Stop Loss
Immediately after fill:
```csharp
// In OnOrderUpdate or OnExecutionUpdate when entry fills:
if (direction == 1) // LONG
    stopOrder = SubmitOrderUnmanaged(0, OrderAction.Sell, OrderType.StopMarket,
                Quantity, 0, stopPrice, "", "SweepFadeStop");
else
    stopOrder = SubmitOrderUnmanaged(0, OrderAction.BuyToCover, OrderType.StopMarket,
                Quantity, 0, stopPrice, "", "SweepFadeStop");
```

### Trail Stop Update
When trail activates, modify the existing stop order:
```csharp
ChangeOrder(stopOrder, stopOrder.Quantity, 0, newTrailStopPrice);
```

### Target (VWAP only)
```csharp
if (currentStrategy == "vwap_sweep")
    targetOrder = SubmitOrderUnmanaged(0, OrderAction.Sell/BuyToCover, OrderType.Limit,
                  Quantity, targetPrice, 0, "", "SweepFadeTarget");
```

### EOD Exit
```csharp
if (ToTime(Time[0]) >= 155500 && Position.MarketPosition != MarketPosition.Flat)
{
    // Cancel stop and target orders
    if (stopOrder != null) CancelOrder(stopOrder);
    if (targetOrder != null) CancelOrder(targetOrder);
    // Close at market
    if (Position.MarketPosition == MarketPosition.Long)
        SubmitOrderUnmanaged(0, OrderAction.Sell, OrderType.Market, ...);
    else
        SubmitOrderUnmanaged(0, OrderAction.BuyToCover, OrderType.Market, ...);
}
```

---

## NINJATRADER-SPECIFIC CONSIDERATIONS

### OnStateChange
```csharp
case State.SetDefaults:
    Name = "SweepFade";
    Calculate = Calculate.OnEachTick;
    EntriesPerDirection = 1;
    EntryHandling = EntryHandling.AllEntries;
    IsUnmanaged = true;
    // User-configurable parameters as NinjaScript properties

case State.Configure:
    AddDataSeries(BarsPeriodType.Minute, 60);  // 60-min for 1H swings

case State.DataLoaded:
    // Initialize Swing indicator on 60-min series
    // Initialize VWAP indicator
```

### OnBarUpdate BarsInProgress Handling
```
BarsInProgress == 0 (1-min):
  - Main logic: sweep detection, entry checks, trail management, EOD

BarsInProgress == 1 (60-min):
  - Update 1H swing levels when a new 60-min bar closes
  - Only during pre-session (before 9:30) or use stored levels
```

### Session Reset
At the start of each new trading day (detect via `Bars.IsFirstBarOfSession`):
1. Save current session H/L as prev_day_high/low
2. Reset all sweep states
3. Reset entry counts
4. Regenerate round number levels
5. Regenerate 1H swing levels (from last 5 days of closed 60-min bars)
6. Reset VWAP sweep states

### Backtesting Considerations
- Use **Tick Replay** for most accurate results (processes historical data tick-by-tick)
- Without Tick Replay, `OnEachTick` on historical 1-min bars only gives OHLC (4 ticks per bar)
- For the sweep mechanic, OHLC approximation is OK but not perfect
- The backtester used true tick-by-tick — expect ~10-20% fewer trades on NinjaTrader 1-min bars

### Order of Operations on Each Bar
This is the exact order from the backtester, and NinjaTrader must match:
```
1. Update VWAP (accumulate price × volume)
2. Check EOD → close if past 3:55 PM
3. If in position: check exits (stop → trail → target) and update trail
4. Update sweep tracking for ALL level types (even if in position)
5. If NOT in position and within entry window:
   a. Check Priority 1 (Prev Day) for entry
   b. Check Priority 2 (1H Wick) for entry
   c. Check Priority 3 (Round Numbers) for entry
   d. Check Priority 4 (VWAP) for entry
```

**IMPORTANT**: Sweep tracking (step 4) happens BEFORE entry checks (step 5) in the backtester. On the same tick, a level can be swept AND another level can trigger an entry. But a level cannot be swept and entered on the same tick — the sweep must happen first, then the return on a later tick.

Actually, re-reading the code more carefully: in the backtester, the order within a single tick is:
1. Update VWAP
2. Process pending entries (delay mode only)
3. EOD check
4. Exit checks (stop/trail/target) for active positions
5. Sweep tracking updates (all levels)
6. Entry checks (priority order)

So sweep tracking and entry checks happen on the SAME tick. This means: on tick T, price could sweep past a level (step 5), and then immediately in step 6, if price is back past the level (which it can't be — it just swept past), no entry. The entry happens on a LATER tick when price returns.

But there's a subtle case for 1H levels: the level was ALREADY swept on a previous tick. On this tick, price returns through the level. Step 4 (exits) runs first. Step 5 (sweep tracking) might update the extreme. Step 6 (entry) detects the return. This is correct — the return is detected on the same tick price crosses back.

---

## USER-CONFIGURABLE PARAMETERS (NinjaScript Properties)

```csharp
[NinjaScriptProperty] public int Quantity { get; set; }  // default 1
[NinjaScriptProperty] public bool UsePrevDay { get; set; }  // default true
[NinjaScriptProperty] public bool Use1HWick { get; set; }  // default true
[NinjaScriptProperty] public bool UseRoundNumbers { get; set; }  // default true
[NinjaScriptProperty] public bool UseVWAP { get; set; }  // default true
[NinjaScriptProperty] public int RoundNumberStep { get; set; }  // default 50
[NinjaScriptProperty] public double VwapDeviation { get; set; }  // default 12.0
[NinjaScriptProperty] public double TrailTriggerR { get; set; }  // default 1.0
[NinjaScriptProperty] public double TrailDistanceR { get; set; }  // default 0.5
[NinjaScriptProperty] public double VwapTargetR { get; set; }  // default 2.0
[NinjaScriptProperty] public int StopBuffer { get; set; }  // default 5
[NinjaScriptProperty] public double MaxRiskPts { get; set; }  // default 85
[NinjaScriptProperty] public int H1LookbackDays { get; set; }  // default 5
[NinjaScriptProperty] public int EntryStartTime { get; set; }  // default 94500 (HHMMSS)
[NinjaScriptProperty] public int EntryEndTime { get; set; }  // default 120000
[NinjaScriptProperty] public int EODFlatTime { get; set; }  // default 155500
```

---

## POTENTIAL ISSUES TO WATCH FOR

1. **VWAP level is moving**: Unlike other levels which are static for the day, VWAP+12/-12 changes every tick. The sweep extreme must be tracked relative to the VWAP deviation level at the time of the sweep, but the entry triggers when price crosses the CURRENT VWAP level. This could cause the wick size to be different at entry time than at sweep time.

2. **Overnight data for prev day sweeps**: NinjaTrader must load enough historical data to include overnight bars. If using "US Equities RTH" trading hours, overnight bars may not be included. Use "Default 24/7" or "CME US Index Futures ETH" trading hours template to get globex data.

3. **1H bar alignment**: The backtester resamples to clock-hour bars (10:00, 11:00, 12:00...). NinjaTrader 60-minute bars may align differently depending on session template. Use session-anchored bars to match.

4. **Multiple fills on re-entry**: After a stop loss, the same level can trigger again (up to max_entries). The entry count is per-level, not global. Make sure the level tracking preserves entry counts after a loss.

5. **Order state management**: With unmanaged orders, you must handle `OnOrderUpdate` and `OnExecutionUpdate` carefully. Track order objects (entryOrder, stopOrder, targetOrder) and set to null when filled/cancelled.

6. **Partial fills**: Set `IsUnmanaged = true` and handle partial fills. For simplicity, may want to use `Slippage = 0` in backtesting and handle via the strategy's slippage-tested robustness.

7. **Trail stop ratchet direction**: For longs, trail stop only moves UP. For shorts, trail stop only moves DOWN. Never allow it to move against the trade.

8. **Race condition on EOD**: Make sure EOD flat happens even if the stop/trail hasn't been hit. Cancel all working orders before submitting the close.

---

## TESTING CHECKLIST

After building, verify:
1. [ ] Only enters between 9:45 AM and 12:00 PM ET
2. [ ] Flattens all positions at 3:55 PM ET
3. [ ] Never has more than 1 position open
4. [ ] Prev day H/L levels are correct (compare to chart)
5. [ ] 1H swing levels match visual swing points on 60-min chart
6. [ ] Round numbers are correct 50pt increments
7. [ ] VWAP matches NinjaTrader's built-in VWAP line
8. [ ] Stop is placed at sweep extreme + 5pt buffer
9. [ ] Trail activates when price moves 1R in favor
10. [ ] Trail distance is 0.5R behind best price
11. [ ] Trail stop never moves against the trade
12. [ ] VWAP trades use fixed 2R target instead of trail
13. [ ] Priority system works (prev_day blocks 1H, etc.)
14. [ ] Re-entries work (same level can trigger again after stop)
15. [ ] Max entries per level respected (3 for most, 2 for round numbers)
16. [ ] Prev day sweeps tracked from overnight/globex data
17. [ ] Risk filter works (skip trades where risk > 85 pts)
18. [ ] Wick filters correct per strategy type

---

## REFERENCE: EXPECTED BACKTEST RESULTS

On the Python tick-by-tick backtester over 299 days (Feb 2025 - Feb 2026):
```
Total trades:  3,910 (13.1/day)
Win rate:      69.2%
Avg R:         +0.798
R/day:         +10.43
Max drawdown:  8.6R
Calmar ratio:  361.8
Weekly WR:     100%

Strategy breakdown:
  1h_wick:     1,721 trades  +1,420R  +0.825 avgR
  round_num:   1,630 trades  +1,144R  +0.702 avgR
  prev_day:      151 trades    +281R  +1.864 avgR
  vwap_sweep:    408 trades    +273R  +0.669 avgR
```

NinjaTrader on 1-minute bars will likely show fewer trades (~2,500-3,000 instead of 3,910) due to reduced granularity vs tick data. The avgR should be similar or slightly different. The key relationships should hold: 1H wick is the biggest contributor, round numbers second, prev_day highest avgR.
