---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/WickFade_5M_Research_Findings.docx
related:
  - wiki/concepts/wick-fade.md
  - wiki/summaries/wickfade-complete.md
  - wiki/maps/strategies-moc.md
tags: [summary, wick-fade, 5m, tick-level]
---

# WickFade 5M Research Findings — Summary

> YES, profitable. ~$3,500/day per contract at 0.5pt spread over 258 days. Not a single month negative. Five simulation bugs fixed during research.

## Key findings

13-month WickFade backtest on 5-minute NQ bars using Databento tick data (Feb 2025–Feb 2026). The strategy fades breakouts of the previous 5M bar's high/low, with flip on stop-out. Fixed 15pt target.

Five critical simulation bugs were identified and fixed (raw sim went from -6 R/day to profitable). The flip timing artifact is significant: 52.6% of flip targets hit within 1 second of entry, representing one-third of total PnL. Removing sub-1s artifacts reduces PnL by 34% but the strategy remains strongly profitable at 177 pts/day ($3,545/contract). Calmar drops from a suspicious 1,076 to a believable 446–499.

The edge persists every single month including summer lull. Fades alone produce 135 pts/day; realistic flips add 42 pts/day.

## Status / caveats

This is an earlier research session — the comprehensive version is [[wiki/summaries/wickfade-complete|WickFade Complete]]. The flip timing artifact analysis is unique to this doc and not duplicated elsewhere.
