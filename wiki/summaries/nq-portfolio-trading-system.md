---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/sf-portfolio/NQ_Portfolio_Trading_System.docx
related:
  - wiki/summaries/sf-portfolio-cluster
  - wiki/maps/strategies-moc
tags: [summary, portfolio, sweep-fail, engulfing, leppyrd, ict]
---

# NQ Portfolio Trading System — Summary

> 9 strategies spanning 24-hour cycle; ~987R/year, 0 losing weeks, 800 trades, 260 days.

## Key findings

**Portfolio architecture:** Sweep & Fail Engulfing (5 strategies: LON-SF, PRE-SF, NY1-SF, NY1.5-SF, NLATE-SF) + Leppyrd ICT (4 strategies: AEARLY-LB, ALATE-LEP, LON-FVG, NY1-H4). All 9 trade NQ independently; no max 1 trade rule across portfolio—multiple strategies can fire same day. Temporal diversification: LON 83% WR +128R, PRE 70% WR +57R, NY-AM sessions dominate, ALATE-LEP 252R/yr at 81% WR.

**Core concepts:** Sweep & Fail (price extends beyond prior candle, triggers stops, reverses through level) + Engulfing filter (current body engulfs prior opposite = 40% raw → 75-96% filtered). Leppyrd bias (PDH/PDL framework) filters early session trades; HTF FVG manipulation + 5M breaker entry; H4 swing point confirmation (sweep & close inside = highest conviction 82% WR).

**Session routing:** Asia Early (AEARLY-LB) 77R/yr, Asia Late (ALATE-LEP) 252R/yr, London (LON-SF, LON-FVG) 232R/yr, Pre-Market (PRE-SF) 57R/yr, NY (NY1-SF, NY1.5-SF, NY1-H4, NLATE-SF) 382R/yr. Combined: +16,798 points = 987R @ 0.75pt slippage. Zero losing weeks (0/53), 82% avg win rate.

## Status / caveats

Backtest period Feb 12, 2025—Feb 11, 2026 (260 days). No live forward testing. ALATE-LEP carries 252R (25% of portfolio); if underperforms live, exposure concentrated. London strategies (LON-FVG breakout) noted as "dead" post-bug-fix in audit report; verify results with corrected stop logic. H4 swing (NY1-H4) produces +5,571 pts (33% of total); high dependency on swing detection accuracy.
