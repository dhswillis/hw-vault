# FVG Entry Timing Analysis: Tempo Trading System
**For:** Harrison
**Status:** Key Analysis
**Date:** 2026-02-17
**Document Purpose:** Bifurcated entry strategy for FVG signals in Model 1 (Sweep + IFVG) and Model 2 (BOS/VA/PGF)

---

## Executive Summary

The Tempo system's FVG entries must account for **how price is approaching the FVG**, not just whether an FVG exists. This document defines two distinct entry regimes:

1. **Passive Fill Regime** (Price retracing INTO FVG) → Limit order at FVG midpoint
2. **Momentum Inversion Regime** (Price blasting THROUGH FVG with momentum) → Market order on next bar confirmation

Current Model 1 implementation enters on the same bar FVG is detected. This timing issue explains performance gaps between Model 1 and Model 2. The fix: **detect the regime, then apply the appropriate entry method.**

---

## Part 1: The Two Entry Regimes Explained

### Regime A: Passive Fill (Price Retracing to FVG)

**What's happening:**
- Price created an imbalance (FVG)
- Price moved away from the FVG zone
- Price is now **returning to the FVG to fill the gap**
- Momentum is NOT pushing aggressively through the FVG

**Trading approach:**
- Place a **limit order at FVG midpoint**
- You are "fishing" — waiting for price to come to you
- Patience = better entry fills
- Lower urgency, higher fill certainty

**When to use:**
- FVG forms after a pullback/consolidation
- Price shows absorption/stalling below/above the FVG
- Candlestick patterns show indecision (doji, pin bar, spinning top)
- Volume is LOW at attempted FVG fill
- Price speed into FVG is slow/gradual

**Edge:** Model 2 already captures this pattern and validated +11.31R/day using BOS/VA/PGF signals before placing limit entries.

---

### Regime B: Momentum Inversion (Price Blasting Through FVG)

**What's happening:**
- Price is moving **aggressively THROUGH the FVG zone**
- Momentum is in the opposite direction of the FVG (inversing it)
- This is a **displacement move**, not absorption
- You cannot wait for a limit fill — the opportunity disappears fast

**Trading approach:**
- Detect on 1M, **confirm on 15s**, then **market order on next 15s candle**
- You are "chasing" but with conviction because momentum aligns with the trade direction
- Urgency is HIGH; fill quality is secondary
- You enter the move, not the level

**When to use:**
- FVG is detected, but price never pauses near it
- Price velocity into/through FVG is HIGH
- Candlestick at FVG entry is a momentum candle (strong close bias, no wick)
- Volume is HIGH during FVG fill attempt
- The 15s granularity shows the FVG is being "run through" not "absorbed into"

**Edge:** This is what Carmine Rosato calls "displacement through FVG" — requires market entry with momentum confirmation, not limit patience.

---

## Part 2: How to Detect Which Regime You're In

### Detection Framework

Use a **tiered decision tree** evaluated when FVG is first detected (in `_ScanForFVGs()`):

#### Level 1: Speed of Fill Attempt
```
Has price touched the FVG zone in the last 2 bars?
├─ NO → Price retracing, too far away → Passive regime (limit order prep)
└─ YES → Go to Level 2
```

#### Level 2: Candlestick Pattern at Entry Bar
```
What pattern formed on the bar that touched/entered FVG?
├─ Indecision candles (doji, spinning top, pin bar) → PASSIVE regime
├─ Momentum candles (marubozu, strong close, no wick) → MOMENTUM regime
└─ Neutral/small body candles → Check Level 3
```

#### Level 3: Volume Profile
```
Is volume at FVG entry HIGH or LOW relative to recent bars?
├─ LOW volume + price stalling → PASSIVE regime
├─ HIGH volume + price pushing through → MOMENTUM regime
└─ Medium volume → Check Level 4
```

#### Level 4: 5M Bar Context (V7aa finding)
```
Does the 5M bar at entry CONFIRM or CONTRADICT the signal direction?
├─ CONFIRMS direction → Can use MOMENTUM regime (more likely fill)
├─ CONTRADICTS direction → PASSIVE regime only (wait for absorption)
└─ Neutral → Default to PASSIVE regime (lower risk)
```

#### Level 5: Next Bar Candle Action (15s)
```
When next 15s bar completes after FVG detection on 1M:
├─ Does 15s candle move INTO/THROUGH the FVG?
│  ├─ If pushing through with body → MOMENTUM regime, market order
│  └─ If absorbed/stalling in FVG → PASSIVE regime, place limit
└─ Does 15s candle RETEST below the FVG (bullish) or above (bearish)?
   └─ Yes → PASSIVE regime, price coming back for fill
```

---

## Part 3: Data Evidence for Each Regime

### Passive Fill Regime Evidence

**From V7z-fix testing:**
- **1M granularity is optimal** for detecting FVG fills (not 15s — adds noise)
- "15S granularity made it WORSE. More noise, more false fills. 1M is the right timeframe."
- Implication: Wait for 1M completion to confirm passive fill pattern

**From V7aa testing (anti-chasing findings):**
- **Indecision candles at entry OUTPERFORM momentum candles** (opposite of intuition)
- "Doji_gravestone: 2.461 avg R" ← Best pattern at FVG entry
- "Hanging_man: 2.155 avg R" ← Strong second
- "Inverted_hammer: 1.813 avg R" ← Works well
- **"Marubozu_bull: -0.210 avg R"** ← LOSE money entering on momentum candles
- Insight: Indecision = absorption = passive fill working. Momentum = chasing = worse fill, more reversals.

**From V7r testing (BOS_FVG entry timing):**
- "15S candle at BOS_FVG entry: BOS_FVG + strong 15S confirm: 39.1% WR, 1.816 avg R (n=87) — BEST combo"
- "Counterintuitive: 'neutral' 15S candle outperforms confirmation overall (34.1% vs 31.1% WR)"
- "LOW volume at 15S entry beats HIGH (0.659 avg R vs 0.435 avg R)"
- Insight: Neutral/low-volume 15s entries = passive regime, best risk/reward

**Passive Regime Summary:**
- **Win Rate:** 31–34% (Model 2 baseline)
- **Average R:** 0.659–2.461 depending on entry pattern
- **Optimal candle:** Indecision/neutral (doji, pin bar, small body)
- **Optimal volume:** LOW at entry
- **Action:** Place limit at FVG midpoint, wait for fill

---

### Momentum Inversion Regime Evidence

**From V7aa (gate testing on Model 1):**
- "gate_5m_good_AND_no_contra is BEST PRACTICAL GATE: 33.0% WR, 1.369 avg R"
- The "5m_good" gate = 5M bar confirms direction (NO contradiction)
- Implication: When 5M aligns with signal, you can use momentum to enter
- But avoid entries where 5M contradicts signal (kills 118% Calmar improvement)

**From V7r (timing on next bar):**
- "Waiting 3×15S for confirmation: WR 34.6% vs 32.4% immediate, avgR 0.696 vs 0.585"
- Implication: Waiting 3 bars helps if momentum is strong (more confirmation = less chasing)
- But for momentum inversions, waiting hurts (price leaves without you)

**Momentum Regime Caveat (The Paradox):**
- Momentum candles at entry = -0.210 avg R (LOSE)
- BUT momentum FVGs (inversions) NEED immediate action
- **The solution:** Enter on momentum + 15s CONFIRMATION, not on 1M momentum candle alone
- Confirmation = 15s candle also shows momentum into/through FVG
- This filters out fake momentum and targets real displacement

**Momentum Regime Summary:**
- **Best gate:** 5M bar confirms direction (no contradiction)
- **Entry timing:** Detect on 1M, wait for next 15s confirmation, market order on 15s close
- **Optimal conditions:** HIGH volume, strong 15s candle, 5M aligned
- **Risk:** Chasing if used without 15s confirmation
- **Average R potential:** 1.369–1.816 when paired with proper confirmation

---

## Part 4: Code Changes Required in main_v4.py

### Current State

**Problem 1: Same-bar entry after FVG detection**
```python
# Current flow in OnBar1m (BAD)
def OnBar1m(self):
    self._ScanForFVGs()      # Detects FVG on bar[0], bar[1], bar[2]
    self._ProcessEntryBar()  # Enters on same bar[0] close
    # FVG detected and entered on SAME 1M candle — too fast
```

**Problem 2: No regime detection**
- System enters on every FVG without distinguishing passive vs momentum
- Indiscriminate entries hurt average R and win rate

**Problem 3: No 15s confirmation for momentum entries**
- Model 1 should wait for next 15s bar to confirm momentum before market order

---

### Recommended Code Architecture

**Step 1: Add FVG regime detection to `_ScanForFVGs()`**

```python
def _ScanForFVGs(self):
    """
    Detect FVGs AND classify regime (PASSIVE or MOMENTUM)
    """
    # Existing FVG detection logic
    fvgs = self._IdentifyFVGs()  # bars[0], bars[1], bars[2]

    for fvg in fvgs:
        # NEW: Classify regime
        regime = self._ClassifyFVGRegime(fvg)

        # Store FVG with regime flag
        self.pending_fvg = {
            'fvg': fvg,
            'regime': regime,  # 'PASSIVE' or 'MOMENTUM'
            'detected_on_bar': self.current_bar_index,
            'entry_level': self._CalculateFVGMidpoint(fvg)
        }

        if regime == 'PASSIVE':
            self._PrepareLimitEntry(fvg)
        elif regime == 'MOMENTUM':
            self._MonitorForMomentumConfirmation(fvg)

def _ClassifyFVGRegime(self, fvg):
    """
    Tiered decision tree for regime detection
    Returns: 'PASSIVE', 'MOMENTUM', or None
    """

    # Level 1: Is price already IN the FVG?
    price_in_fvg = self._IsPriceInFVGZone(fvg)
    if not price_in_fvg:
        return 'PASSIVE'  # Price is away, retracing expected

    # Level 2: Candlestick pattern at entry
    entry_bar = self.bars[0]
    pattern = self._IdentifyCandlePattern(entry_bar)

    if pattern in ['doji', 'spinning_top', 'pin_bar', 'gravestone_doji']:
        return 'PASSIVE'  # Indecision = absorption

    if pattern in ['marubozu_bull', 'marubozu_bear', 'strong_close']:
        # Don't return MOMENTUM yet; momentum candles alone are bad
        # Go to Level 3
        pass

    # Level 3: Volume profile
    entry_volume = entry_bar.volume
    avg_volume = self._Get20BarAverageVolume()

    if entry_volume < avg_volume * 0.8:
        return 'PASSIVE'  # Low volume = absorption

    if entry_volume > avg_volume * 1.2 and pattern in ['marubozu_bull', 'marubozu_bear']:
        # High volume + momentum candle = candidate for MOMENTUM regime
        pass

    # Level 4: 5M bar alignment (V7aa best gate)
    fvg_direction = 'BULL' if fvg['type'] == 'bullish' else 'BEAR'
    bar_5m = self._GetLast5MBar()
    bar_5m_direction = 'BULL' if bar_5m.close > bar_5m.open else 'BEAR'

    if bar_5m_direction != fvg_direction:
        return 'PASSIVE'  # Contradiction = must wait for absorption

    if bar_5m_direction == fvg_direction:
        # Level 5: This is a momentum candidate
        # But we need 15s confirmation before committing
        return 'MOMENTUM'

    # Default to passive if unclear
    return 'PASSIVE'
```

**Step 2: Separate entry handlers by regime**

```python
def _PrepareLimitEntry(self, fvg):
    """
    For PASSIVE regime: Set limit order at FVG midpoint
    NO entry yet — just prepare the order
    """
    entry_level = self._CalculateFVGMidpoint(fvg)

    # Place limit order
    self.limit_order = {
        'level': entry_level,
        'fvg': fvg,
        'active': True,
        'set_bar': self.current_bar_index,
    }

    # Check it on each 1M bar update (not immediately)
    # If price comes to limit, execute
    # If price moves away without touching, cancel after N bars

def _MonitorForMomentumConfirmation(self, fvg):
    """
    For MOMENTUM regime: Wait for next 15s bar confirmation
    Do NOT enter on 1M confirmation bar
    """
    self.pending_momentum_fvg = {
        'fvg': fvg,
        'detected_at_1m': self.current_bar_index,
        'awaiting_15s_confirm': True,
    }

    # This FVG will be re-evaluated in OnBar15s()

def _ProcessEntryBar(self):
    """
    MODIFIED: Only process PASSIVE limit orders here
    Remove direct market entry on FVG detection
    """
    # Check if any limit orders were filled
    if self.limit_order and self.limit_order['active']:
        if self._IsLimitOrderFilled(self.limit_order):
            self._ExecuteTradeAtLimit(self.limit_order)
            self.limit_order = None

def OnBar15s(self):
    """
    NEW: Handle momentum FVG confirmations on 15s bar
    """
    if hasattr(self, 'pending_momentum_fvg') and self.pending_momentum_fvg['awaiting_15s_confirm']:
        fvg = self.pending_momentum_fvg['fvg']

        # Check if 15s bar confirms momentum through FVG
        current_15s_bar = self.bars_15s[0]

        if self._Does15sConfirmMomentum(fvg, current_15s_bar):
            # YES: Price is moving through FVG with momentum
            # Enter on NEXT 15s bar close, not this one
            self.pending_momentum_fvg['confirmed'] = True
            self.pending_momentum_fvg['confirm_bar_index'] = self.current_15s_bar_index
        else:
            # NO: False signal, downgrade to passive
            self.pending_momentum_fvg['awaiting_15s_confirm'] = False
            self._PrepareLimitEntry(fvg)

def OnBar15sClose(self):
    """
    Execute momentum entries on next 15s bar AFTER confirmation
    """
    if (hasattr(self, 'pending_momentum_fvg') and
        self.pending_momentum_fvg.get('confirmed') and
        self.current_15s_bar_index > self.pending_momentum_fvg['confirm_bar_index']):

        # Market order on this 15s close
        fvg = self.pending_momentum_fvg['fvg']
        self._ExecuteMarketEntry(fvg)
        self.pending_momentum_fvg = None

def _Does15sConfirmMomentum(self, fvg, candle_15s):
    """
    Check if 15s candle shows momentum INTO/THROUGH the FVG
    """
    fvg_zone = (fvg['low'], fvg['high'])
    fvg_direction = 'BULL' if fvg['type'] == 'bullish' else 'BEAR'

    # Is the 15s candle INSIDE the FVG zone?
    candle_in_fvg = (
        (candle_15s.low >= fvg_zone[0] and candle_15s.high <= fvg_zone[1]) or
        (candle_15s.low < fvg_zone[0] and candle_15s.high > fvg_zone[0]) or
        (candle_15s.low < fvg_zone[1] and candle_15s.high > fvg_zone[1])
    )

    if not candle_in_fvg:
        return False

    # Is the 15s candle PUSHING THROUGH the FVG in the direction of the signal?
    if fvg_direction == 'BULL':
        # Bullish FVG: 15s should have strong close above FVG midpoint
        fvg_midpoint = (fvg_zone[0] + fvg_zone[1]) / 2
        momentum_signal = (candle_15s.close > fvg_midpoint and
                          candle_15s.close > candle_15s.open and
                          (candle_15s.close - candle_15s.open) > (candle_15s.high - candle_15s.close))
    else:
        # Bearish FVG: 15s should have strong close below FVG midpoint
        fvg_midpoint = (fvg_zone[0] + fvg_zone[1]) / 2
        momentum_signal = (candle_15s.close < fvg_midpoint and
                          candle_15s.close < candle_15s.open and
                          (candle_15s.open - candle_15s.close) > (candle_15s.close - candle_15s.low))

    return momentum_signal
```

**Step 3: Add helper functions**

```python
def _IdentifyCandlePattern(self, candle):
    """
    Classify candlestick patterns at FVG entry
    """
    body_size = abs(candle.close - candle.open)
    total_range = candle.high - candle.low
    upper_wick = candle.high - max(candle.open, candle.close)
    lower_wick = min(candle.open, candle.close) - candle.low

    if body_size < total_range * 0.3:
        # Small body
        if upper_wick > total_range * 0.4 and lower_wick > total_range * 0.4:
            return 'spinning_top'
        elif upper_wick > total_range * 0.5:
            return 'pin_bar'
        elif upper_wick > total_range * 0.4 and candle.close < candle.open:
            return 'gravestone_doji'
        else:
            return 'doji'
    elif body_size > total_range * 0.8:
        # Large body, minimal wicks
        if candle.close > candle.open:
            return 'marubozu_bull'
        else:
            return 'marubozu_bear'
    elif candle.close > candle.open and upper_wick < total_range * 0.2:
        return 'strong_close'
    else:
        return 'neutral'

def _Get20BarAverageVolume(self):
    """Get average volume from last 20 bars"""
    volumes = [bar.volume for bar in self.bars[-20:]]
    return sum(volumes) / len(volumes) if volumes else 0

def _IsLimitOrderFilled(self, limit_order):
    """Check if limit order has been filled"""
    current_price = self.current_bid  # or ask, depending on direction
    level = limit_order['level']
    return (current_price <= level)  # Adjust for direction

def _ExecuteTradeAtLimit(self, limit_order):
    """Execute trade when limit is filled"""
    self._PlaceMarketOrder(limit_order['fvg'], entry_method='LIMIT_FILL')

def _ExecuteMarketEntry(self, fvg):
    """Execute market order for momentum entry"""
    self._PlaceMarketOrder(fvg, entry_method='MOMENTUM_MARKET')

def _PlaceMarketOrder(self, fvg, entry_method):
    """Unified market order placement"""
    direction = 'LONG' if fvg['type'] == 'bullish' else 'SHORT'
    entry_price = self.last_price

    # Place order with entry method tag for analysis
    order = self._CreateOrder(
        direction=direction,
        entry_price=entry_price,
        fvg=fvg,
        entry_method=entry_method,
        stop_loss=self._CalculateStop(fvg),
        risk_reward_target=self._CalculateTarget(fvg),
    )

    self.active_trade = order
    return order
```

---

## Part 5: The Paradox — Why Momentum at Entry Kills Returns

### The Data Contradiction

From V7aa findings:
- **Entering on momentum candles loses money**: Marubozu = -0.210 avg R
- **Yet momentum FVGs (inversions) NEED immediate entry** to catch the move

How is this reconciled?

### The Solution: Entry Method, Not Candle Pattern

**The mistake:** Confusing "momentum candle at FVG entry" with "momentum confirming displacement."

**Correct interpretation:**
1. **Momentum candle ALONE** = Chasing the move without structural confirmation = BAD (-0.210 R)
   - You enter because price is moving fast, not because structure supports it
   - Price quickly reverses because the move was exhaustion

2. **Momentum through FVG with structural alignment** = Displacement confirming weakness = GOOD (1.3-1.8 R)
   - Price broke through FVG (displacement = weakness)
   - 15s candle shows momentum continuing (not a spike wick)
   - 5M bar aligns with signal direction (structural confirmation)
   - You enter the MOVE, not the level — but only after displacement is confirmed

**The filter that works:**
- **Reject entries where 5M bar contradicts signal** (V7aa: -118% Calmar improvement)
- **Require 15s candle confirmation showing momentum INTO FVG** (not just 1M)
- **Wait for next 15s bar** to enter (don't jump in on the confirming bar itself)

This explains why V7r found: "Neutral 15s candle outperforms confirmation (34.1% vs 31.1%)" — because you're entering on STABILITY after the momentum move, not during the spike.

### The Real Edge for Momentum FVGs

When an FVG is being inversed (displaced) with momentum:
- **Speed matters** — you need to catch it before price runs too far
- **But timing matters more** — entering on the momentum candle itself = buying the spike
- **Solution:** Confirm on 15s (displacement is real), wait for next 15s (momentum stabilizes), enter on that close (momentum confirmed, not chasing the spike)

This gives you:
- The edge of catching displacement before it's too late
- The fill quality of entering on confirmation, not the spike
- The structural alignment of 5M support/resistance confirming direction

---

## Part 6: Carmine Rosato Methodology Alignment

### Two Core Concepts from Carmine's Framework

#### Absorption (Price at FVG)
"The market absorbed at the FVG level — price spent time there and moved away"

**Tempo mapping:**
- This is PASSIVE regime
- Use **limit orders** at FVG midpoint
- Price comes back to fill = entry triggered
- Candle patterns show indecision = absorption is happening
- Volume is low = no urgency, absorption in progress

**From Carmine:** "When the market absorbs at an imbalance, we wait for price to return and enter on the retracement."

---

#### Displacement (Price Through FVG)
"The market blasted through the FVG — no absorption, just momentum"

**Tempo mapping:**
- This is MOMENTUM regime
- Use **market orders on next bar confirmation**
- Price never slows at FVG = displacement confirmed
- Candlestick patterns show strong closes = displacement moving
- Volume is high = urgency, displacement in progress

**From Carmine:** "When the market displaces through an imbalance with momentum, you must enter the displacement move before the runner completes."

---

### Integration into Tempo

**Carmine's method → Tempo implementation:**

| Carmine Concept | Tempo Detection | Entry Method | Timing | Pattern |
|---|---|---|---|---|
| Absorption at FVG | Low volume, indecision candle, price stalling | Limit at midpoint | Wait for fill | Doji, pin bar, stalling |
| Displacement through FVG | High volume, strong close, aggressive move | Market on 15s confirm | Next bar | Strong body, minimal wick |
| No absorption, run away | Price never touches FVG | Skip entry | N/A | Momentum away from level |
| Retracement back to FVG | Price moving back after displacement | Switch to limit | When price returns | Reversal structure |

---

## Part 7: Implementation Recommendations

### Phase 1: Immediate (This Week)

**1. Add regime classification to main_v4.py**
- Implement `_ClassifyFVGRegime()` function
- Log detected regimes (PASSIVE, MOMENTUM) for 5 days
- Compare performance: do PASSIVE trades outperform MOMENTUM?
- **Expected result:** Passive trades show higher win rate, better average R

**2. Split entry handlers**
- Separate `_PrepareLimitEntry()` for PASSIVE regime
- Create `_MonitorForMomentumConfirmation()` for MOMENTUM regime
- Stop entering on same 1M bar — defer to next bar at minimum

**3. Add 15s confirmation for momentum**
- Implement `OnBar15s()` callback if not already present
- Add `_Does15sConfirmMomentum()` function
- Require 15s bar to show momentum into/through FVG before market entry
- **Expected improvement:** +3-5% win rate by filtering false momentum entries

### Phase 2: Short-term (Next 2 Weeks)

**4. Validate with walk-forward testing**
- Run 5-day walk-forward with bifurcated entries
- Compare:
  - All entries (current)
  - PASSIVE only (limit orders)
  - MOMENTUM only (15s market)
  - Bifurcated (regime-based)
- **Target:** Bifurcated > all entries > PASSIVE-only > MOMENTUM-only

**5. Test V7aa gates on MOMENTUM entries**
- Require "5M bar confirms direction" gate
- Reject entries where "5M contradicts direction"
- This should kill worst 20% of momentum trades
- **Expected improvement:** +15-25% average R on momentum trades

**6. Profile candlestick patterns at live entry bars**
- Track: Doji vs pin bar vs marubozu at each entry
- Correlate pattern with trade outcome
- Validate V7aa finding that indecision > momentum at entry

### Phase 3: Optimization (3-4 Weeks)

**7. Dynamic stop placement**
- Use V7z-fix finding: "FOLLOW with dynamic stops is very profitable" for large wick sweeps
- Implement trailing stops for momentum displacement trades
- Static stops for passive limit entries

**8. FVG fill timeouts**
- For PASSIVE regime: How long to wait for limit fill?
- Test: Cancel after 5 bars, 10 bars, 15 bars
- Some FVGs never fill = skip them early

**9. Multiple FVGs in same timeframe**
- When multiple FVGs exist, which regime takes priority?
- Test: Smallest FVG, closest FVG, most recent FVG, most aligned with 15m trend

---

## Part 8: Key Metrics to Track

Once bifurcated entry is live, track these metrics separately:

### For PASSIVE Regime (Limit Entries)
- **Fill rate:** % of limit orders actually filled
- **Win rate:** Should be 31-34% based on V7r baseline
- **Average R:** Target 0.659-2.461 depending on candle pattern
- **Time in trade:** How long until limit is filled? (Longer is OK)
- **Candle pattern distribution:** Doji % vs pin bar % vs neutral %

### For MOMENTUM Regime (Market Entries)
- **Win rate:** Target 33%+ when using "5m_good_AND_no_contra" gate
- **Average R:** Target 1.3-1.8R when properly confirmed
- **False signal rate:** % of momentum entries that reverse immediately
- **15s confirmation accuracy:** How often does 15s confirmation actually predict successful momentum?
- **Slippage:** What's the actual fill vs ideal fill price on market orders?

### Cross-Regime
- **Total win rate:** Should improve to 33-35% if bifurcation works
- **Total average R:** Should improve to 0.8-1.2 if regime detection is accurate
- **Regime distribution:** % PASSIVE vs % MOMENTUM detected
- **Regime accuracy:** Do PASSIVE trades actually outperform MOMENTUM trades consistently?

---

## Part 9: Common Pitfalls to Avoid

### Pitfall 1: Premature Exit from Passive Regime
**Problem:** Seeing price gap away from FVG, thinking the trade is dead.
**Solution:** Remember — retracements take time. A 20-bar, 30-bar wait for a fill is normal. Don't cancel limits too early.

### Pitfall 2: Market Entering Too Late on Momentum
**Problem:** Waiting for perfect 15s confirmation, but price is already 10 pips into the move.
**Solution:** The 15s bar that **confirms** the momentum (2nd touch into FVG) is when you should have your order ready. Enter on the **next** 15s bar close, not the confirming bar.

### Pitfall 3: Treating Every FVG as Momentum
**Problem:** Detecting an FVG and immediately thinking it's displacement.
**Solution:** Use the tiered decision tree (speed, candle pattern, volume, 5M alignment). Default to PASSIVE unless multiple signals align.

### Pitfall 4: Ignoring the 5M Gate
**Problem:** Taking momentum FVG entries even when 5M bar contradicts direction.
**Solution:** The V7aa data is clear: contradicting 5M kills -118% Calmar. Make this a hard filter, not a soft suggestion.

### Pitfall 5: Entering on the Confirming 15s, Not the Next One
**Problem:** When 15s confirms momentum, immediately market order on that close.
**Solution:** Wait for the next 15s bar to close. This gives you confirmation + stability without chasing the spike.

---

## Summary: Decision Tree for Traders

When you detect an FVG:

```
1. Does price speed into FVG zone immediately? (next 1-2 bars)
   NO  → PASSIVE: Price will retrace, place limit at FVG midpoint
   YES → Check #2

2. What's the candlestick pattern at FVG?
   Indecision (doji, pin bar) → PASSIVE: Absorption happening
   Momentum (marubozu, strong close) → Check #3

3. What's the volume compared to recent bars?
   LOW volume → PASSIVE: Stalling at level, absorption confirmed
   HIGH volume → Check #4

4. Does the 5M bar align with the signal direction?
   NO (contradicts) → PASSIVE: Must wait for absorption
   YES (aligns) → Check #5

5. Has the next 15s bar confirmed momentum INTO/THROUGH the FVG?
   NO → Downgrade to PASSIVE
   YES → MOMENTUM: Market order on next 15s bar close
```

---

## Conclusion

The bifurcated entry approach solves the timing problem in Model 1:

**Current issue:** Entering on the same 1M bar FVG is detected = catching the spike, not the move
**Bifurcated solution:**
- PASSIVE regime = Wait for limit fill (Model 2 approach, proven +11.31R/day)
- MOMENTUM regime = Confirm on 15s, enter on next bar (catches displacement without chasing)

This aligns Tempo with Carmine Rosato's methodology and matches the empirical findings from V7r, V7t, V7aa, and V7z backtests.

**Expected outcome:**
- Win rate improvement to 33-35% (from lower in Model 1)
- Average R improvement to 0.8-1.2+ (from divergent performance)
- Reduced drawdown and smoother equity curve due to regime-appropriate entries

The implementation requires separating limit order logic (PASSIVE) from market entry logic (MOMENTUM + 15s confirmation), adding regime detection, and tracking metrics separately for each regime.

---

**Document Status:** Ready for implementation discussion with Harrison
**Next Steps:** Code implementation review, walk-forward validation (Phase 1-2), live testing with regime metrics tracking
