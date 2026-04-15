---
created: 2026-04-14
updated: 2026-04-14
status: active
owner: Harrison
tags: [project, trading, tempo]
links:
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/tempo-methodology.md
  - wiki/maps/strategies-moc.md
  - wiki/maps/tempo-moc.md
---

# Tempo Trading

Active trading project using the [[tempo-methodology|Tempo methodology]] on NQ futures via [[ninjatrader-v5|NinjaTrader]] and [[quantconnect|QuantConnect]].

## Current state

- **Validated portfolio (backtest, not yet live):** [[ifvg|IFVG]] + [[lumi-strategy-spec|Lumi]] (V15 rewrite to spec, +19.8 pts/day combined, Calmar 23.1 on tick-level backtest)
- **In development:** [[sf-portfolio-cluster|SF Portfolio]] (tick-level validated, NinjaTrader bar-gating fix applied)
- **Unproven:** [[wick-fade|Wick Fade]] (backtest only, never live/sim, 68% of edge from debunked flip leg)
- **Dead:** [[bos-fvg|BOS FVG]] (killed by [[bar-sim-trailing-bug]]), see [[maps/bos-fvg-saga-moc|full saga]]
- **System:** [[tempo-trading-system|Tempo Trading System]] with [[bayesian-belief-engine|Bayesian belief engine]] and [[tempo-context-engine-spec|Context Engine]]

## Open actions

- [ ] Pre-approve scheduled tasks in Cowork sidebar
- [ ] Run Claude Code deep ingest on remaining raw sources
- [ ] Set up `/trade-review` daily habit

## Trade log

Daily trade logs are stored in `work/projects/tempo-trading/trade-log/YYYY-MM-DD.md`. Use `/trade-review` to create them.

## Key references

- [[invalidation-rules|Invalidation Rules]] — the "do not forget" ledger of every bug and dead end
- [[maps/audit-history-moc|Audit History]] — chronological view of all audits
- [[maps/strategies-moc|Strategies MOC]] — full strategy landscape
- [[maps/tempo-moc|Tempo MOC]] — Tempo-specific deep dive
