# KNOWLEDGE BASE - Harrison Willis Trading System
## Last Updated: February 10, 2026

---

## SYSTEM OVERVIEW

Harrison is building an autonomous NQ/ES futures trading system based on the Tempo Trading methodology. The system uses multiple "brains" (strategy rule sets) with Tempo as the primary strategy. The goal is automated signal generation and eventually automated execution across 15-20 prop firm accounts via Replikanto copy trading.

### Target Performance
- **Win rate:** 70%+ (Tempo V3 achieves 88% manually)
- **Average win:** 30-50 pts NQ
- **Average loss:** 15-20 pts NQ
- **Trades per day:** 1-2 (A/A+ setups only)
- **Daily target:** 3R across 6-8 smaller trades (all strategies combined)
- **EV target:** +1 per trade minimum

### Strategies (Brains)
1. **Tempo** - Primary. IFVG + SMT + Sweep. See docs/brains/TEMPO.md
2. **Lanto** - Secondary. (Rules TBD)
3. **Carmine** - Secondary. (Rules TBD)
4. **Omega** - Secondary. (Rules TBD)
5. **Flipping Markets** - Secondary. (Rules TBD)

---

## CURRENT STATE

### Backtesting Platform
- **QuantConnect** - Used for backtesting ONLY (not production)
- Project ID: 28083727
- QC API Auth: SHA256(token + ":" + timestamp), Basic auth header
- File size limit: 64,000 characters per file

### Production Architecture (Target)
- **Option 4: Hybrid Full Stack**
- Signal generation on VPS (OpenClaw + Claude API)
- Execution via Quantower + Replikanto (copy trading)
- 15-20 prop firm accounts simultaneously

### Version History
| Version | WR | Trades | Return | Period | Notes |
|---------|-----|--------|--------|--------|-------|
| V3.0 | 39% | - | -19.45% | 2024-2025 | Initial implementation |
| V3.1 | 21% | 210 | -11.42% | 2024-2025 | Refinement pass |
| V3.2 | 50% | 40 | -1.30% | 2024-2025 | Getting closer |
| V3.3 | 62% | 492 | +14.10% | 2024-2025 | Best so far, 2-year period |
| V3.4 | 41% | 37 | -20.36% | 2025 only | REGRESSION - too few trades, bugs |

### V3.4 Issues (Needs Fixing)
1. **Only 37 trades in 12 months** - Should be 200-300+
2. **Short bias**: 30 shorts vs 7 longs - Long entry path likely broken
3. **Stops too wide**: Avg loss 32 pts vs avg win 13.6 pts (inverted R:R)
4. **3-month gap**: May-July had ZERO trades - Possible contract rollover bug
5. **Nov-Dec gap**: Also zero trades
6. Monthly breakdown: Feb(2), Mar(5), Apr(13), May-Jul(0), Aug(6), Sep(7), Oct(4), Nov-Dec(0)

### Root Cause Hypotheses
- Contract rollover (March/June/September quarterly rolls) may break level tracking
- Sweep detection may be biased toward highs (producing shorts) not lows
- Combined sweep + IFVG + quality >= 3 filters are too restrictive together
- IFVG stop placement creates stops that are structurally too wide (32pt avg)
- Long entry code path may have a bug preventing triggers

---

## KEY TEMPO RULES (Quick Reference)

### Entry Checklist (ALL must be true)
1. Within kill zone: 9:35 AM - 11:00 AM ET
2. Liquidity sweep detected at session level (London H/L, Asia H/L, prev day H/L)
3. IFVG forms on 30s/1m/2m/3m timeframe
4. IFVG gap size: 5-30 pts (optimal 8-15)
5. Candle CLOSES through gap (not just wick)
6. Quality score >= 3 confluences
7. SMT divergence present (NQ vs ES)
8. Not within 30min of high-impact news
9. Not already at 2 losses for day
10. Daily bias confirmed (premium/discount framework)

### Stop Loss
- Preferred: Behind IFVG gap level (structural)
- Typical: 20-30 pts
- Soft stops preferred (90% of trades) - exit on candle CLOSE past level
- Hard stops only on funded accounts or walk-away
- **Actual Tempo V3 average loss: 15-20 pts** (our algo is averaging 32 - TOO WIDE)

### Trim Strategy (V3 Conservative)
- TP1: 15-20 pts -> trim 55% of position
- Break-even: Move stop to entry after 15-20 pts profit
- TP2: 30-40 pts -> trim 35%
- Runner: Hold remaining for major targets (used sparingly in V3)

### SMT (Smart Money Technique)
- NQ vs ES divergence at key levels
- Bullish: NQ sweeps low but ES doesn't
- Bearish: ES sweeps high but NQ doesn't
- V3: SMT is PRIMARY confluence (without it, WR drops from 80% to 60%)

---

## CONTEXT VARIABLES (Updated each session)

### Market Regime
- VIX level: [CHECK DAILY]
- Current NQ range: [CHECK DAILY]
- Trend day probability: [CHECK DAILY]

### Session Levels (Set pre-market)
- London High: [SET DAILY]
- London Low: [SET DAILY]
- Asia High: [SET DAILY]
- Asia Low: [SET DAILY]
- Previous Day High: [SET DAILY]
- Previous Day Low: [SET DAILY]
- Previous Day Close: [SET DAILY]
- 50% Level (Yellow Line): [SET DAILY]

### Active Signals
- [Updated in real-time by monitoring system]

---

## OPERATIONAL RULES

### Never Do
- Trade within first 2-5 minutes of market open
- Trade during FOMC/Powell/Trump speeches
- Trade on bank holidays or Christmas week
- Revenge trade after 2 losses
- Use hard stops in volatile conditions
- Enter without SMT confirmation on funded accounts
- Chase pre-market setups

### Always Do
- Check higher timeframes (4H -> 1H -> 15m) before entry
- Confirm sweep BEFORE looking for IFVG
- Go break-even after 15-20 pts profit
- Lock funded accounts after daily target hit
- Log every trade with full context
- Wait for A-setup after any loss
