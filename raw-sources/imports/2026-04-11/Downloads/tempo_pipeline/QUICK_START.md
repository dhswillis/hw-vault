# Quick Start: Tempo Trading Brain on QuantConnect

Get your algorithm running in 5 minutes.

---

## The Fastest Path

### Step 1: Sign Up (2 minutes)
```
1. Go to https://www.quantconnect.com
2. Click "Sign Up"
3. Use email/GitHub to create account
4. Verify email
```

### Step 2: Copy Algorithm (1 minute)
```
1. Click "New Project" → "Algorithm" → "Python"
2. Name it "Tempo ES"
3. Delete default code
4. Copy entire contents of qc_tempo_brain.py and paste
5. Press Ctrl+S to save
```

### Step 3: Run Backtest (30 seconds)
```
1. Click "Backtest" button (top right)
2. Wait 30-60 seconds
3. View results
```

That's it. You now have a working backtest.

---

## What to Look For in Results

After backtest completes, check these numbers:

| Metric | Good | Great |
|--------|------|-------|
| **Win Rate** | >50% | >60% |
| **Sharpe Ratio** | >0.8 | >1.5 |
| **Max Drawdown** | <25% | <15% |
| **Total Return** | 10-50% | 50%+ |
| **Number of Trades** | 10+ | 20+ |

If your results match "Good" column or better: You're on track.

---

## First Customization: Change Dates

To backtest a different period:

In the algorithm code, find this section (around line 65):
```python
self.SetStartDate(2020, 1, 1)      # ← Change this
self.SetEndDate(2025, 12, 31)      # ← Change this
```

Change to your preferred range:
```python
self.SetStartDate(2023, 1, 1)      # Start 2023
self.SetEndDate(2024, 12, 31)      # End 2024
```

Then run backtest again.

---

## Second Customization: Adjust Risk

To trade with more or less risk per trade:

Find this line (around line 48):
```python
RISK_PER_TRADE = 0.01                 # 1% risk per trade
```

Change to:
- `0.005` for **smaller** positions (0.5% risk) - safer, slower gains
- `0.015` for **larger** positions (1.5% risk) - more aggressive

Example:
```python
RISK_PER_TRADE = 0.005  # Now risk only 0.5% per trade
```

Then run backtest again.

---

## Third Customization: Adjust Session Hours

The algorithm only trades during these hours (Eastern Time):
- **Morning:** 9:30 AM - 11:30 AM
- **Afternoon:** 1:30 PM - 3:00 PM

To trade different hours, find (around line 59):
```python
PRIMARY_SESSION_START = 9.5        # 9:30 ET
PRIMARY_SESSION_END = 11.5         # 11:30 ET
SECONDARY_SESSION_START = 13.5     # 1:30 PM ET
SECONDARY_SESSION_END = 15.0       # 3:00 PM ET
```

Change to your preferred times:
```python
PRIMARY_SESSION_START = 8.5        # 8:30 ET (starts earlier)
PRIMARY_SESSION_END = 16.0         # 4:00 PM ET (stays later)
SECONDARY_SESSION_START = 13.5
SECONDARY_SESSION_END = 15.0
```

Then run backtest again.

---

## Understanding the Algorithm in 30 Seconds

**What it does:**
1. Watches for a specific pattern (IFVG) on 5-minute charts
2. Checks if daily trend is going up or down
3. Enters trades that align with the daily direction
4. Places stops below recent swing lows
5. Exits at previous day's high/low or on stop loss

**How often it trades:**
- Typically 5-20 trades per month on ES
- Only trades during NY market hours
- Only trades when bias is clear (skips choppy markets)

**Why it's mechanical:**
- All signals are automatic (no subjective decisions)
- All rules are coded (no human discretion)
- Results are reproducible (same backtest = same results)

---

## Comparing Backtests

Want to compare two different parameter setups?

### Test 1: Original Settings
1. Run backtest with default parameters
2. Write down the results (or screenshot)

### Test 2: Modified Settings
1. Change one parameter (e.g., RISK_PER_TRADE)
2. Run backtest again
3. Compare results

**Which is better?**
- Higher Sharpe Ratio = Smoother returns
- Higher Win Rate = More consistent
- Lower Max Drawdown = Less scary
- Look at all three, not just return

---

## Troubleshooting in 2 Steps

**Problem: "No trades executed"**
1. Check if date range is too narrow
2. Expand PRIMARY_SESSION hours to include more of the day

**Problem: "Huge drawdowns (30%+)"**
1. Reduce RISK_PER_TRADE to 0.005 (half)
2. Run backtest again

**Problem: "Error in code"**
1. Check that you copied ENTIRE qc_tempo_brain.py file
2. Make sure no lines were cut off
3. Paste again, verify all 800+ lines are there

---

## Next Steps After First Successful Backtest

### If Results Look Good (Sharpe > 1.0, DD < 20%)
1. Run 2-3 more backtests with different date ranges
2. If all look good, paper trade for 1 week
3. If paper trading matches backtest, ready to trade for real

### If Results Look Weak (Sharpe < 0.5, DD > 30%)
1. Try adjusting parameters:
   - Reduce risk to 0.005
   - Increase session hours to trade more often
   - Change EMA periods (shorter = more signals)

2. Test on different time periods (2021, 2022, 2023, etc.)

3. If still poor, the strategy may need market conditions to match

---

## Reality Check

**Remember:**
- Backtest results are best-case scenario
- Live trading will be 10-20% worse due to slippage
- Paper trading first is critical
- Start with 1 micro contract for real trading
- Only scale up after 10+ winning weeks

**What to expect:**
- Not every trade is a winner
- Drawdowns happen (that's normal)
- The goal is positive expected value over 20+ trades
- Winning traders are right only 55-60% of the time
- Position sizing makes the difference

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `qc_tempo_brain.py` | Main algorithm (this is what you run) |
| `QC_SETUP_GUIDE.md` | Full detailed setup guide |
| `TEMPO_RULES_IMPLEMENTATION.md` | How each rule is coded |
| `QUICK_START.md` | This file (30-sec overview) |

---

## Most Important Things to Remember

1. **Session times matter** - Only trades 9:30-11:30 and 1:30-3:00 ET
2. **Daily bias is critical** - Won't enter counter to daily trend
3. **Position sizing is automatic** - Risk always 1% (unless you change it)
4. **Stops always in code** - Not arbitrary, based on ATR/structure
5. **Patience built in** - Skips bad setups, waits for A+ only

---

## Getting Help

**Quick questions:**
- Check TEMPO_RULES_IMPLEMENTATION.md (explains every rule)

**Setup problems:**
- Check QC_SETUP_GUIDE.md (has troubleshooting section)

**Algorithm questions:**
- Look at code comments (each section explains itself)
- Compare code to rules document

**QuantConnect help:**
- Forum: https://www.quantconnect.com/forum
- Docs: https://www.quantconnect.com/docs/v2/our-platform/overview

---

## Your First Week

**Day 1:** Sign up and run first backtest
**Day 2:** Try 2-3 different parameter combinations
**Day 3-4:** Read through TEMPO_RULES_IMPLEMENTATION.md to understand logic
**Day 5:** Paper trade (simulate trading without real money)
**Day 6:** Verify paper results match backtest
**Week 2:** If paper matches backtest, consider starting with 1 contract

---

## Pro Tips

**Tip 1:** Save backtest results
- Screenshot or download CSV of results
- Makes it easy to compare later

**Tip 2:** Test one parameter at a time
- Change ONLY risk, then run
- Change ONLY session hours, then run
- Don't change everything at once

**Tip 3:** Paper trade even if you trust backtest
- Paper trading shows real slippage
- You see how execution actually works
- Builds confidence before risking real money

**Tip 4:** Track your trades
- Keep a journal of live trades
- Compare to backtest performance
- Learn what conditions work best

---

**That's it. You're ready.**

Run the backtest. See if it works. Adjust parameters. Paper trade. Then decide if you want to trade for real.

The algorithm does the heavy lifting. Your job is to be disciplined and follow the rules.

Good luck.
