---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/Tempo_IBKR_Migration_Plan.docx
related:
  - wiki/entities/quantconnect
  - wiki/entities/tempo-trading-system
tags: [summary, migration, execution, ibkr, python]
---

# Tempo IBKR Migration Plan — Summary

> Migrate from NinjaTrader 8 C# to Python + ib_insync; eliminates platform-induced bugs (orphaned positions, callback races, phantom trades).

## Key findings

**Why migrate:** NT8's IsExitOnSessionCloseStrategy cancels working stops in unmanaged mode, leaving positions unprotected with net-zero on same instrument. Reaction scoring forward-scan algorithm mis-translates to NT8's backward barsAgo (critical scan-direction bug). OnExecutionUpdate callback chains create race conditions on stop/target submission. No native bracket order support (entry/stop/target separate orders = multiple failure points). Python direct sequential code eliminates all these.

**Backtest performance (Mar 2025—Feb 2026):** 6 components (C0-C5: 15S/30S/1M shorts/longs), 1,707 total trades, ~60% WR, +1,913R/year, 20.3R max drawdown, Calmar 93.7. Session-routed: C0 (NY_AM, 323 trades, 66.3% WR, 2,065R), C1 (NY_Pre, 196 trades, 56.1% WR, 940R), C2-C5 distributed across NY_Lun, PM, London, Asia.

**Architecture (5 modules):** ib_insync broker connection, IBKR real-time 5-sec bars aggregated to 15S/30S/1M/15M/1H, strategy_engine.py (direct port of composite_strategy.py, no translation), order_manager OCA bracket orders (entry+stop+target atomic), risk_manager per-component daily/weekly R limits, news blackout, circuit breakers.

**Order flow:** Bar close → strategy_engine.run_bar() (sweep/FVG/entry check) → EntrySignal if triggered → order_manager submits OCA bracket to IBKR → fill callback updates state → risk_manager logs R result. Sequential, no callbacks races. Stop/target auto-cancel when one fills.

**Why this fixes NT8 issues:** No IsExitOnSessionCloseStrategy interference. No barsAgo translation. No phantom close orders. No async callback chains. Same exact Python code running in both backtest and live (zero divergence). News blackout trivial to add.

## Status / caveats

Architecture proven in principle. Implementation requires: bar_aggregator (DST transitions), strategy_engine port (reuse composite_strategy.py logic), order_manager (handle rejections/cancellations), risk_manager (daily/weekly/streak limits), main runner (watchdog, reconnection). No blocker issues. Key assumption: IBKR fills reliable enough that stop placement timing not critical (vs NT8 callback timing). News calendar integration assumed available.
