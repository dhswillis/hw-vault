---
created: 2026-04-17
updated: 2026-04-17
type: summary
sources:
  - /Users/harrisonwillis/Documents/strategies/tempo/scripts/AUDIT_RESULTS_20260329.md
  - /Users/harrisonwillis/Documents/strategies/tempo/results/optimized_composite_results.txt
  - /Users/harrisonwillis/Documents/strategies/tempo/results/fullyear_mix_results.txt
  - /Users/harrisonwillis/Documents/strategies/tempo/results/market_vs_limit_summary.json
  - /Users/harrisonwillis/Documents/strategies/tempo/scripts/composite_strategy.py
  - /Users/harrisonwillis/Documents/trading-system/results/IFVG_AUDIT_RESULTS_V2.md
related:
  - wiki/summaries/tempo-portfolio-v26.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/invalidation-rules.md
tags: [trading, ifvg, audit, validated, canonical]
---

# IFVG Composite Strategy — Validated Audit Results (Mar 2026)

> **This is the CANONICAL validated audit** for the 6-component IFVG MTF cascade that became [[tempo-portfolio-v26]]. These results were produced in late March 2026 using `composite_strategy.py` on tick-level Databento data. **All vault updates should reference these numbers, not re-derive from scratch.**

## IFVG Audit V2 — Production config (Mar 9, 312 days)

From `IFVG_AUDIT_RESULTS_V2.md`:

| Metric | Value |
|---|---|
| Trades | 3,665 (11.7/day) |
| WR | 72.6% |
| avg pts/trade | +9.6 |
| **PPD** | **+112.4** |
| $/day (1 contract) | $2,248 |
| Calmar | 56.5 |
| Daily WR | 73% |
| MaxDD | 620.5 pts |
| Walk-forward | **10/10 months PASS, OOS improves** |
| Bootstrap 95% CI | **[+8.2, +10.9] — well above zero** |

Every month profitable. Every session profitable. OOS outperforms IS.

## Optimized 6-component composite (Mar 28, 247 days)

From `optimized_composite_results.txt`, using per-component session whitelists + daily R limits + streak limits:

| Component | RT | Fills | T/d | WR | R/trade | R/Day | MDD(R) | Calmar |
|---|---|---|---|---|---|---|---|---|
| 15S_short | 3.0 | 323 | 1.3 | **66.3%** | +1.580 | +2.067 | 16.0 | 0.129 |
| 15S_long | 3.0 | 196 | 0.8 | 56.1% | +1.162 | +0.922 | 23.3 | 0.040 |
| 30S_short | 2.0 | 317 | 1.3 | 60.9% | +0.777 | +0.997 | 17.6 | 0.057 |
| 30S_long | 2.0 | 395 | 1.6 | 61.0% | +0.773 | +1.235 | 10.7 | 0.115 |
| 1M_short | 3.0 | 198 | 0.8 | **61.1%** | +1.407 | +1.128 | **7.4** | **0.152** |
| 1M_long | 3.0 | 278 | 1.1 | 52.5% | +1.065 | +1.198 | 13.7 | 0.088 |
| **COMBINED** | — | **1,707** | **6.9** | **60.0%** | — | **+7.547** | **21.8** | **0.346** |

**Combined portfolio**: +7.55 R/day, +$2,492/day, MDD 21.8R (270 pts), 57.3% win days.

## Market vs Limit entry (Mar 27, 30 days)

From `market_vs_limit_summary.json` — this comparison was **already done**:

| Entry mode | Fills | Fill rate | WR | avg R | PPD | Calmar |
|---|---|---|---|---|---|---|
| MARKET (bar close) | 354 | 100% | 48% | +0.57 | +17.9 | 0.12 |
| LIMIT_MID (FVG mid) | 285 | 81% | 35% | +0.54 | +23.6 | 0.13 |
| LIMIT_EDGE (FVG edge) | 252 | 71% | 27% | +0.35 | +10.5 | 0.07 |

**Even MARKET entry (100% fill, no limit assumption) shows +17.9 PPD and +0.57R.** The strategy has edge at any fill mode.

## Lumi post-fix (Mar 29)

Lumi 1H-to-3M look-ahead bug was found and fixed. Post-fix:
- 139 fills, 41% WR, +0.231 R/trade, +0.130 R/Day, MDD 12.4R
- Marginal but positive. Much smaller than IFVG components.

## Bug fixes applied (Mar 29 audit)

1. **CRITICAL — Lumi 1H-to-3M look-ahead**: `searchsorted` on start time instead of end time. Fix: offset by 1H duration.
2. **TP touch-fill instead of tick-through**: `>=` changed to strict `>` (limit orders require price PAST level).
3. **Missing flatten fallback**: added mark-to-market at last bar close.

23 NT C# bugs found in v20→v23 progression (all fixed in v23, carried to v24-v26). Python ↔ C# cross-reference verified: all 6 component configs match exactly.

## Why this supersedes the 2026-04-12 autonomous research

The autonomous session on 2026-04-12 rebuilt the IFVG sim from v14 logic instead of using the validated `composite_strategy.py`. It then found "no edge at tick-through fills" — but this was because:
1. It used a different (broken) fill mechanic that searched for retraces AFTER bar close
2. It didn't use the 6-component architecture with optimized session/streak/daily-R filters
3. The market_vs_limit comparison already showed **market entry at +17.9 PPD** — positive with zero fill assumptions

The correct reference implementation is `composite_strategy.py` (607 lines). All future backtesting should start there.

## File locations

| File | Purpose |
|---|---|
| `strategies/tempo/scripts/composite_strategy.py` | **Canonical Python engine** (607 lines) |
| `strategies/tempo/scripts/AUDIT_RESULTS_20260329.md` | Bug report + post-fix results |
| `strategies/tempo/results/optimized_composite_results.txt` | 6-component portfolio results |
| `strategies/tempo/results/fullyear_mix_results.txt` | 172-config parameter sweep |
| `strategies/tempo/results/market_vs_limit_summary.json` | Fill mode comparison |
| `trading-system/results/IFVG_AUDIT_RESULTS_V2.md` | Production config audit (312d) |
| `strategies/tempo/results/ifvg_15s30s_sweep_results_v2.json` | Latest 15S/30S sweep (Apr 7) |
