---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/LUMI_STRATEGY_SPEC.md
related:
  - wiki/summaries/tempo-v14-corrections.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/tempo-cluster.md
tags: [trading, lumi, htf]
---

# Lumi Strategy Spec — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/LUMI_STRATEGY_SPEC.md` — **60 lines**, a compact specification for the Lumi engine extracted from the @LumiTraders account on X/Twitter in early 2026. Lumi is a multi-timeframe sweep + order-block + FVG cascade strategy that Harrison experimentally included in the v14 NinjaTrader implementation alongside the Tempo IFVG strategy.

## The strategy (as specified)

**Signal chain:**

1. **HTF level identification** — M15 / M30 / H1 / H4 FVG, Order Block, or swing high/low
2. **HTF sweep** — price sweeps the HTF level (M30 sweep for M30 levels, H1 sweep for H1/H4 levels)
3. **LTF Order Block** — on M3 (when HTF trigger was M30) or M5 (when HTF trigger was H1 or H4)
4. **LTF FVG** — FVG within/near the LTF OB. FVG can form before OR after the OB (unlike Tempo's IFVG, which requires pre-existing FVG)
5. **Entry** — at the FVG
6. **Target** — **2R measured from the FVG gap** (not from entry). This is a critical difference from IFVG's session-aware TPs.

**Session hours:** 8:00 AM – 4:00 PM EST, with strength windows at:
- 8:00 – 9:00 AM (pre-open / open)
- 10:00 – 11:00 AM (mid-morning)
- 1:30 – 2:30 PM (post-lunch)

## How Lumi differs from Tempo IFVG

| Dimension | Tempo IFVG | Lumi |
|---|---|---|
| FVG timing | Pre-existing FVG, inverted by sweep | Can form before OR after OB |
| HTF setup | Single-TF sweep → IFVG entry | HTF level → HTF sweep → LTF OB → LTF FVG |
| Target | Session-aware (100% runner / 30-70 / 55-35-10) | 2R from FVG gap, uniform |
| Stops | Soft stop at FVG edge, close-through | (spec doesn't specify clearly) |
| Primary confluence | SMT (NQ vs ES) | HTF alignment (structural) |
| Hours | 9:30–11:00 ET (primary window) | 8:00 AM – 4:00 PM ET full session |

These are different strategies. Lumi is a momentum-aligned multi-TF continuation; IFVG is a liquidity fade. They are not variations of each other, which is structurally *good* news for a portfolio — they should be uncorrelated and potentially diversifying.

## Reported vs backtested performance

**Twitter-claimed (from the @LumiTraders source spec):** 65% WR on ~200 backtested trades + 40 forward trades, tracked over 4 months.

**Harrison's v14 Python audit (30 days, 2025-03-03 → 2025-04-04):**

- **35 trades, 29 days, 20% WR, −3.4 pts/day, Calmar −0.4**
- Lumi targets 3R and needs ~25% WR for break-even. At 20% it's below.
- Only HTF_OB sources show a small edge (18.2% WR, +0.5 pts/day)
- Only the London session shows a small edge (23.5% WR, +1.5 pts/day)
- All other sources negative

## Why the gap

The 45-percentage-point WR gap between the Twitter-claimed 65% and the Python-audited 20% has multiple candidate explanations:

1. **Bar construction** — NinjaTrader's native bar construction differs from `pandas.resample('1min')`. Lumi's edge (if any) might live in the wick structure that pandas' resample loses.
2. **Different market regime** — the Twitter spec covers an earlier 4-month window. Harrison's audit covers a specific 30-day slice with different volatility characteristics.
3. **Spec ambiguity** — "FVG can form before OR after the OB" is a generous detection rule that might produce very different signal populations depending on how strictly the OB is defined.
4. **Overfitting on the source** — @LumiTraders' own backtest was on a dataset that may have been exactly the period the rules were tuned to.
5. **Missing filters** — the 60-line spec may not capture all the discretionary filters the live trader applies. Tempo's corpus also has this issue (281 transcripts vs 23 written rules).

## Disposition

**V14 (broken): EXCLUDED.** The v14 audit found adding Lumi to the IFVG portfolio hurt it: IFVG alone = +26.4 pts/day; IFVG + Lumi = +23.0 pts/day at 20% WR. The v14 implementation had bugs (Lumi FVG inversions, entry price errors, missing state management).

**V15 (rewritten to spec): LIVE.** The V15 rewrite fixed the V14 bugs: Lumi engine rewritten to true @LumiTraders spec (HTF levels on M15/M30/H1/H4), MSS requirement removed (not in original spec), FVG soft stop replacing swing-extreme stops (risk drops from ~29pt to ~9pt), M15 source filtering applied. Combined IFVG + Lumi portfolio: **+19.8 pts/day, 1,208 trades, 264 days, Calmar 23.1** at tick level on Databento data. See [[tempo-portfolio-v15]] for full results.

## Cross-references

- [[tempo-v14-corrections]] — where Lumi was tested and found underperforming
- [[tempo-ifvg-research]] — earlier doc that reported Lumi at 62% WR in an even more in-sample setting, before the v14 audit tightened
- [[tempo-cluster]] — master cluster summary that categorizes Lumi as Layer 2 suspect

## Source documents

- [[raw-sources/trading/tempo/LUMI_STRATEGY_SPEC]]
