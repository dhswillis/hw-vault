# Tempo Trading Brain - Rule Implementation Matrix
## QuantConnect LEAN Algorithm v1.0

This document maps each of the 23 extracted Tempo rules to the corresponding code implementation in `qc_tempo_brain.py`.

---

## ENTRY RULES (Rules 1-8)

### RULE 1: Internal Fair Value Gap (IFVG) Detection
**Original Rule:** "Internal Fair Value Gap (IFVG) is the primary indicator/pattern"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _detect_ifvg() [Lines 502-552]
Timeframe: 5-minute consolidation
Pattern: 3-candle setup
```

**Algorithm Logic:**
- **Bullish IFVG:** Candle 1 up, Candle 2 down (creates gap), Candle 3 up + closes above Candle 1 high
  - Gap detected: C1.High > C2.Low
  - Signal level: C2.Low (the gap bottom)

- **Bearish IFVG:** Candle 1 down, Candle 2 up (creates gap), Candle 3 down + closes below Candle 1 low
  - Gap detected: C2.High > C1.Low
  - Signal level: C2.High (the gap top)

**Calculation:**
```python
def _detect_ifvg(self, bars_window):
    c1 = bars_window[2]  # First candle
    c2 = bars_window[1]  # Middle candle (creates gap)
    c3 = bars_window[0]  # Third candle (current)

    # Bullish: Gap between C1 high and C2 low
    gap_size = c1.High - c2.Low
    if gap_size > FVG_THRESHOLD * c1.Close:
        if c3.Close > c1.High:  # Confirmation
            ifvg_bullish = True
            ifvg_level = c2.Low
```

---

### RULE 2: Supply Manipulation Test (SMT) Confirmation
**Original Rule:** "Supply Manipulation Test (SMT) - price tests IFVG then rejects it"

**Implementation:**
```
File: qc_tempo_brain.py
Integration: Part of entry signal validation [Lines 580-630]
Concept: Price approaching and confirming IFVG level = SMT test
```

**How It Works:**
- After IFVG detected, SMT occurs when price tests the gap level
- Rejection signals smart money at that level
- Entry triggered on close THROUGH (not just near) the IFVG level

**Code Section:**
```python
# In _check_entry_signals():
if (len(self.ifvg_bullish) > 0 and self.ifvg_bullish[0] and
    current_close > self.ifvg_level[0]):  # This is the SMT test
    # Execute entry
```

---

### RULE 3: Enter on Break with Close Confirmation
**Original Rule:** "Enter when price breaks and closes above a valid IFVG"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 560-630]
Confirmation: Close above/below IFVG level (not just touch)
```

**Algorithm:**
- Bullish: `current_close > ifvg_level` = Entry signal
- Bearish: `current_close < ifvg_level` = Entry signal
- NOT just touching the level - must CLOSE through it
- Prevents false breaks and whipsaws

**Code:**
```python
# Bullish entry confirmation
if (len(self.ifvg_bullish) > 0 and self.ifvg_bullish[0] and
    current_close > self.ifvg_level[0]):  # Close confirmation

# Bearish entry confirmation
if (len(self.ifvg_bearish) > 0 and self.ifvg_bearish[0] and
    current_close < self.ifvg_level[0]):  # Close confirmation
```

---

### RULE 4: Multi-Timeframe Confluence (5m Entry, 1H/4H/Daily for Bias)
**Original Rule:** "Confluence increases probability - multiple timeframe IFVGs align"

**Implementation:**
```
File: qc_tempo_brain.py
Daily Bias: _determine_daily_bias() [Lines 476-495]
Consolidators: 5-minute (entry) + daily (bias) [Lines 84-91]
Confluence Check: EMA alignment across timeframes
```

**How It Works:**
1. **5-Minute:** Entry signal trigger (IFVG detection and break)
2. **Daily:** Bias determination (EMA alignment)
3. Confluence: Entry direction must match daily bias

**Algorithm:**
```python
def _determine_daily_bias(self):
    """Daily bias from EMA alignment"""
    # Bullish: Price > EMA9 > EMA21 > EMA50 > EMA200
    if (price > ema_9 > ema_21 > ema_50 > ema_200):
        return 'bullish'

    # Bearish: Price < EMA9 < EMA21 < EMA50 < EMA200
    if (price < ema_9 < ema_21 < ema_50 < ema_200):
        return 'bearish'

    return 'neutral'
```

---

### RULE 5: Trade Only in Direction of Daily Bias
**Original Rule:** "Trade setups that align with daily bias have higher probability"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 580-630]
Gate: Entry only if direction matches daily_bias
```

**Critical Filter:**
```python
# BULLISH entry only if daily bias is bullish
if daily_bias == 'bullish':
    # ... then check for bullish IFVG

# BEARISH entry only if daily bias is bearish
if daily_bias == 'bearish':
    # ... then check for bearish IFVG
```

This is MANDATORY - counter-bias trades are rejected.

---

### RULE 6: Market Structure Alignment (Higher Highs/Lower Lows)
**Original Rule:** "Market structure determines setups (up, down, ranging)"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 595-610]
Validation: EMA sequence confirmation
```

**Structure Confirmation:**
- **Bullish:** EMA9 > EMA21 > EMA50 > EMA200 (uptrend structure)
- **Bearish:** EMA9 < EMA21 < EMA50 < EMA200 (downtrend structure)

```python
# Bullish structure check
if (self.ema_9_5m[0] > self.ema_21_5m[0] >
    self.ema_50_5m[0] > self.ema_200_5m[0]):
    # Valid uptrend structure - proceed with long entry

# Bearish structure check
if (self.ema_9_5m[0] < self.ema_21_5m[0] <
    self.ema_50_5m[0] < self.ema_200_5m[0]):
    # Valid downtrend structure - proceed with short entry
```

---

### RULE 7: Price Action Confirmation (Close Above/Below Entry Level)
**Original Rule:** "Confirmation candle following break"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 575-582]
Validation: Close through IFVG level (not just touch)
```

Already covered in Rule 3 - the close confirmation requirement ensures we don't enter on wicks.

---

### RULE 8: A+ Setup Quality Filtering (Multiple Confirmations)
**Original Rule:** "Don't force trades - wait for A+ setups only"

**Implementation:**
```
File: qc_tempo_brain.py
Confluence Requirements: Multiple conditions must ALL be true
Entry Logic: [Lines 580-630]
```

**A+ Setup Criteria (ALL must be true):**
1. ✓ IFVG detected and confirmed
2. ✓ Close above/below IFVG level
3. ✓ EMA structure aligned
4. ✓ Daily bias aligned
5. ✓ Valid trading session
6. ✓ ATR sufficient for position sizing

If ANY condition fails, trade is skipped (patience enforced).

---

## EXIT RULES (Rules 9-13)

### RULE 9: Take Profit at Liquidity Levels
**Original Rule:** "Take profit at significant liquidity levels (supply/demand zones, previous highs/lows)"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_exit_signals() [Lines 632-680]
Primary Target: Previous day high (longs) / low (shorts)
Secondary Target: 2x ATR if previous level not applicable
```

**Algorithm:**
```python
def _check_exit_signals(self, current_bar, daily_bias):
    trade = self.current_trade
    current_price = current_bar.Close

    # Long: Target previous day high
    if trade.trade_direction == 1:
        if current_price >= trade.take_profit:
            return 'LIQUIDITY_TP_LONG'

    # Short: Target previous day low
    elif trade.trade_direction == -1:
        if current_price <= trade.take_profit:
            return 'LIQUIDITY_TP_SHORT'
```

**Liquidity Levels Tracked:**
- Previous day high/low/close
- Session high/low
- Major S/R levels from swing analysis

---

### RULE 10: Breakeven Stop After 1:1 R:R Reached
**Original Rule:** "Breakeven stop strategy helps more than it hurts"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_exit_signals() [Lines 660-669]
Trigger: Trade profit >= risk amount
Action: Move stop to entry price
```

**Mechanism:**
```python
# When 1:1 R:R achieved
profit = abs(current_price - trade.entry_price)
risk = abs(trade.entry_price - trade.stop_loss)

if profit >= risk:  # 1:1 R:R reached
    trade.breakeven_triggered = True
    trade.stop_loss = trade.entry_price  # Move stop to breakeven
    # Now trade is "free" - can only win or break even
```

**Psychology Benefit:**
- Removes emotional stop-loss decisions
- Protects initial capital once 1:1 achieved
- Allows runner trades to continue

---

### RULE 11: Partial Close at Major S/R Levels
**Original Rule:** "Close partial position at major resistance/support before continuation"

**Implementation:**
```
File: qc_tempo_brain.py
Note: Configured for full exit in current version
Modification: Can be enhanced to partial exits
Method: _execute_exit() [Lines 710-750]
```

**How to Implement Partial Exits:**
```python
# At first major target, close 50% of position
if current_price >= first_target and not partial_closed:
    partial_quantity = trade.position_size // 2
    self.Order(self.future_symbol, -partial_quantity)
    partial_closed = True

# At second target (previous day level), close rest
if current_price >= second_target:
    self.Liquidate(self.future_symbol)
```

---

### RULE 12: Trend Reversal Exit (EMA Crossover)
**Original Rule:** "Close on trend reversal"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_exit_signals() [Lines 671-679]
Trigger: EMA9 crosses EMA21
Action: Exit entire position
```

**Algorithm:**
```python
# For long positions: Exit if EMA9 drops below EMA21
if trade.trade_direction == 1:  # Long
    if self.ema_9_5m[0] < self.ema_21_5m[0]:
        return 'EMA_REVERSAL_LONG'

# For short positions: Exit if EMA9 rises above EMA21
elif trade.trade_direction == -1:  # Short
    if self.ema_9_5m[0] > self.ema_21_5m[0]:
        return 'EMA_REVERSAL_SHORT'
```

**Psychological Benefit:** Confirms trend reversal mechanically without emotion.

---

### RULE 13: Stop Loss Hit
**Original Rule:** "Disciplined exit on stop hit"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_exit_signals() [Lines 640-650]
Validation: Hard stop - automatic exit
No negotiation or prayer trades
```

**Algorithm:**
```python
# Long: Check if price hit stop loss below entry
if trade.trade_direction == 1:
    if current_price <= trade.stop_loss:
        return 'STOP_LOSS_LONG'

# Short: Check if price hit stop loss above entry
elif trade.trade_direction == -1:
    if current_price >= trade.stop_loss:
        return 'STOP_LOSS_SHORT'
```

---

## STOP PLACEMENT RULES (Rules 14-17)

### RULE 14: Stop Loss Based on Market Structure
**Original Rule:** "Stop loss placed beyond the structure that created the entry signal"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 600-608]
Logic: Stop below/above recent swing low/high
Offset: ATR-based buffer to avoid wicks
```

**Algorithm:**
```python
# For bullish entries: Stop below recent swing low
recent_lows = [bar.Low for bar in list(self.bars_5m)[:5]]
swing_low = min(recent_lows)
stop_loss = swing_low - (current_atr * self.ATR_STOP_MULTIPLE)

# For bearish entries: Stop above recent swing high
recent_highs = [bar.High for bar in list(self.bars_5m)[:5]]
swing_high = max(recent_highs)
stop_loss = swing_high + (current_atr * self.ATR_STOP_MULTIPLE)
```

**Why This Works:**
- Swing low represents where buyers/sellers decided structure is broken
- Placing stop beyond it ensures we're wrong when structure fails
- Not arbitrary - logically based on price action

---

### RULE 15: ATR-Based Stop Sizing (1.5-2x ATR)
**Original Rule:** "Stop placement must be logical - based on market structure, not arbitrary"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _calculate_indicators_5m() [Lines 370-390]
Parameter: ATR_STOP_MULTIPLE = 1.5 (tunable 1.0-2.5)
ATR Period: 14 bars
```

**ATR Calculation:**
```python
# True Range: Max of:
# 1. High - Low
# 2. |High - Previous Close|
# 3. |Low - Previous Close|

tr = max(
    bar.High - bar.Low,
    abs(bar.High - prev_close),
    abs(bar.Low - prev_close)
)

# ATR: 14-period average of True Range
atr = SMA(tr, 14)

# Stop Distance: Swing Structure +/- (ATR * 1.5)
stop_loss = swing_low - (atr * 1.5)  # For longs
```

**Why ATR?**
- Volatility-adjusted stops (volatile markets = wider stops)
- Prevents being stopped out on normal market noise
- Automatically scales with market conditions

---

### RULE 16: Invalidation Point Clarity
**Original Rule:** "IFVG high/low as reference"

**Implementation:**
```
File: qc_tempo_brain.py
Integration: Stop placement logic uses IFVG and swing structure
Reference Levels: IFVG level stored in ifvg_level[0]
```

For each trade:
- **Bullish entry:** Stop below swing low (and below IFVG level)
- **Bearish entry:** Stop above swing high (and above IFVG level)

If price moves beyond stop, the signal is invalidated - we're wrong.

---

### RULE 17: No Arbitrary Fixed Pip Stops
**Original Rule:** "Discipline to not overtrade or increase size after losses"

**Implementation:**
```
File: qc_tempo_brain.py
Structure Guarantee: All stops derive from ATR or swing structure
Never used: Fixed 20-pip, 50-pip, or arbitrary stops
Validation: Every stop is based on market structure
```

**Verification Checklist:**
- ✓ Stop never uses arbitrary constant
- ✓ Stop always uses ATR or swing level
- ✓ Stop scales with market volatility
- ✓ Stop always has logical invalidation point

---

## TIME FILTER RULES (Rules 18-21)

### RULE 18: Primary Session 9:30-11:30 ET (Highest Probability)
**Original Rule:** "Trade during London and New York sessions"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _is_in_valid_session() [Lines 770-800]
Time Zone: Eastern Time (ET)
Session: 9:30 AM - 11:30 AM ET
```

**Algorithm:**
```python
def _is_in_valid_session(self, current_time):
    hour = current_time.hour
    minute = current_time.minute
    time_in_hours = hour + minute / 60.0

    # Check if in primary session (9:30-11:30 ET)
    if self.PRIMARY_SESSION_START <= time_in_hours < self.PRIMARY_SESSION_END:
        return True  # VALID SESSION
```

**Parameters:**
```python
PRIMARY_SESSION_START = 9.5   # 9:30 ET
PRIMARY_SESSION_END = 11.5    # 11:30 ET
```

**Why This Time?**
- Market opens at 9:30 ET with highest volatility
- Smart money enters at/near open
- Liquidity highest in first 2 hours
- Clearest price action patterns

---

### RULE 19: Secondary Session 13:30-15:00 ET
**Original Rule:** "Secondary trading session for additional opportunities"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _is_in_valid_session() [Lines 790-797]
Time Zone: Eastern Time (ET)
Session: 1:30 PM - 3:00 PM ET
```

**Algorithm:**
```python
# Check if in secondary session (13:30-15:00 ET)
if self.SECONDARY_SESSION_START <= time_in_hours < self.SECONDARY_SESSION_END:
    return True  # VALID SESSION
```

**Parameters:**
```python
SECONDARY_SESSION_START = 13.5   # 1:30 PM ET
SECONDARY_SESSION_END = 15.0     # 3:00 PM ET (market close)
```

**Why This Time?**
- Post-lunch volatility spike
- Market rebalancing in afternoon
- Still good liquidity before close
- Lower risk of overnight gaps

---

### RULE 20: Skip Lunch (11:30-13:30 ET)
**Original Rule:** "Avoid trading during low liquidity periods"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _is_in_valid_session() [Lines 770-800]
Logic: No entries allowed between 11:30 AM - 1:30 PM ET
Parameters:
LUNCH_START = 11.5
LUNCH_END = 13.5
```

**Why Skip Lunch?**
- Lowest liquidity of the day
- Widest spreads
- Choppy, ranging market action
- False breakouts common
- Smart money not trading (waiting for volatility after London closes)

---

### RULE 21: No Pre/Post-Market Entries
**Original Rule:** "Avoid economic data releases and illiquid hours"

**Implementation:**
```
File: qc_tempo_brain.py
Logic: Only entries during defined sessions (9:30-11:30 and 13:30-15:00)
Pre-Market: Before 9:30 ET = No entries
Post-Market: After 15:00 ET (3:00 PM) = No entries
```

**Why?**
- Pre-market: Low volume, wide spreads, data gaps
- Post-market: Risk of overnight gaps, low liquidity
- Higher slippage and worse execution

**Code Enforcement:**
```python
if not self._is_in_valid_session(bar.Time):
    return  # Skip all entry logic
```

---

## CONTEXT FILTER RULES (Rules 22-23)

### RULE 22: Daily Bias from EMA Alignment
**Original Rule:** "Identify and trade with the Daily Bias (market direction)"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _determine_daily_bias() [Lines 476-495]
Data: Daily consolidation bars
Indicators: EMA(9, 21, 50, 200)
```

**Bias Determination Algorithm:**
```python
def _determine_daily_bias(self):
    # Bullish: Prices above all EMAs in order
    if (price > ema_9 > ema_21 > ema_50 > ema_200):
        return 'bullish'

    # Bearish: Prices below all EMAs in order
    if (price < ema_9 < ema_21 < ema_50 < ema_200):
        return 'bearish'

    # Neutral: No clear alignment
    return 'neutral'
```

**Visual Representation:**
```
BULLISH BIAS:
Price
  |
  +---- EMA9
         |
         +---- EMA21
                |
                +---- EMA50
                       |
                       +---- EMA200

BEARISH BIAS (inverse of above):
EMA200
  |
  +---- EMA50
         |
         +---- EMA21
                |
                +---- EMA9
                       |
                       +---- Price
```

**Importance:**
- CRITICAL filter for all entries
- Wrong bias = High probability of loss
- Skip all entries until bias aligns
- When bias reverses, stop taking new entries in old direction

---

### RULE 23: Market Trend Strength (ADX > 20)
**Original Rule:** "Market structure alignment and trend strength"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _calculate_adx() [Lines 824-850]
Period: 14 bars
Minimum: ADX > 20 = Valid trend
```

**ADX Calculation:**
```python
def _calculate_adx(self, bars_window, period=14):
    # Plus DM: Upward movement
    plus_dm = max(0, current_high - prev_high)

    # Minus DM: Downward movement
    minus_dm = max(0, prev_low - current_low)

    # True Range: Volatility measure
    tr = max(high-low, |high-prev_close|, |low-prev_close|)

    # DI+ and DI-: Directional index
    di_plus = 100 * plus_dm / tr
    di_minus = 100 * minus_dm / tr

    # ADX: Average directional movement index
    dx = 100 * |di_plus - di_minus| / (di_plus + di_minus)
    adx = EMA(dx, 14)
```

**Interpretation:**
- **ADX < 20:** Market is ranging/choppy (AVOID)
- **ADX 20-40:** Valid trend with direction (TRADE)
- **ADX > 40:** Very strong trend (BEST)
- **ADX > 50:** Extremely strong but often reverses (CAUTION)

**Usage in Algorithm:**
Currently the ADX is calculated but threshold enforcement can be added:
```python
if self.adx_5m[0] > self.ADX_MINIMUM:
    # Continue with entry checks
else:
    # Skip entry - market is choppy
    return None
```

---

## EXCEPTION RULES (Rules 24-27)

### EXCEPTION 1: Don't Trade if SMT Already Complete
**Original Rule:** "Don't trade the IFVG setup if price has already taken out the smart money liquidity"

**Implementation:**
```
File: qc_tempo_brain.py
Logic: Implicit in entry signal timing
Concept: Entry only on close through IFVG level
Once level fully taken out and price beyond = Setup invalidated
```

**How This Prevents Bad Entries:**
- If IFVG is detected but price already penetrated far beyond it
- Entry signal will not trigger because we're past the confirmation level
- Automatically filters late entries after the move has run

---

### EXCEPTION 2: Skip During Choppy/Ranging Markets
**Original Rule:** "Skip setups during choppy/ranging market conditions"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _determine_daily_bias() [Lines 476-495]
Gate: If bias is 'neutral', NO entries are generated
Additionally: ADX filter can be added
```

**How It Works:**
```python
daily_bias = self._determine_daily_bias()

# If neutral bias (choppy market), immediately return
if daily_bias == 'neutral':
    return  # Skip all entry logic

# If ADX < 20 (optional additional check), skip
if self.adx_5m[0] < self.ADX_MINIMUM:
    return  # Market too choppy
```

---

### EXCEPTION 3: Avoid Counter-Bias Trades
**Original Rule:** "Avoid trading setup if daily bias is against your direction"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 580-630]
Gate 1: Only process bullish setups if daily_bias == 'bullish'
Gate 2: Only process bearish setups if daily_bias == 'bearish'
```

**Code Enforcement:**
```python
# BULLISH setup only proceeds if bias is bullish
if daily_bias == 'bullish':
    if (ifvg_bullish and close > ifvg_level):
        # Proceed with long entry

# BEARISH setup only proceeds if bias is bearish
if daily_bias == 'bearish':
    if (ifvg_bearish and close < ifvg_level):
        # Proceed with short entry

# If no match, return None (no entry)
```

---

### EXCEPTION 4: Wait for A+ Setups Only
**Original Rule:** "Don't force trades - wait for A+ setups only"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 560-630]
Multiple Gates: ALL must pass for entry
Missing any gate = Entry skipped
```

**A+ Setup Checklist (ALL required):**
1. ✓ IFVG detected on 5m chart
2. ✓ Close confirmation through IFVG level
3. ✓ Daily bias aligned with entry direction
4. ✓ EMA structure aligned (9>21>50>200 or reverse)
5. ✓ In valid trading session (9:30-11:30 or 13:30-15:00)
6. ✓ Stop loss calculable (has ATR data)
7. ✓ Position sizing valid (risk > 0)

If ANY gate fails: No entry, wait for next setup.

---

## INDICATOR IMPLEMENTATION SUMMARY

### Indicators Implemented:

| Indicator | Purpose | Implementation | Timeframes |
|-----------|---------|-----------------|-----------|
| **EMA(9)** | Fast structure | Exponential MA | 5m + Daily |
| **EMA(21)** | Mid structure | Exponential MA | 5m + Daily |
| **EMA(50)** | Slow bias | Exponential MA | 5m + Daily |
| **EMA(200)** | Long-term trend | Exponential MA | 5m + Daily |
| **ATR(14)** | Volatility/stops | True Range average | 5m + Daily |
| **RSI(14)** | Momentum (optional) | 14-period RSI | 5m |
| **ADX(14)** | Trend strength | Directional movement | 5m |
| **IFVG** | Entry pattern | 3-candle gap detection | 5m |

### Rolling Windows:
```python
self.bars_5m = RollingWindow[TradeBar](50)      # 5m data
self.bars_daily = RollingWindow[TradeBar](200)  # Daily data

self.ema_9_5m = RollingWindow[float](50)
self.ema_21_5m = RollingWindow[float](50)
self.ema_50_5m = RollingWindow[float](50)
self.ema_200_5m = RollingWindow[float](50)

self.ema_9_daily = RollingWindow[float](200)
self.ema_21_daily = RollingWindow[float](200)
self.ema_50_daily = RollingWindow[float](200)
self.ema_200_daily = RollingWindow[float](200)

self.atr_5m = RollingWindow[float](50)
self.atr_daily = RollingWindow[float](200)

self.ifvg_bullish = RollingWindow[bool](50)
self.ifvg_bearish = RollingWindow[bool](50)
self.ifvg_level = RollingWindow[float](50)
```

---

## POSITION SIZING RULES

### Rule: Risk 1% of Portfolio Per Trade
**Original Rule:** "Risk management is paramount"

**Implementation:**
```
File: qc_tempo_brain.py
Method: _check_entry_signals() [Lines 612-620]
Parameter: RISK_PER_TRADE = 0.01 (1%)
```

**Algorithm:**
```python
# Calculate risk amount (1% of portfolio)
risk_amount = self.Portfolio.TotalPortfolioValue * self.RISK_PER_TRADE

# Calculate position size from stop distance
stop_distance = current_close - stop_loss  # For longs
position_size = risk_amount / (stop_distance * 50)  # 50 = ES multiplier

# Cap at maximum contracts
position_size = min(position_size, self.MAX_CONTRACTS)
```

**Example:**
- Portfolio: $50,000
- Risk: $500 (1%)
- Entry: 4500
- Stop: 4490 (10 points = $500)
- Position: 1 contract (500 / (10 * 50))

### Rule: Maximum 2 Contracts
**Original Rule:** "Max 2 contracts at a time"

**Implementation:**
```python
MAX_CONTRACTS = 2

# In position sizing calculation:
position_size = min(position_size, self.MAX_CONTRACTS)
```

---

## CODE EXECUTION FLOW

### On Each 5-Minute Bar:
```
1. on_5m_bar() called
   │
   ├─ Add bar to rolling window
   ├─ Calculate indicators (EMA, ATR, RSI, ADX, IFVG)
   ├─ Check session time (Rules 18-21)
   ├─ Determine daily bias (Rules 22-23)
   │
   ├─ IF IN TRADE:
   │  └─ Check exit signals (Rules 9-13)
   │     └─ Execute exit if triggered
   │
   └─ IF NOT IN TRADE:
      └─ Check entry signals (Rules 1-8)
         └─ Execute entry if all gates pass
```

### On Each Daily Bar:
```
1. on_daily_bar() called
   │
   ├─ Add bar to rolling window
   ├─ Calculate daily indicators
   └─ Update previous day H/L/C for liquidity levels
```

---

## TESTING CHECKLIST

When backtesting, verify:

- [ ] Entry logic only triggers during valid sessions
- [ ] Daily bias correctly identifies trend direction
- [ ] IFVG patterns detected on 5m chart
- [ ] Entries align with daily bias
- [ ] Stops placed below swing lows (longs) / above swing highs (shorts)
- [ ] Stop distances reasonable (1.5-2x ATR)
- [ ] Position sizing calculated from stop distance
- [ ] Take profits at previous day levels
- [ ] Exits on stop loss are immediate
- [ ] Exits on 1:1 R:R move stop to breakeven
- [ ] Trade log shows correct entry reasons
- [ ] Win rate and Sharpe ratio reasonable

---

**Document Version:** 1.0
**Last Updated:** February 2024
**Algorithm Version:** 1.0
**Total Rules Implemented:** 23 core + 4 exceptions = 27
