---
name: trading-operator
description: Monitor and manage the NQ futures trading system. Check backtest status, view results, manage NautilusTrader, review mining data, and monitor the live execution pipeline.
emoji: 📈
author: willis-holdings
version: 1.0.0
requires:
  bins:
    - python3
tags:
  - trading
  - nautilus
  - backtest
  - nq-futures
---

# Trading Operator

Manages the NQ futures trading system — backtest monitoring, result analysis, and live pipeline management.

## Trading System Overview

### Two-Book Architecture
**Book 1: FVG Trend-Following (BE+Trail exits)**
- BOS_FVG: 11,348 trades, 63.5% WR, +15,368R, +1.35R/trade
- fvg_exit_fvg_stop: 3,472 trades, 57.9% WR, +2,890R
- fvg_exit_body_stop: 3,584 trades, 56.4% WR, +2,118R

**Book 2: Sweep Mean-Reversion (5R fixed targets)**
- sweep_wick_sweep: 2,016 trades, 45.9% WR, +674R
- sweep_close_through: 1,560 trades, 45.6% WR, +512R
- swing_failure: 1,840 trades, 42.4% WR, +298R

**Combined Portfolio:** 22,896 trades, +20,662R, 81.8% green days, 13/13 green months, Calmar 425

### Sniper Tiers
- 100% WR: BOS_FVG + body_confirms + co=0 (309 trades, ALL reach 1R)
- Sharpshooter: al>=5, co<=0 (1,624 trades, 76.3% WR)
- Gunner: al>=3 (6,911 trades, 69.4% WR)

## Common Tasks

### Check NautilusTrader backtest status
```bash
# Check if backtest is running
ps aux | grep -i nautilus | grep -v grep

# Check latest logs
tail -50 ~/trading_operator/nautilus-bt/backtest.log 2>/dev/null || echo "No log file"

# Check Python processes
ps aux | grep python | grep -v grep
```

### View mining results
```bash
# Result file
python3 -c "
import json
with open('$HOME/trading_operator/cross_mining_results.json') as f:
    data = json.load(f)
print(f'Total trades: {len(data)}')
signals = {}
for t in data:
    s = t.get('signal','unknown')
    signals[s] = signals.get(s,0) + 1
for s,c in sorted(signals.items(), key=lambda x:-x[1]):
    print(f'  {s}: {c}')
"
```

### Check VPS batch status
```bash
ssh -o ConnectTimeout=5 root@172.93.181.209 "tail -10 ~/batch3_costmodel.log" 2>/dev/null || echo "VPS unreachable"
```

### Monitor NautilusTrader data loading
```bash
# Check how many data files exist
ls ~/trading_operator/data2/GLBX-*/*.trades.dbn.zst 2>/dev/null | wc -l
```

## Live Execution Pipeline
```
NautilusTrader (local Mac) → CrossTrade REST API ($49/mo) → NinjaTrader → Replikanto → Prop Accounts
```

## Key Files
- Strategy: `~/trading_operator/nautilus-bt/strategies/bos_fvg_strategy.py`
- Backtest runner: `~/trading_operator/nautilus-bt/run_backtest.py`
- Mining results: `~/trading_operator/cross_mining_results.json`
- Excel report: `~/trading_operator/NQ_Mining_Results_V2.xlsx`

## BE+Trail Exit Mechanism
1. Move stop to breakeven at 1R profit (+0.5pt buffer)
2. Begin trailing from 2R with formula: stop = entry + (max_r - 1.0) × risk
3. 29.5% of BOS_FVG trades are 2R+ runners → these produce +17,461R
4. Other 70.5% combined for -2,094R
5. Net: the runners more than pay for the losers and breakevens
