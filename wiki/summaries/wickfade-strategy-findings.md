---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Findings.docx
related:
  - wiki/concepts/wick-fade.md
  - wiki/summaries/wickfade-complete.md
  - wiki/maps/strategies-moc.md
tags: [summary, wick-fade, optimization]
---

# WickFade Strategy Findings — Summary

> Best validated WickFade config produces $7,452/day vs V13's $1,451/day, but requires trail stops on both legs and multi-position architecture.

## Key findings

The WickFade strategy fades breakouts of the previous 1-minute bar's high/low on NQ, flipping on stop-out. Backtested across 312 days of Databento tick data. The meta-test progression shows trail stops on BOTH legs (not just flip) was the single biggest improvement, raising WR from 41.6% to 53.3%.

Confluence filter analysis on 10,919 setups found that NO confluence filter is worth adding — the edge is structural. EMA21 direction, VWAP position, and 5M/15M sweeps are all dead. Walk-forward validation shows 100% positive OOS months on both rolling 3-month and 6-month train/test splits, indicating LOW overfitting risk.

Dynamic stop tick backtest revealed that fixed S=2 T=10 is deeply negative on tick data due to slippage eating 12.5% of the 2pt stop. Wider stops dramatically reduce slippage impact.

## Status / caveats

This is an earlier version of the [[wiki/concepts/wick-fade|WickFade]] research. The comprehensive findings are in [[wiki/summaries/wickfade-complete|WickFade Complete]]. Read this for the optimization journey; read that for final production parameters.
