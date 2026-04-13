---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sf-portfolio/PORTFOLIO_STATE_20260320.md
related:
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/summaries/nq-sf-engulfing-strategy.md
  - wiki/maps/strategies-moc.md
tags: [summary, sf-portfolio, portfolio-state, march-2026]
---

# SF Portfolio State (March 2026) — Summary

> Best portfolio: +2.26 R/Day over 260 days, zero look-ahead, 12-leg NQ+ES system. 87% winning weeks, 100% winning months.

## Key findings

Point-in-time portfolio snapshot (March 20, 2026). 12 signal legs across NQ and ES: sweep-fail (4 legs across sessions), 5M momentum continuation (3 entries/day), 1M breakout, stop market entry, VWAP fade, London close reversal, 4H offset. Total ~2,700 trades.

The SF_1H_LON_RUN (runner) leg dominates at +1.053 R/Day — a single fat-tailed leg producing 47% of total portfolio PnL from only 98 trades (16% WR). Reduced 8-leg portfolio (dropping lowest-value legs) produces +1.94 R/Day with a cleaner equity curve. 8 signal concepts across the full portfolio.

## Status / caveats

This is the pre-tick-level-audit state. The [[wiki/summaries/sf-portfolio-cluster|SF Portfolio Cluster]] has the updated assessment after tick-level verification. The RUNNER leg's fat-tailed nature means short evaluation windows can show large variance — the 22-day sanity check showed -0.428 R/Day vs published +1.384 (variance, not a bug).
