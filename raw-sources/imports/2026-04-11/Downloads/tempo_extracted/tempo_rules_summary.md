# Tempo Trading System - Rule Summary

*Generated: 2026-02-09 16:12:38*

## Summary Statistics

**Total Rules Extracted: 23**
- Entry Conditions: 3
- Exit Rules: 3
- Stop Placement: 2
- Position Sizing: 3
- Time Filters: 3
- Market Context: 3
- Exceptions: 4
- Indicator Rules: 4

---

## Entry Conditions

### Rule 1
- **Text**: Enter when price breaks and closes above a valid IFVG (Internal Fair Value Gap)
- **Category**: Entry
- **Confidence**: 0.95
- **Conditions**: Price must close above the IFVG level, Confirmation candle following break, Institutional manipulation shown with SMT (Supply/Manipulation Test)
- **Parameters**: {"minimum_candle_size": "varies", "break_confirmation": "close above level"}

### Rule 2
- **Text**: IFVG setup on daily and lower timeframes as primary entry mechanism
- **Category**: Entry
- **Confidence**: 0.93
- **Conditions**: Internal Fair Value Gap identified, Multiple timeframe confluence, SMT (Supply/Manipulation Test) confirmation
- **Parameters**: {"timeframes": ["Daily", "4H", "1H"], "validation_method": "Multi-timeframe confirmation"}

### Rule 3
- **Text**: Enter after break of balance/equilibrium structure
- **Category**: Entry
- **Confidence**: 0.88
- **Conditions**: Market structure shift identified, Balance broken with momentum, Follow institutional direction
- **Parameters**: {"structure_type": "Balance/Equilibrium", "momentum_requirement": "Clear directional move"}

## Exit Rules

### Rule 1
- **Text**: Take profit at significant liquidity levels (supply/demand zones, previous highs/lows)
- **Category**: Exit
- **Confidence**: 0.92
- **Conditions**: Price reaches identified liquidity level, Historical support/resistance confirmed, No reversal signals at level
- **Parameters**: {"target_types": ["Previous highs", "Previous lows", "Supply zones", "Demand zones"], "partial_profit": "Common strategy mentioned"}

### Rule 2
- **Text**: Breakeven stop strategy helps more than it hurts
- **Category**: Exit
- **Confidence**: 0.85
- **Conditions**: Trade moved 5-10 pips in profit direction, Move stop to entry price, Removes downside risk while retaining upside
- **Parameters**: {"trigger_point": "5-10 pips profit", "adjustment": "Stop to entry price"}

### Rule 3
- **Text**: Close partial position at major resistance/support before continuation
- **Category**: Exit
- **Confidence**: 0.82
- **Conditions**: Trade in profit, Approaching known resistance or support, Reduce risk while maintaining position
- **Parameters**: {"partial_amount": "Variable", "remaining_exposure": "Continue with reduced risk"}

## Stop Placement

### Rule 1
- **Text**: Stop loss placed beyond the structure that created the entry signal
- **Category**: Stop Loss
- **Confidence**: 0.94
- **Conditions**: IFVG high/low as reference, SMT (Supply Manipulation Test) extreme as reference, Clear invalidation point
- **Parameters**: {"reference_structure": "IFVG or SMT extreme", "offset_pips": "2-5 pips beyond"}

### Rule 2
- **Text**: Stop placement must be logical - based on market structure, not arbitrary
- **Category**: Stop Loss
- **Confidence**: 0.91
- **Conditions**: Structure-based logic only, No arbitrary fixed pip stops, Invalidation point clear
- **Parameters**: {"method": "Structure-based", "discipline": "Strictly required"}

## Position Sizing

### Rule 1
- **Text**: Risk management is paramount - control position size to protect account
- **Category**: Position Sizing
- **Confidence**: 0.96
- **Conditions**: Risk per trade defined before entry, Position size calculated from stop distance, Account size considered
- **Parameters**: {"risk_per_trade": "Risk management focus", "calculation": "Account size * Risk % / Stop distance in pips"}

### Rule 2
- **Text**: Discipline to not overtrade or increase size after losses
- **Category**: Position Sizing
- **Confidence**: 0.89
- **Conditions**: Consistent position sizing across trades, No revenge trading, No scaling up after drawdown
- **Parameters**: {"consistency": "Required", "emotional_control": "Critical success factor"}

### Rule 3
- **Text**: Small positions during learning phase
- **Category**: Position Sizing
- **Confidence**: 0.78
- **Conditions**: New traders recommended small size, Build consistency before scaling, Error margin for execution mistakes
- **Parameters**: {"recommendation": "Micro/Mini lots for education", "scaling_threshold": "Consistent profitability demonstrated"}

## Time Filters

### Rule 1
- **Text**: Trade during London and New York sessions for ES and NQ
- **Category**: Time Filter
- **Confidence**: 0.9
- **Conditions**: Higher volatility in these sessions, Better liquidity for entries/exits, Clearer price action
- **Parameters**: {"sessions": ["London Open", "New York Open"], "instruments": ["ES (E-mini S&P 500)", "NQ (E-mini Nasdaq)"], "typical_hours": "8:00-15:00 ET"}

### Rule 2
- **Text**: London Gold and Oil trade London session primarily
- **Category**: Time Filter
- **Confidence**: 0.88
- **Conditions**: Gold (XAU/USD) trades well London session, Oil (WTI/Brent) trades well London session, Volatility and liquidity best during UK hours
- **Parameters**: {"instruments": ["Gold", "Oil/WTI"], "session": "London (8:00-17:00 GMT)"}

### Rule 3
- **Text**: Avoid trading during low liquidity periods and major news events
- **Category**: Time Filter
- **Confidence**: 0.87
- **Conditions**: Avoid economic data releases, Avoid central bank announcements, Avoid illiquid hours for specific instruments
- **Parameters**: {"avoid": ["Economic calendar events", "Central bank meetings", "Low volume hours"]}

## Context Filters

### Rule 1
- **Text**: Identify and trade with the Daily Bias (market direction)
- **Category**: Market Context
- **Confidence**: 0.94
- **Conditions**: Determine if market biased up or down on daily, Trade only in direction of bias, Reversal when bias breaks
- **Parameters**: {"timeframe": "Daily", "determination": "Based on structure and recent price action", "adherence": "Critical to consistency"}

### Rule 2
- **Text**: Trade setups that align with daily bias have higher probability
- **Category**: Market Context
- **Confidence**: 0.92
- **Conditions**: Bias direction must match trade direction, Counter-bias trades risky, Wait for new bias confirmation
- **Parameters**: {"win_rate_improvement": "Trading with bias vs against", "risk_reward": "Better in direction of bias"}

### Rule 3
- **Text**: Market structure determines setups (up, down, ranging)
- **Category**: Market Context
- **Confidence**: 0.91
- **Conditions**: Uptrend - higher lows and highs, Downtrend - lower lows and highs, Range - support/resistance clear
- **Parameters**: {"market_types": ["Uptrend", "Downtrend", "Range"], "setup_selection": "Based on structure"}

## Exceptions

### Rule 1
- **Text**: Don't trade the IFVG setup if price has already taken out the smart money liquidity
- **Category**: Exception
- **Confidence**: 0.89
- **Conditions**: SMT (Supply Manipulation Test) already complete, Entry opportunity already passed, Setup invalidated by further price movement
- **Parameters**: {"invalidation_type": "SMT completion", "missed_entry": "Wait for next setup"}

### Rule 2
- **Text**: Skip setups during choppy/ranging market conditions
- **Category**: Exception
- **Confidence**: 0.84
- **Conditions**: Market lacks clear direction, False breaks common, Risk/reward unfavorable
- **Parameters**: {"market_type": "Choppy/Range", "action": "Wait for clear bias"}

### Rule 3
- **Text**: Avoid trading setup if daily bias is against your direction
- **Category**: Exception
- **Confidence**: 0.9
- **Conditions**: Your setup suggests short but daily is bullish, Or setup suggests long but daily is bearish, Counter-bias trades low probability
- **Parameters**: {"check": "Always confirm daily bias first", "compliance": "Essential for consistency"}

### Rule 4
- **Text**: Don't force trades - wait for A+ setups only
- **Category**: Exception
- **Confidence**: 0.93
- **Conditions**: Setup must meet ALL confluence criteria, Multiple confirmations required, Patience is core discipline
- **Parameters**: {"trade_quality": "A+ setups only", "pass_rate": "Skip most setup opportunities"}

## Indicator Rules

### Rule 1
- **Text**: Internal Fair Value Gap (IFVG) is the primary indicator/pattern
- **Category**: Indicator
- **Confidence**: 0.96
- **Conditions**: Three-candle pattern where middle candle creates gap, Gap is unfilled area between candles, Represents imbalance and liquidity void
- **Parameters**: {"calculation": "Gap between high of first candle and low of third candle (if middle is larger)", "timeframes": ["1H", "4H", "Daily"], "significance": "Primary entry signal"}

### Rule 2
- **Text**: Supply Manipulation Test (SMT) - price tests IFVG then rejects it
- **Category**: Indicator
- **Confidence**: 0.91
- **Conditions**: Price goes into IFVG area, Creates false break (enters gap area), Then rejects and reverses strongly
- **Parameters**: {"purpose": "Confirms smart money interest", "action": "Enter on confirmation of rejection", "significance": "High probability signal"}

### Rule 3
- **Text**: No other indicators required - price action and structure are sufficient
- **Category**: Indicator
- **Confidence**: 0.88
- **Conditions**: IFVG + SMT + Bias form complete system, Additional indicators may cause confusion, Keep it simple and mechanical
- **Parameters**: {"system": "Price action based", "indicators": "None needed beyond IFVG/SMT", "discipline": "Mechanical execution required"}

### Rule 4
- **Text**: Confluence increases probability - multiple timeframe IFVGs align
- **Category**: Indicator
- **Confidence**: 0.9
- **Conditions**: IFVG on 1H AND 4H AND Daily, All three point to same area, Higher probability setup
- **Parameters**: {"confluence_levels": 2, "minimum_timeframes": 2, "boost_effect": "Significantly increases win rate"}

---

## Key Insights

### Core Trading Framework
The Tempo Trades system is built on **Internal Fair Value Gaps (IFVG)** as the primary entry mechanism. The system emphasizes:

1. **Price Action Based** - No fancy indicators needed, just IFVG + Supply Manipulation Tests + Daily Bias
2. **Mechanical Execution** - The model is binary and mechanical, leaving little room for emotional decisions
3. **Strict Discipline** - Patience to wait for A+ setups only; skip most opportunities
4. **Risk Management** - Position sizing and stop placement are critical - "self control separates profitable from non-profitable traders"

### Highest Confidence Rules
- Rule placement (confidence: 0.96) - Position Sizing: Risk management is paramount
- IFVG indicator (confidence: 0.96) - The primary indicator/pattern
- Entry: IFVG break (confidence: 0.95) - Core entry mechanism
- Stop placement (confidence: 0.94) - Structure-based, logical stops
- Daily bias (confidence: 0.94) - Trade direction matters significantly

### Critical Exceptions
Do NOT trade when:
- Daily bias is against your setup direction
- Already taken out smart money liquidity (SMT complete)
- Market is choppy/ranging
- Setup doesn't meet A+ criteria

### Instruments
- ES (E-mini S&P 500) - Trade London/NY sessions
- NQ (E-mini Nasdaq) - Trade London/NY sessions
- Gold (XAU/USD) - Trade London session
- Oil (WTI/Brent) - Trade London session

### Success Factors Mentioned
- Discipline and patience are emphasized repeatedly
- Psychology and self-control are more important than the model itself
- Consistency beats perfection - many members report profitability after understanding and following the system
- The model is simple but requires strict adherence - "easy bread but most importantly the knowledge"

---

## Model Completeness

This extraction successfully captured the Tempo Trades model's core framework across all 8 categories. The system is:
- **Mechanical**: Clear rules for entry, exit, stops, and position sizing
- **Structured**: Based on market structure and price action, not indicators
- **Discipline-Focused**: Repeated emphasis on patience, risk management, and psychological control
- **Well-Defined**: Specific rules for what to do and (more importantly) what NOT to do
