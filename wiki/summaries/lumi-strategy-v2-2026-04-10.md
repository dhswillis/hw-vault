---
created: 2026-04-16
updated: 2026-04-16
type: summary
sources:
  - /Users/harrisonwillis/Documents/strategies/tempo/LumiStrategyV2.cs
  - /Users/harrisonwillis/Documents/lumi_backtest_1.75r.py
  - /Users/harrisonwillis/Documents/strategies/tempo/results/lumi_1.75r_results.json
related:
  - wiki/summaries/lumi-strategy-spec.md
  - wiki/summaries/tempo-portfolio-v15.md
  - wiki/summaries/tempo-portfolio-v26.md
  - wiki/summaries/lumi-es-strategy-v2-2026-04-10.md
tags: [trading, lumi, ninjatrader, live-strategy, current, nq]
---

# LumiStrategyV2 — Current Lumi (Apr 10, 2026, NQ)

> **Current production Lumi strategy** for NQ. File: `/Users/harrisonwillis/Documents/strategies/tempo/LumiStrategyV2.cs` (22,542 bytes, Apr 10, 2026). Separate from [[tempo-portfolio-v26]] — Lumi was extracted out of TempoPortfolio between v19 and v23 and now runs as a standalone strategy.

## What this supersedes

- [[lumi-strategy-spec]] — V15-era spec with FVG soft stop + 2R target. **Superseded**.
- Lumi engine inside TempoPortfolio v14 / v15 / v19. **Removed** from TempoPortfolio, now standalone.

## Default parameters

| Param | Default | Notes |
|---|---|---|
| Qty | 1 | Fallback for fixed contract size |
| MaxTrades/Day | 3 | Per-day limit |
| TargetR | **1.75** | Not 2R anymore — MFE analysis showed 2R→2.5R cliff; 1.75 harvests better |
| MinSweep (pts) | 1.0 | Price must wick past level by this much |
| SLBuf (pts) | 1.0 | Buffer past swing stop |
| OB Min Body % | 0.25 | Order Block body ≥ 25% of bar range |
| FVG Min Gap | 0.5 pt | Minimum FVG size |
| Swing LB | 3 | Swing lookback for swing H/L |
| Commission | 0.50 pt | Per trade |

## Architecture

Multi-timeframe: 1-min primary + 3m / 5m / 15m / 30m / 60m secondary (BIP 1-5).

Level sources built on each timeframe:
- **Swing H/L** (3-bar lookback per `SwLB`)
- **Order Block** (displacement candle: body / range ≥ 0.5, full displacement candle body as level)
- **FVG** (3-bar gap: bar[3].low > bar[1].high bearish, reverse for bullish; midpoint as level)
- **Equal H/L (liquidity)** (same H/L within 1.0pt across 3-8 bars)

## Signal chain

1. **HTF sweep** on M15 / M30 / H1: bar closes back inside after wicking past a level by ≥ MinSweep
2. **MSS confirmation** (phase): market structure shift in reversal direction
3. **LTF OB+FVG**: on M3 (when sweep was M30) or M5 (when sweep was H1), find OB + FVG cluster
4. **Limit entry** at FVG edge (or OB boundary if FVG formed before OB)
5. **Stop**: swing high/low ± SLBuf (typically 30-40pt away)
6. **Target**: entry ± (risk × 1.75) — fixed R multiple
7. **Cancel unfilled limit** after 5 x 1-min bars

## Latest Python backtest results (2025-03-01 → 2026-02-01, lumi_backtest_1.75r.py)

**LIMIT-FILL ASSUMED** — entry fills at OB/FVG edge without requiring a tick to trade through.

| Target R | Trades | WR | avg R | Total R | PF | Avg risk |
|---|---|---|---|---|---|---|
| 1.5R | 1,102 | 58.2% | +0.41 | +454 | 2.26 | 39 pts |
| **1.75R (current)** | **1,102** | **56.3%** | **+0.45** | **+493** | **2.29** | **39 pts** |
| 2R (V15-era) | 1,102 | 46.0% | +0.42 | +464 | 1.90 | 39 pts |

~4.2 trades/day over 260 days. Swing stop gives much wider risk (39pt) than the V15 FVG-soft-stop (~9pt), but target is tightened proportionally (1.75R vs 2R).

## Major changes from V15 Lumi

| | V15 Lumi (vault spec) | Lumi V2 (current) |
|---|---|---|
| Stop | FVG soft stop (gap edge + 1pt) | **Swing H/L ± buffer** |
| Avg risk | ~9 pt | **39 pt** |
| Target | 2R from FVG | **1.75R from entry** |
| Trades | 345 (H1+M30 filtered) | **1,102 (all TFs)** |
| Avg R expectancy | +0.064 | **+0.45** |
| Source filter | H1+M30 only (M15 excluded) | All TFs, no exclusion |

V15 vault doc concluded "FVG soft stop is superior" because the 2R target stayed the same while risk shrunk to ~9pt. **Latest V2 reversed that decision** — went back to swing stop. This contradicts V15's own reasoning. No vault doc explains why the reversal happened.

## Limit-fill caveat (same as v26)

The Python backtest's `simulate_trade` starts from `entry_price = fvg_entry` (or `ob_high` depending on entry method) and walks ticks forward for stop/target only. Entry is **assumed to fill** at the signal price. Real live performance requires price to trade through the limit level, which in Lumi's case is a mid-structure price (OB body or FVG boundary), generally closer to current price than IFVG's bar-extreme edges. Fill rate is probably better than IFVG, but the headline +0.45R is the ceiling.

Needs a tick-through variant to establish an honest floor. Not yet done for Lumi V2 (was done for IFVG v26 — see [[tempo-portfolio-v26]]).

## Open questions

- Why did V2 abandon FVG soft stop (V15's best finding) for swing stop? No explanation in code or docs.
- Why 1.75R instead of 2R? Probably from MFE analysis but no written rationale found.
- Is the +0.45R edge robust after tick-through test? Unknown.
- No vault doc for V1 (Mar 31) or intermediate iterations between V15 and V2.
- Interaction between LumiV2 (running alongside TempoPortfoliov26) is not characterized — the V15 doc reported 0.131 daily correlation between IFVG and Lumi legs, but that was the V15 Lumi flavor. Current V2 correlation vs current v26 IFVG is unknown.

## Cross-references

- [[lumi-strategy-spec]] — V15-era spec, now superseded
- [[tempo-portfolio-v15]] — V15 combined portfolio (IFVG + Lumi), also superseded
- [[tempo-portfolio-v26]] — current IFVG MTF cascade, runs alongside LumiV2 in live production
- [[lumi-es-strategy-v2-2026-04-10]] — ES port (new)
- [[invalidation-rules]] — Rule 1 (bar-sim trailing) not applicable; Lumi uses hard stop + fixed R target
