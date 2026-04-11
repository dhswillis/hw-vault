---
name: ceo-briefing
description: Generate Willis Holdings CEO briefings. Collect status from all divisions, run decision engine, produce morning and EOD briefings for Harrison via Telegram.
emoji: 📊
author: willis-holdings
version: 1.0.0
requires:
  bins:
    - python3
tags:
  - ceo
  - briefing
  - divisions
  - conglomerate
---

# CEO Briefing System

Generates daily briefings for Harrison by collecting status from all Willis Holdings divisions.

## Division Status Collection

### Active Divisions (collect every cycle)

**1. Trading (TEMPO)**
Check for:
- Latest backtest results in `~/trading_operator/results/`
- VPS batch status: `ssh root@172.93.181.209 "tail -5 ~/batch3_costmodel.log"`
- NautilusTrader progress: `ps aux | grep nautilus`
- Any new mining winners

**2. Finance (CFO)**
Check for:
- Next tax deadline (current: S-Corp returns 2026-03-15)
- Quarterly estimate status
- FP&A projections: Bull $730K / Base $504K / Bear $201K annual

**3. IT Infrastructure**
Check for:
- VPS health: `ssh -o ConnectTimeout=5 root@172.93.181.209 "uptime && df -h /"`
- GitHub PAT expiry: 2026-05-15 (check days remaining)
- Disk usage alerts (>85% = flag)
- Monthly infra cost (~$70/mo: VPS $20 + APIs $50)

**4. HR & Bots**
Check for:
- Active agent count (currently 5)
- Monthly API costs (~$55/mo)
- Agent error rates
- Model routing optimization opportunities

### Planned Divisions (status = "planned", no data collection yet)
5. Arbitrage — Polymarket/Kalshi
6. Content — Trading insights pipeline
7. Research — Daily intelligence
8. BizDev — Incubator
9. Fundraising — Activates after 6mo track record

## Briefing Format

### Morning Briefing
```
WILLIS HOLDINGS — MORNING BRIEFING
{date}

P&L: Yesterday {amount} | MTD {amount} | YTD {amount}

DIVISIONS:
{emoji} Trading: {summary}
{emoji} Finance: {summary}
{emoji} IT: {summary}
{emoji} HR: {summary}

OPPORTUNITIES (by conviction):
- [9] {description}
- [7] {description}

RISKS (by severity):
- [8] {description}
- [5] {description}

AUTONOMOUS DECISIONS MADE:
- {decision}

FLAGGED FOR AWARENESS:
- {item}

NEEDS YOUR APPROVAL:
- {item}

TODAY'S PLAN:
- Trading: {plan}
- IT: {plan}
```

### EOD Briefing
```
WILLIS HOLDINGS — EOD BRIEFING
{date}

TODAY'S RESULTS: {day_pnl} | MTD {mtd}

VARIANCE VS MORNING PLAN:
{emoji} Trading: planned X, actual Y
{emoji} Finance: planned X, actual Y

TOMORROW'S PRIORITIES:
1. {priority}
2. {priority}

NEEDS HUMAN ATTENTION:
- {item}
```

## Decision Thresholds
- < $500: Auto-execute + log
- $500-$5K: Execute + flag in briefing
- > $5K: Propose to Harrison, wait for approval
- New business lines: Always propose

## Running the Briefing

### Via Python (full conglomerate system)
```bash
cd ~/trading_operator && python3 -m conglomerate.ceo.orchestrator morning --dry-run
```

### Quick status check
```bash
cd ~/trading_operator && python3 -m conglomerate.ceo.orchestrator test
```
