---
date: 2026-04-14
status: decided
deciders: [harrison]
project: tempo-trading-system
tags: [decision, trading, tempo, lumi, promotion]
links:
  - wiki/summaries/tempo-portfolio-v26.md
  - wiki/summaries/ifvg-composite-audit-20260329.md
  - wiki/concepts/ifvg.md
---

# Lumi: EXCLUDED → LIVE (V15 V2)

## Context

Lumi (the ES-momentum-aligned overlay) had been held in `EXCLUDED` state pending walk-forward evidence. W16 V15 V2 run produced a combined portfolio showing **+19.8 pts/day** with **Calmar 23.1** when Lumi was included as an overlay on the existing IFVG stack. The overlay lifts per-trade edge by ~+7.4% WR when the ES 1h momentum agrees with the NQ signal (alignment rate 63.9% of triggers), and walk-forward folds held positive across the tested window.

## Options considered

### Option 1: Promote Lumi to LIVE as-is (overlay on existing V26 live strategy)

**Pros:**
- +19.8 pts/day combined throughput, Calmar 23.1 — top config seen across V15 V2 sweep.
- Walk-forward validated; alignment mechanic is interpretable (ES momentum agreement), not a curve-fit parameter.
- `LumiStrategyV3.cs` (902 lines) already built and ported to NinjaTrader syntax.

**Cons:**
- Limit-fill vs market-fill sensitivity: backtest +0.45R/trade **collapses to −0.065R** if all entries are forced to market fills. Live execution quality now materially determines whether the edge survives.
- Adds a second live strategy to monitor on the same NQ chart — increases collision surface (see V5 double-counting precedent).

### Option 2: Hold Lumi in PAPER for another 4–6 weeks before promotion

**Pros:**
- More live-tape evidence on fill quality before risking capital.
- Cleaner separation: promote only after fill-quality telemetry matches backtest assumptions.

**Cons:**
- Opportunity cost — if the fill assumption holds, 4–6 weeks of +19.8 pts/day foregone.
- Paper trading doesn't resolve the limit-vs-market question; only live limit fills with real queue priority do.

## Decision

Promote Lumi from `EXCLUDED` to `LIVE` as a V15 V2 overlay on the V26 stack, with limit-entry orders and a live fill-quality monitor that kills the overlay automatically if observed per-trade R drops below +0.15R over any 20-trade rolling window.

## Why

The alignment mechanic (ES 1h momentum) is not a fit parameter — it's a regime filter with a priori economic justification (index-component correlation tightens directional probability). The walk-forward held. The one real risk is fill quality, and that is the correct thing to monitor live, not model in paper — paper fills don't reveal queue priority behavior. The autokill gate (<+0.15R over 20 trades) makes the downside bounded: in the worst case this is a ~20-trade experiment, not an open-ended commitment.

## Consequences

**Enables:**
- Combined live throughput target steps from V26 baseline (~+7.55 R/day, 60% WR from [[wiki/summaries/ifvg-composite-audit-20260329]]) toward the +19.8 pts/day V15 V2 target.
- Establishes the overlay pattern — future regime filters (e.g. VIX-gated, time-of-day-gated) can follow the same promotion protocol.

**Closes off / trade-offs:**
- Second live strategy on NQ chart means any future NT chart-stats work has to account for two consumers (watch for collision bugs analogous to the V5 VWAP double-count).
- Committing to limit-entry means accepting missed-fill variance. Market-fallback is explicitly rejected; the edge doesn't survive it.

## Review trigger

- **Hard trigger:** autokill if rolling 20-trade R < +0.15. Revisit decision immediately if triggered.
- **Soft trigger:** after 60 live trading days OR 200 Lumi-triggered trades, whichever comes first. Compare live R/trade and WR against V15 V2 backtest; if live underperforms backtest by >1 standard deviation, demote to PAPER and reopen the fill-quality investigation.
