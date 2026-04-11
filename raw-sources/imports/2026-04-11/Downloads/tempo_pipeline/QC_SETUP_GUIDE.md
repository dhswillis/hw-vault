# QuantConnect LEAN Algorithm Setup Guide
## Tempo Trading Brain for ES Futures Backtesting

This guide walks you through setting up and running the Tempo Trading Brain algorithm on QuantConnect.

---

## Table of Contents
1. [Creating a QuantConnect Account](#creating-a-quantconnect-account)
2. [Setting Up the Algorithm](#setting-up-the-algorithm)
3. [Running Your First Backtest](#running-your-first-backtest)
4. [Interpreting Results](#interpreting-results)
5. [Modifying Parameters](#modifying-parameters)
6. [Troubleshooting](#troubleshooting)

---

## Creating a QuantConnect Account

### Step 1: Sign Up
1. Go to [quantconnect.com](https://www.quantconnect.com)
2. Click **"Sign Up"** in the top-right corner
3. Choose your registration method:
   - Email/Password (recommended for privacy)
   - GitHub account (fastest)
   - Google/other SSO

### Step 2: Email Verification
- Check your email and click the verification link
- This activates your free account

### Step 3: Account Activation
- QuantConnect's free tier includes:
  - Unlimited backtests
  - Access to 15+ years of historical data (US stocks, crypto, futures, forex)
  - 8 CPU-hours per month for live trading
  - Community support
  - No credit card required

### Step 4: First Time Login
- You'll be directed to the IDE (Integrated Development Environment)
- Create your first project by clicking "New Project"

---

## Setting Up the Algorithm

### Step 1: Create a New Project
1. Log in to QuantConnect
2. Click **"New Project"** (or go to Dashboard > Projects)
3. Select **"Algorithm"** (Python)
4. Name your project: `Tempo Trading Brain ES` (or preferred name)
5. Click **"Create Project"**

### Step 2: Upload the Algorithm File
The algorithm code is located at: `qc_tempo_brain.py`

**Option A: Copy-Paste Method (Easiest)**
1. Open the `qc_tempo_brain.py` file in your text editor
2. In QuantConnect IDE, select all code (Ctrl+A) and delete
3. Paste the entire algorithm code
4. Press Ctrl+S to save

**Option B: Upload via CLI**
If you have QuantConnect CLI installed:
```bash
lean push --project-name "Tempo Trading Brain ES" --file qc_tempo_brain.py
```

**Option C: File Manager**
1. In the IDE, right-click the project files
2. Select "Upload File"
3. Select `qc_tempo_brain.py`
4. Set as main algorithm file

### Step 3: Verify Algorithm Setup
- Check that the algorithm is syntactically correct (IDE will show errors)
- Common issues:
  - Missing imports: Ensure `from AlgorithmImports import *` is the first import
  - Indentation errors: Python requires consistent indentation (4 spaces)
  - Missing data: Ensure `self.future_symbol` is set correctly

---

## Running Your First Backtest

### Step 1: Configure Backtest Parameters
Before running, set the backtest parameters:

1. Click the **"Backtest"** button (top-right of IDE)
2. You'll see "New Backtest" dialog

### Step 2: Backtest Configuration
The algorithm comes pre-configured with:
- **Start Date:** January 1, 2020
- **End Date:** December 31, 2025
- **Starting Capital:** $50,000 USD
- **Data Resolution:** 1-minute bars (consolidated to 5m)
- **Instrument:** ES (E-mini S&P 500) Futures

**To modify any of these:**

Edit these lines in the `initialize()` method:
```python
self.SetStartDate(2020, 1, 1)      # Change start date
self.SetEndDate(2025, 12, 31)      # Change end date
self.SetCash(50000)                 # Change starting capital
```

### Step 3: Run the Backtest
1. Click the **"Backtest"** button
2. QuantConnect will run the backtest (may take 30-60 seconds)
3. You'll see real-time logs in the IDE output window
4. Once complete, results appear in the **Results** tab

### Step 4: First Backtest Run
- **Expected result:** The algorithm should execute trades during NY session hours
- **Common first result:** 5-15 trades depending on date range and market conditions
- **If no trades:** Check your filters and make sure the date range has volatile markets

---

## Interpreting Results

### Main Backtest Results Panel
After a backtest completes, you'll see:

#### Key Metrics Explained

| Metric | What It Means | Target |
|--------|--------------|--------|
| **Total Return** | Overall profit/loss as % | Positive |
| **Annual Return** | Annualized return % | 20%+ is excellent |
| **Sharpe Ratio** | Risk-adjusted returns | >1.0 is good, >2.0 is excellent |
| **Max Drawdown** | Largest peak-to-trough loss | <15% is excellent |
| **Win Rate** | % of profitable trades | >60% is good |
| **Profit Factor** | Gross profit / Gross loss | >1.5 is good, >2.0 is excellent |
| **Sortino Ratio** | Like Sharpe but ignores downside | >1.5 is good |

#### Understanding the Charts

**Equity Curve (Top Chart)**
- Shows account growth/decline over time
- Steep upslope = good, flat = no edge, downslope = losses
- Smooth curve = consistent performance
- Jagged curve = inconsistent or risky

**Drawdown (Bottom Chart)**
- Red areas show losses from peak
- Deeper red = worse drawdown
- Look for how quickly it recovers

### Trade Log
Click **"Trade Log"** tab to see:
- Each trade's entry and exit prices
- Profit/loss per trade
- Entry reason (which rule triggered)
- Trade duration
- Risk-reward ratio

### How to Find Problems

**No Trades Executed:**
- Your filters may be too strict
- Session time filters eliminating all opportunities
- Date range has low volatility

**High Drawdown:**
- Stops may be too tight (getting stopped out on noise)
- Position sizing too aggressive
- Market conditions unsuitable for strategy

**Low Win Rate (<40%):**
- Entry signals may have low quality
- Stop placement may be poor
- Consider adding more confluence filters

---

## Modifying Parameters

### Basic Parameters to Adjust

All parameters are defined at the top of the class:

```python
# In the TempoTradingBrain class
EMA_FAST = 9          # Primary structure (shorter = more responsive)
EMA_MID = 21          # Secondary structure
EMA_SLOW = 50         # Tertiary structure
EMA_BASE = 200        # Long-term bias

ATR_STOP_MULTIPLE = 1.5  # Stop distance (increase for wider stops)
RISK_PER_TRADE = 0.01    # 1% risk (0.01 = 1%, 0.02 = 2%)

PRIMARY_SESSION_START = 9.5   # 9:30 ET (market open)
PRIMARY_SESSION_END = 11.5    # 11:30 ET
```

### Parameter Tuning Strategy

**For Fewer Trades (Tighter Filtering):**
- Increase EMA periods (more conservative)
- Increase ADX_MINIMUM threshold
- Reduce RISK_PER_TRADE

**For More Trades:**
- Decrease EMA periods (more sensitive)
- Decrease ADX_MINIMUM
- Increase secondary session times

**For Wider Stops (Less Stop Outs):**
- Increase ATR_STOP_MULTIPLE from 1.5 to 2.0

**For Tighter Stops (More Risk Control):**
- Decrease ATR_STOP_MULTIPLE from 1.5 to 1.0

### Modifying Session Times
Change the session hours:
```python
PRIMARY_SESSION_START = 9.5      # 9:30 ET
PRIMARY_SESSION_END = 11.5       # 11:30 ET
SECONDARY_SESSION_START = 13.5   # 1:30 PM ET (after lunch)
SECONDARY_SESSION_END = 15.0     # 3:00 PM ET (market close)
```

**Note on Time Zones:**
- QuantConnect uses UTC by default
- ES futures trade in ET (Eastern Time)
- Add 5 hours to convert (9:30 ET = 14:30 UTC)
- The algorithm handles this conversion internally

### After Modifying Parameters
1. Save the file (Ctrl+S)
2. Run a new backtest
3. Compare results to previous backtest
4. Iterate until satisfied with performance

---

## Troubleshooting

### Common Issues and Solutions

#### Problem: "No data for this security"
**Cause:** ES futures contract not recognized
**Solution:**
- Ensure you're using: `Futures.Indices.SP500EMini`
- Check that date range includes valid trading days

#### Problem: Algorithm runs but no trades
**Cause:** Filters too strict or market conditions not matching
**Solution:**
- Reduce confluence requirements (test with just IFVG)
- Expand session times temporarily to see if that's the issue
- Check trade log to see why entries are being rejected

#### Problem: "Runtime error" or "IndexError"
**Cause:** Insufficient data in rolling windows
**Solution:**
- Algorithm starts trading after 50 bars of data
- This is normal for first few bars, gets faster afterward
- Check your start date - make sure it's not right after market opens

#### Problem: Very high drawdowns (>30%)
**Cause:** Position sizing too aggressive or stops too tight
**Solution:**
- Reduce RISK_PER_TRADE from 0.01 to 0.005 (0.5%)
- Increase ATR_STOP_MULTIPLE from 1.5 to 2.0
- Add session filters to skip choppy periods

#### Problem: "Out of memory" or "Timeout"
**Cause:** Backtest is too long or data resolution too high
**Solution:**
- Reduce date range to test smaller periods first
- Use longer consolidation periods (10m instead of 5m)
- Test with fewer assets

### Getting Help

If you encounter issues:

1. **Check QuantConnect Logs:**
   - Click "Open Logs" in the IDE
   - Look for error messages at the end

2. **Test with Shorter Date Range:**
   - Start with 1 month of backtest
   - Verify it works before expanding

3. **Use QuantConnect Community:**
   - Forum: https://www.quantconnect.com/forum
   - Discord: QuantConnect community server
   - Email: support@quantconnect.com (for paid support)

---

## Advanced Features

### Examining Individual Trades
1. In Results tab, click "Trade Log"
2. View detailed info for each trade:
   - Entry price and time
   - Exit price and time
   - Profit/loss
   - Reason for entry/exit

### Comparing Backtests
1. Run multiple backtests with different parameters
2. Click "Backtest History" to see all runs
3. Compare key metrics side-by-side
4. Save the best-performing version

### Optimizing Parameters
QuantConnect offers parameter optimization:
1. Click "Optimize" (in the Backtest menu)
2. Define parameter ranges:
   ```
   EMA_FAST: 5 to 15 (step 2)
   ATR_STOP_MULTIPLE: 1.0 to 2.5 (step 0.25)
   ```
3. Select optimization metric (Sharpe Ratio, Total Return, etc.)
4. Let it run (may take hours depending on range)
5. Review results for optimal parameters

---

## Performance Expectations

### Realistic Backtesting Results for ES Futures

**What to Expect:**
- **Win Rate:** 45-65% is typical for futures
- **Annual Return:** 15-50% on $50k account is good
- **Max Drawdown:** 10-20% is normal
- **Sharpe Ratio:** 1.0-2.0 is healthy

**Red Flags:**
- Win rate > 80% = likely overfitting (won't work live)
- Annual return > 100% = likely overfitting
- Max drawdown > 50% = too risky
- Sharpe ratio < 0.5 = inconsistent returns

### Reality Check
- Backtests are optimistic (no slippage, perfect fills)
- Live trading will be 10-20% worse due to:
  - Slippage (didn't get exact entry/exit price)
  - Commissions ($2-5 per round-trip trade)
  - Spread costs
  - Partial fills
  - Unexpected market gaps

---

## Next Steps After Backtesting

### If Results Look Good (Sharpe > 1, Max DD < 20%)
1. Paper trade (practice trade with no real money) for 1-2 weeks
2. Monitor for any discrepancies vs backtest
3. If paper trading confirms, start with 1 micro contract
4. Scale up gradually only after 10+ winning weeks

### If Results Look Poor
1. Try different parameter combinations
2. Add more confluence filters
3. Review entry reasons in trade log
4. Consider market regime (backtests include all market types)

### Suggested Parameter Tweaks to Test
Test these variations (one at a time):
```
Test 1: Reduce risk to 0.5% per trade (0.005)
Test 2: Add 4H timeframe confluence requirement
Test 3: Increase session to 9:30-14:00 (full primary session)
Test 4: Reduce ATR multiplier to 1.2x
Test 5: Add ADX > 25 minimum (stronger trends only)
```

---

## Important Disclaimers

### Backtesting Limitations
- Backtests assume perfect execution (unrealistic)
- Backtests assume you're always awake and can trade (not realistic)
- Past performance doesn't guarantee future results
- Market regimes change; what worked in 2020-2022 may not work in 2024-2025

### Risk Warning
- Futures trading involves leverage and significant risk of loss
- Start small and only risk money you can afford to lose
- A losing streak can wipe out a trading account
- Paper trade first before using real money
- Consider learning from losses before scaling

### Paper Trading First
Before risking real capital:
1. Paper trade for at least 2 weeks
2. Verify the strategy matches backtest performance
3. Check that you can execute the trades manually
4. Ensure you understand the risks

---

## Useful Resources

### Learning QuantConnect
- [QuantConnect Documentation](https://www.quantconnect.com/docs)
- [Algorithm Tutorial](https://www.quantconnect.com/learning)
- [YouTube Channel](https://www.youtube.com/c/QuantConnect)

### Trading Education
- Understanding IFVG: Search "Internal Fair Value Gap trading"
- Futures trading basics: CBOT or CME education sites
- Risk management: "Position Sizing" and "Kelly Criterion"

### Community Resources
- [QuantConnect Forum](https://www.quantconnect.com/forum)
- [GitHub Examples](https://github.com/QuantConnect/Lean/tree/master/Algorithm.Examples)
- [Discord Community](https://discordapp.com) - Search QuantConnect

---

## Algorithm Files Reference

### Main Files
- `qc_tempo_brain.py` - Main algorithm (this file)
- `tempo_rules.json` - Extracted Tempo rules documentation
- `tempo_brain_v1.py` - Reference implementation (non-QC version)

### Important Code Sections

**Entry Logic:** Lines 250-350 (see `_check_entry_signals`)
**Exit Logic:** Lines 352-420 (see `_check_exit_signals`)
**Indicators:** Lines 280-340 (see `_calculate_indicators_5m`)
**Session Filter:** Lines 475-500 (see `_is_in_valid_session`)

---

## FAQ

**Q: Can I use this algorithm for live trading?**
A: After successful backtesting and paper trading, yes. Start with the smallest position size possible.

**Q: How do I add alerts for trades?**
A: Add `self.NotifyUser()` or integrate with Slack/Discord webhooks (QuantConnect docs).

**Q: Can I trade other instruments?**
A: Yes, but adjust parameters. NQ (Nasdaq) and MES (Micro ES) would need different settings.

**Q: What time zone are the parameters in?**
A: Eastern Time (ET). Adjust the `PRIMARY_SESSION_START/END` times as needed.

**Q: How often should I rebalance parameters?**
A: Monthly or quarterly. Run backtests on recent data and adjust if performance degrades.

**Q: Is QuantConnect free?**
A: Yes! Free tier includes unlimited backtests, 15+ years of data, and community support.

---

**Last Updated:** February 2024
**Algorithm Version:** 1.0
**QuantConnect API:** Latest (LEAN)
