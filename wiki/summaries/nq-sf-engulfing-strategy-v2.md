---
created: 2026-04-12
updated: 2026-04-21
type: summary
sources:
  - raw-sources/trading/sf-portfolio/NQ_SF_Engulfing_Strategy_V2.docx
related:
  - wiki/summaries/sf-portfolio-cluster
tags: [summary, engulfing, sweep-fail, leppyrd-ict]
---

# NQ SF Engulfing Strategy V2 — Summary

> V2 refines the 9-strategy portfolio with explicit Leppyrd ICT framework integration and engagement rules.

## Key findings

**System overview:** 9-strategy portfolio combining Sweep & Fail Engulfing (SF Engulfing) and Leppyrd ICT models spanning 24-hour cycle. Backtested Feb 12, 2025—Feb 11, 2026 (260 days): ~987R/year, +16,798 total points, 0 losing weeks, 800 trades, session-routed independent entries.

**Why SF works:** Sweep & Fail (wick beyond level → close back through) is "turtle soup" reversal triggering stops. Engulfing body filter (current body engulfs prior opposite) transforms 40% raw win rate to 75-96%. Leppyrd daily bias (PDH/PDL close position relative to previous day) creates directional filter for Asia/London sessions. HTF FVG manipulation (price drawn into 15M/1H FVG against bias) followed by 5M breaker block (opposing candle + breakout) = entry trigger. H4 swing confirmation (sweep & close inside) = highest conviction.

**Temporal independence:** 9 strategies span full 24h: Asia Early (6-10 PM CT), Asia Late (10 PM-2 AM CT), London (2-4 AM CT), Pre-Market (6-8:30 AM CT), NY Open/Lunch/PM (8:30 AM-3 PM CT). Different time windows + different concepts (SF vs Leppyrd) = natural portfolio diversification. Losing weeks in one session offset by wins in others.

**Key stats:** LON-SF 128R/yr, ALATE-LEP 252R/yr (highest contributor), NY1-H4 186R/yr, combined 987R/yr. Win rates 46-83% by strategy. R:R ratios 1.0-2.0. Transaction costs 0.75pt (0.25 slippage + 0.50 commission) included.

## Status / caveats

Document is summary/refresh of the primary NQ Portfolio system. Emphasis on Leppyrd framework integration and why temporal diversification prevents consecutive losing weeks. No new components vs primary system doc; serves as engagement/architecture reference. Highest-concentration risk: ALATE-LEP 252R = 25% of portfolio. Lowest: NLATE-SF 50R. London strategies post-emergency-fix status unclear; cross-reference with [[wiki/summaries/tempo-ifvg-audit-report]].

## Supersedes

- [[nq-sf-engulfing-strategy]] — v1 of this doc. V2 adds the 9-strategy portfolio structure, Leppyrd ICT integration, and the 987R/year consolidated result. Retain v1 for historical provenance; cite v2 for current claims.
