---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/summaries/comprehensive-mining-report-v2.md
tags: [trading, exit, risk-management, fat-tail]
---

# BE+Trail Exit Mechanism

**BE+Trail** (break-even + trailing stop) is a trend-following exit model: move the stop to breakeven at 1R, then trail it. It converts losing trades into near-zero breakevens, then lets winners run. It is the exit that makes [[bos-fvg|BOS_FVG]]'s fat-tail edge materialize.

## Why it matters

[[comprehensive-mining-report-v2|V2]] numbers make this stark. For BOS_FVG (n=11,348):

| Exit | WR | avg R |
|---|---|---|
| Fixed R=1.0 target | 39.7% | **−0.260** |
| BE+Trail | 63.5% | **+1.354** |

Same signal, same entries — but the fixed-target exit is a losing strategy and BE+Trail is the best signal in the dataset. **The exit mechanism is the edge.**

## The fat-tail distribution

BOS_FVG's BE+Trail returns break down as:
- **29.5% of trades** are 2R+ runners producing **+17,461R**
- **70.5% of trades** combined produce **−2,094R**

Without runners, BOS_FVG loses money. Capping R targets at 1-2R kills the edge. Classic fat-tail: a small fraction of trades is doing all the work.

## Signal-by-signal comparison

| Signal | BE+Trail WR | R=1.0 WR | WR gap | Near-zero % | BE avgR | R=1.0 avgR |
|---|---|---|---|---|---|---|
| BOS_FVG | 63.5% | 39.7% | +23.8% | 19.4% | +1.354 | −0.260 |
| fvg_exit_fvg | 67.2% | 38.0% | +29.2% | 30.3% | +0.589 | −0.290 |
| fvg_exit_body | 60.5% | 50.3% | +10.2% | 34.8% | +0.265 | −0.027 |
| sweep_wick | 47.5% | 45.6% | +1.9% | 33.0% | −0.313 | −0.111 |

Signals with high "WR gap" are signals whose edge comes from asymmetric payoffs. Signals with low WR gap (like `sweep_wick`) don't benefit from BE+Trail — they need fixed high-R targets instead.

## Two distinct exit books

This is the basis for Tempo's two-book portfolio construction:

1. **FVG trend-following book** — BE+Trail on BOS_FVG + fvg_exit variants. High WR, captures runners.
2. **Sweep mean-reversion book** — fixed 5.0R targets on sweep_wick / sweep_close_through / swing_failure with `al≥2` filter. Low WR, big binary payoffs.

Average pairwise correlation between these books is **0.134** — nearly uncorrelated. Portfolio diversification is real.

## The 100% WR anomaly explained

BOS_FVG + body_confirms + `co≤0` + BE+Trail shows 100% WR on 309 trades. This is real but **explained by the BE mechanism, not the signal**:

- All 309 trades reach 1R (minimum max_r_reached = 1.11R; avg 7.01R)
- BE triggers, so the worst case is a near-zero win
- At fixed R=1.0 the same subset shows 91.9% WR (still excellent, not 100%)
- Only 12 trades (3.9%) end as near-breakeven wins with 0 < r < 0.1

In other words: the 100% WR isn't 309 "winners" — it's runners that all reach 1R first and then either run or breakeven. The economic content is the R distribution, not the WR.

## Rules of thumb

- **Never cap R** on signals whose BE+Trail R/T exceeds their best fixed-target R/T
- **Use fixed R targets** on low WR-gap signals (sweep family) — they benefit from binary big-payoff exits
- **Pay attention to near-zero-win percentages** — the higher it is, the more the BE mechanism is doing (not the signal)
- **Report both BE+Trail and fixed R=1.0** numbers so anyone reading can see which mechanism is contributing
