---
created: 2026-04-10
updated: 2026-04-10
type: entity
sources:
  - raw-sources/Tempo_Quick_Start_Guide.docx
  - raw-sources/Tempo_Context_Engine_Spec.docx
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
related:
  - wiki/summaries/tempo-quick-start-guide.md
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bayesian-belief-engine.md
  - wiki/entities/quantconnect.md
  - wiki/entities/databento.md
  - wiki/entities/ninjatrader-v5.md
tags: [trading, tempo, system, overview]
---

# Tempo Trading System

Tempo is the personal NQ futures trading system operated out of `~/Documents/trading-system/`. It started as a backtesting engine for parameter mining and is evolving into a context-driven trading intelligence system.

## What it does today

Runs 2000 strategy variants through [[quantconnect|QuantConnect]] overnight, collects results, identifies which parameter configs make money under which conditions.

- **Batch runner** on VPS `172.93.181.209`
- **QC project** `28083727`, `main.py` uploaded from `main_v4.py`
- **Two-stage batch runner** — Stage A: 3-month screen → Stage B: 12-month confirm
- **2000-variant generator** covering signals, R targets, BE modes, sessions
- **Auto-deploy pipeline** via GitHub (`dhswillis/trading-system`)

## What it will do (Context Engine)

The [[tempo-context-engine-spec|Context Engine]] is the planned intelligence layer. Before each session, it predicts the [[session-type-taxonomy|session type]] (trend/range/expansion/compression/news-shock) using price structure, [[volume-profile|volume profile]], order flow, and news context, then selects the best strategy configuration for that environment via a [[bayesian-belief-engine|Bayesian belief update]]. Intra-session checkpoints refine the prediction as evidence arrives.

## Core signals

[[bos-fvg|BOS_FVG]] is the best single signal by total R and walk-forward stability. [[comprehensive-mining-report-v2|V2 cleaned results]]: 63.5% WR, +15,368R, R/T +1.354, stable IS→OOS. The full portfolio of 7 signals generates +20,662R over 258 days with Calmar 425.

Two distinct books in the portfolio:
1. **FVG trend-following** (BOS_FVG + fvg_exit variants with [[be-trail-mechanism|BE+Trail]]) — high WR, captures runners
2. **Sweep mean-reversion** (wick/close/swing with 5.0R fixed targets) — low WR, binary payoffs

Average pairwise correlation 0.134 — genuinely diversifying.

## Historical blast radius

The V1-era mining research was contaminated by the [[v10i-look-ahead-bug|V10i look-ahead bug]]. Any configuration claiming > 90% WR from that era is suspect. The [[mining-reports-v1-v2-reconciliation|reconciliation synthesis]] tracks what survived cleanup.

## Infrastructure

- **Mac (primary workstation):** `~/Documents/trading-system/`, Claude Code is the primary tool
- **VPS:** `172.93.181.209`, runs batch jobs and (eventually) the Context Engine
- **GitHub:** `dhswillis/trading-system` — source of truth (see also the auto-memory note on `dhswillis` GitHub identity)
- **QuantConnect:** project `28083727`, runs the backtests
- **[[databento|Databento]]:** tick data vendor, key saved on VPS `/root/.env` but not yet integrated
- **OpenClaw / Telegram:** phone monitoring (bot configured, custom Tempo skill not finished)
- **[[ninjatrader-v5|NinjaTrader V5 strategies]]:** separate live-execution stack (SF portfolio) currently being debugged — see [[v5-strategy-bug-audit]]

## Memory system (document chain)

```
CLAUDE.md                        (auto-read by Claude Code on `tempo` session start)
  ↓
START_HERE.md                    (project state, infra, next steps)
  ↓
CLAUDE_BOOTSTRAP.md              (credentials, integrations, tech details)
CONTEXT.md                       (strategy mechanics, signals, batch results)
Tempo_Context_Engine_Spec.docx   (context engine eng spec)
```

**Golden rule:** if it's not in `START_HERE.md` or linked from it, it doesn't exist next session.

## Session startup

```bash
tempo      # alias: cd ~/Documents/trading-system && git pull && claude
```

See [[tempo-quick-start-guide]] for the full onboarding doc.

## Active initiatives (as of raw sources, Feb–Mar 2026)

1. **Batch 3** running on VPS (2000 variants with realistic cost modeling: slippage, fees, tick-through, BE offset)
2. **Context Engine Phase 1** — price-only day classifier (starts when Batch 3 finishes)
3. **V5 bug fixes** — [[v5-strategy-bug-audit|seven bugs]] identified in NinjaTrader V5 SF portfolio, Bug 1 (VWAP double-counting) is the priority
4. **OpenClaw tempo-trading skill** — unfinished
5. **Stage B 12-month confirmation** — top 50 Batch 3 winners, planned
