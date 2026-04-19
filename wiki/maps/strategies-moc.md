---
created: 2026-04-11
updated: 2026-04-18
type: moc
tags: [moc, strategies, trading]
related:
  - wiki/maps/root.md
  - wiki/maps/tempo-moc.md
  - wiki/maps/bos-fvg-saga-moc.md
  - wiki/maps/audit-history-moc.md
---

# Strategies MOC

> Every signal family, its current status, and the evidence chain.
> Updated 2026-04-17 with canonical composite audit and current live strategy files.

## Strategy status board

| Strategy | Status | Key metric | Reference |
|---|---|---|---|
| **IFVG 6-component** | **VALIDATED + LIVE (v26)** | +7.55 R/day, 60% WR, 1707 trades/247d | [[ifvg-composite-audit-20260329]] |
| **IFVG market-entry** | **VALIDATED** | +17.9 PPD with 100% fill, zero assumptions | [[ifvg-composite-audit-20260329]] |
| **Lumi V3 (NQ)** | **LIVE** | V2 signal chain + v26 order patterns (dynamic sizing, IsExiting, orphan safety) | [[lumi-strategy-v3-2026-04-18]] |
| **Lumi V2 (ES)** | **LIVE, unaudited** | No Python audit yet | [[lumi-es-strategy-v2-2026-04-10]] |
| **SF Portfolio** | **TICK-CONFIRMED** | +3.34 R/day on 260d, RUNNER fat-tailed | [[sf-portfolio-cluster]] |
| **OR_FAIL (V13)** | **VALIDATED** | 89% WR, Calmar 22.9 | [[or-fail]] |
| **Doji Break** | **VALIDATED (Asia)** | 89% WR, Calmar 589 | [[doji-break]] |
| **Wick Fade 5m** | **UNPROVEN** | 10/10 WF but never live/sim, 68% from flip | [[wickfade-complete]] |
| **BOS FVG** | **DEAD** | tick +0.001 avgR | [[bos-fvg-failure-consolidated]] |
| **Sweep Fade** | **DEAD/MARGINAL** | Most variants dead | [[sweep-cluster]] |

## Current live production (Apr 2026)

### IFVG — TempoPortfoliov26.cs

The **canonical reference** is `composite_strategy.py` (607 lines) validated March 28-29, 2026. v26 is the NT8 port with identical configs plus dynamic sizing + safety fixes.

- [[ifvg-composite-audit-20260329]] — **START HERE.** 6-component results, market-vs-limit comparison, 3 bugs found+fixed, Python↔C# cross-reference verified.
- [[tempo-portfolio-v26]] — Current NT C# implementation. v26 adds dynamic position sizing (v25 fixed race condition, v24 introduced 6-component MTF cascade).
- [[ifvg]] — The signal concept.
- [[invalidation-rules]] — The "do not forget" rule ledger. Read before citing any pre-April number.

### Lumi — LumiStrategyV3.cs (NQ) + LumiESStrategyV2.cs (ES)

Extracted from TempoPortfolio between v19 and v23. Now runs standalone on both NQ and ES.

- [[lumi-strategy-v3-2026-04-18]] — **Current NQ strategy (V3).** V2 signal chain + v26 production patterns (dynamic sizing, IsExiting race fix, orphan safety, circuit breakers). 902 lines.
- [[lumi-strategy-v2-2026-04-10]] — **Superseded by V3.** V2 NQ with internal BT sim.
- [[lumi-es-strategy-v2-2026-04-10]] — ES port (still V2, no V3 ES yet, no Python audit).
- [[lumi-strategy-spec]] — **Superseded.** V15-era spec with FVG soft stop.

**Open work**: Lumi needs a market-entry tick-level test comparable to what was done for IFVG (`market_vs_limit_summary.json`). The +0.45R headline assumes limit fills. V3 does not change the signal chain — the tick-through caveat still applies.

**Overlay research (Apr 19 2026)**: [[lumi-overlay-research-2026-04-19]] — ES momentum alignment lifts WR from 56.5% to 63.9% (walk-forward validated). Leppyrd and SMT rejected. See `strategies/tempo/results/OVERLAY_RESEARCH_RESULTS.md`.

## Validated but not live

- [[sf-portfolio-cluster]] — SF Portfolio: tick-confirmed at +3.34 R/Day. Not deployed to NT live.
- [[or-fail]] — OR_FAIL: validated at 89% WR. Production system exists in trading-system.
- [[doji-break]] — Doji Break: Asia session only. 89% WR, Calmar 589.

## Dead strategies

- [[bos-fvg-saga-moc|BOS FVG Saga]] — Full case study. Three contamination layers, three audits.
- [[sweep-cluster]] — Most variants dead at tick level.
- [[v8-mining-synthesis]] — V8 mining: all invalidated.

## Key files on disk (not in vault as wiki pages)

| File | What | Date |
|---|---|---|
| `strategies/tempo/scripts/composite_strategy.py` | **Canonical IFVG Python engine** | Mar 28 |
| `strategies/tempo/scripts/AUDIT_RESULTS_20260329.md` | Bug report + post-fix results | Mar 29 |
| `strategies/tempo/results/optimized_composite_results.txt` | 6-component portfolio | Mar 28 |
| `strategies/tempo/results/market_vs_limit_summary.json` | Fill mode comparison | Mar 27 |
| `strategies/tempo/results/fullyear_mix_results.txt` | 172-config sweep | Mar 28 |
| `strategies/tempo/results/ifvg_15s30s_sweep_results_v2.json` | Latest 15S/30S sweep | Apr 7 |
| `lumi_backtest_1.75r.py` | Current Lumi Python backtest | Apr 10 |
| `strategies/tempo/LumiStrategyV3.cs` | **Current Lumi NQ (V3)** — v26 order patterns | Apr 18 |
| `strategies/tempo/results/lumi_1.75r_results.json` | Lumi 1.75R results | Apr 10 |
| `trading-system/results/IFVG_AUDIT_RESULTS_V2.md` | Production config 312d audit | Mar 9 |

## Version gaps (undocumented)

- TempoPortfolio v16–v25: 10 versions with no vault summaries
- LumiStrategyV1 (Mar 31): no vault summary
- Each version may contain regressions — v25 had a race condition that wasn't in v24

## Invalidation rules

Before trusting ANY pre-April 2026 number, check [[invalidation-rules]]:
- No bar-sim trailing stops
- No V10i alignment
- No break-even stops on Model 2
- No phantom B/E fills
- No P/D full-day H/L (future data)
