---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/Willis_Holdings_Strategy_Audit.docx
related:
  - wiki/entities/ninjatrader-v5.md
  - wiki/maps/strategies-moc.md
  - wiki/maps/audit-history-moc.md
  - wiki/concepts/invalidation-rules.md
tags: [summary, audit, strategy-status, march-2026]
---

# Willis Holdings Strategy Audit — Summary

> March 2026 status report: 4H fakeout + rejection candle is the best single strategy (50% of portfolio PnL). ALL fades of structural levels are losers. V7/V8 mining results invalidated by look-ahead.

## Key findings

This is a point-in-time snapshot (March 9, 2026) of all strategies in the Willis Holdings system. Two strategies are live on NinjaTrader Sim101 (MNQ). The strategy that showed the strongest edge is the 4H Fakeout + 1M Rejection Candle — entry when price fakes out 10+ pts beyond 4H candle open, exit at end of 4H candle. Low win rate but massive avgR per winner, contributing 50% of total portfolio PnL.

Key finding: ALL fades of structural levels are losers — every single one tested. Only fakeouts on higher timeframes (4H, Daily) and the Sweep-Fail portfolio (V17SF, 10-leg with Leppyrd filters) showed real edge. V7/V8 mining results are explicitly invalidated due to multi-timeframe alignment using forming bar closes (look-ahead bias).

## Status / caveats

Partially stale — some strategies referenced here have been further validated or killed since March 2026. Cross-reference [[wiki/maps/strategies-moc|Strategies MOC]] for current status. The V7/V8 invalidation warning aligns with [[wiki/concepts/v10i-look-ahead-bug]].
