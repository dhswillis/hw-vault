> Maintained by Claude per `CLAUDE.md`.

# Index

Catalog of every wiki page, one line each. Updated on every `ingest` and any time a wiki page is created or renamed.

## Concepts

- [[wiki/concepts/bos-fvg]] — **DEAD (2026-04-10).** BOS_FVG is NOT validated. Bar-sim trailing inflated prior results by +0.29 R/trade; tick replay shows +0.001 avgR.
- [[wiki/concepts/bar-sim-trailing-bug]] — Structural reason trailing stops can't be backtested on 1-minute OHLC. Caused the BOS_FVG illusion.
- [[wiki/concepts/mtf-alignment]] — Multi-timeframe alignment. Prior WR claims contaminated by V10i look-ahead; re-test after tick-level rework.
- [[wiki/concepts/be-trail-mechanism]] — Break-even + trailing stop exit. The mechanism whose apparent edge was the [[bar-sim-trailing-bug]] illusion on BOS_FVG.
- [[wiki/concepts/v10i-look-ahead-bug]] — Critical alignment look-ahead that contaminated all V1-era mining research. Fixed in V2 (but V2 still has the bar-sim bug).
- [[wiki/concepts/volume-profile]] — POC, VA, HVN/LVN + footprint analysis. Layer 2/3 of the Context Engine.
- [[wiki/concepts/session-type-taxonomy]] — TREND / RANGE / EXPANSION / COMPRESSION / NEWS-SHOCK classification used by the Context Engine.
- [[wiki/concepts/bayesian-belief-engine]] — Multinomial naive Bayes classifier with checkpoint-driven posterior updates, heart of the Context Engine.
- [[wiki/concepts/vwap-double-counting-bug]] — NinjaTrader `Calculate.OnEachTick` bug that broke live/backtest parity in the V5 SF portfolio.

## Entities

- [[wiki/entities/tempo-trading-system]] — The overall NQ futures trading system at `~/Documents/trading-system/`. Research + backtest + planned Context Engine.
- [[wiki/entities/quantconnect]] — Backtest/execution platform. Project `28083727`. Runs the Tempo batch variants.
- [[wiki/entities/databento]] — Tick-level data vendor. Source for volume profile + footprint. Key saved on VPS, not yet integrated live.
- [[wiki/entities/ninjatrader-v5]] — The V5 SF portfolio (NQ VWPM, ES VWPM, NQ LF1170). Live-execution stack with 7 open bugs.

## Summaries

- [[wiki/summaries/bos-fvg-failure-consolidated]] — **2026-04-10 failure report.** Paired bar-vs-tick replay: bar +0.359 avgR, tick +0.001 avgR. Invalidates every prior BOS_FVG "validated" claim.
- [[wiki/summaries/comprehensive-mining-report-v2]] — NQ Futures Mining V2: 34,323 trades, look-ahead fixed, slippage + min-risk floor added. BOS_FVG numbers now suspect due to [[bar-sim-trailing-bug]].
- [[wiki/summaries/strategy-mining-report-312d]] — Earlier V1 mining (312d, 8,358 trades). Headline 97.9% WR tiers are contaminated by V10i look-ahead. Now also suspect for bar-sim inflation.
- [[wiki/summaries/v5-strategy-bug-audit]] — 2026-03-23 audit of V5 NinjaTrader strategies. Seven bugs, Bug 1 (VWAP double-counting) is the primary cause of ES VWPM live/backtest divergence.
- [[wiki/summaries/tempo-context-engine-spec]] — Engineering spec v1.0 for the Tempo Context Engine. Session classifier, five feature layers, nine checkpoints, Bayesian belief updates, 5-phase build plan.
- [[wiki/summaries/tempo-quick-start-guide]] — How to start a Tempo session. `tempo` alias, file locations (Mac/VPS/GitHub/QC), credentials reference, memory document chain.

## Syntheses

- [[wiki/syntheses/mining-reports-v1-v2-reconciliation]] — Cross-source analysis: why V1 claimed 97.9% WR while V2 shows 63.5%. Three corrections documented (entry idx, risk floor, V10i alignment). **Note 2026-04-10**: even the V2 "honest" 63.5% WR is now invalidated for BOS_FVG — see [[bos-fvg-failure-consolidated]].

## Projects (work)

*(none yet)*

## Projects (personal)

*(none yet)*
