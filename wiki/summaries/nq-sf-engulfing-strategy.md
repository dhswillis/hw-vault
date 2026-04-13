---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/NQ_SF_Engulfing_Strategy.docx
related:
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/summaries/nq-sf-engulfing-strategy-v2.md
  - wiki/maps/strategies-moc.md
tags: [summary, sf-portfolio, sweep-fail, engulfing]
---

# NQ SF Engulfing Strategy — Summary

> Sweep & Fail with Engulfing confirmation achieves zero losing weeks across 260 trading days when deployed across five independent time windows.

## Key findings

This documents the complete NQ futures trading system built around the Sweep & Fail + Engulfing confirmation pattern, discovered through exhaustive tick-level backtesting on 260 days of Databento data. The system trades five distinct time windows spanning London open through the second hour of US regular trading, each operating independently with its own entry criteria, target, and stop.

The Sweep & Fail is an ICT concept (liquidity sweep reversal): price breaks beyond a prior swing to trigger stop clusters, fills large orders, then reverses. The Engulfing filter is what transforms a mediocre ~40% WR concept into an exceptional 75-96% WR system. Not every sweep is tradeable — the engulfing candle confirmation is the critical quality gate.

## Status / caveats

V1 of the SF portfolio documentation. See [[wiki/summaries/nq-sf-engulfing-strategy-v2|V2]] and [[wiki/summaries/sf-portfolio-cluster|SF Portfolio Cluster]] for the evolved version with Leppyrd ICT framework. The "zero losing weeks" claim should be checked against the [[wiki/concepts/bar-sim-trailing-bug|bar-sim trailing bug]] if any configs use trailing stops.
