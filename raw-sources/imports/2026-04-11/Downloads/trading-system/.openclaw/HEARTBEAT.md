# Daily Trading Schedule (Eastern Time)

## 5:30 AM - Pre-Market Context Build
- Read overnight price action
- Check economic calendar for high-impact events
- Determine VIX regime (low/normal/high/extreme)
- Identify key levels: London H/L, Asia H/L, prev day H/L, 50% level
- Scan 4H and 1H charts for draw on liquidity targets
- Identify unmitigated FVGs and order blocks on 15m+
- Write context/daily.json with all session data
- Send context summary to Harrison via Telegram

## 9:25 AM - Session Open Prep
- Re-read KNOWLEDGE_BASE.md and active brain files
- Confirm draw on liquidity targets still valid
- Check for any last-minute news additions
- Verify no bank holidays or special sessions
- Send "READY" status to Telegram with key levels

## Every 5 min (9:35 AM - 11:00 AM) - Primary Kill Zone Monitoring
- Check for liquidity sweeps at session levels
- If sweep detected: scan for IFVG formation on 30s/1m/2m/3m
- Evaluate quality score (need >= 3 confluences)
- Check SMT divergence (NQ vs ES)
- If valid setup: send signal to Telegram with full context
- Log all evaluations (taken AND skipped) to execution/trades.log

## 11:00 AM - 12:00 PM - Extended Session
- Reduce to every 15 min checks
- Only A+ setups (quality >= 4)
- Monitor any open positions

## 12:00 PM - Midday Review
- Summarize morning performance
- Calculate current P&L and R-multiple
- Check if 3R daily target hit (if yes, lock accounts)
- Log midday snapshot to logs/

## 3:30 PM - End of Day Report
- Full performance summary with per-trade breakdown
- Strategy attribution (which brain triggered each trade)
- Context analysis: what worked, what didn't, why
- Compare actual vs expected (strategy rules compliance)
- Update KNOWLEDGE_BASE.md if new learnings discovered
- Log EOD report to backtest/results/YYYY-MM-DD.json
- Send EOD report to Harrison via Telegram

## Hourly (during market hours) - Git Sync Check
- Check if ~/trading-system has new commits on remote
- If yes: git pull and reload knowledge base
- Log sync status

## Weekly (Sunday 8 PM) - Weekly Review
- Aggregate weekly performance
- Win rate, average R, total P&L
- Strategy breakdown (which brains performed best)
- Identify patterns in missed setups
- Send weekly report to Telegram
