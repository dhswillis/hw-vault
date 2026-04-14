---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources: [raw-sources/STRATEGY_MINING_REPORT.md]
related:
  - wiki/summaries/research-journal
  - wiki/summaries/nq-playbook
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/syntheses/mining-reports-v1-v2-reconciliation.md
tags: [trading, nq, mining, v1-era, suspect-results]
---

# NQ Strategy Mining Report — 312d / 8,358 trades

**Source:** `raw-sources/STRATEGY_MINING_REPORT.md`
**Dataset:** 312 days of NQ tick data, 8,358 trades, `lb1m=2 lb5m=5` confirmation engine.
**Status:** ⚠️ V1-era results. Headline numbers (97.9% WR, 100% WR tiers) likely inflated by the [[v10i-look-ahead-bug|V10i look-ahead bug]]. See the [[mining-reports-v1-v2-reconciliation|reconciliation synthesis]] for how these numbers compare to the cleaned [[comprehensive-mining-report-v2|V2 report]].

## Three headline findings (as stated in the report)

1. **MTF_5M gate is the single most important filter**
   - Without `mtf_5m_aligned`: 46.5% WR, -0.28 R/T, -2,378R total
   - With `mtf_5m_aligned`: 70.8% WR, +0.28 R/T, +446R total
   - This single filter flips the system from loser to winner. See [[mtf-alignment]].

2. **[[bos-fvg|BOS_FVG]] + Body Confirmation = near-perfect entries**
   - BOS_FVG + mtf_5m + body confirms direction: **97.9% WR**, n=140, +220R, PF 61.5
   - BOS_FVG + mtf_5m + body break + close near extreme: **98.8% WR**, n=82
   - BOS_FVG + mtf_5m + failed auction: **90.6% WR**, n=127
   - ⚠️ Numbers of this magnitude are a classic symptom of look-ahead contamination. Cross-check against [[comprehensive-mining-report-v2|V2]], which after fixing the bug shows BOS_FVG at 63.5% WR.

3. **Combined TIER_A+B Portfolio**
   - 1,443 trades, 71.2% WR, +443R total, Calmar 28.4
   - 5.7 trades/day, +1.8 R/day
   - Max DD -15.6R
   - Only 1 losing month out of 13 (Nov -1.9R)

## Strategy tiers (as claimed)

- **TIER S (100% WR)** — BOS_FVG + mtf5m + body break (n=61), +delta confirms (n=60), +strong_engulfing (n=27), +two-bar momentum (n=19)
- **TIER A (>90% WR)** — BOS_FVG + mtf5m variants, n=81–140
- **TIER B (>75% WR, high volume)** — sweep + mtf5m combinations, n=95–651

## Entry mechanics (on BOS_FVG + mtf5m base, n=792)

| Filter | n | WR | R/T |
|---|---|---|---|
| Body confirms direction | 140 | 97.9% | +1.57 |
| Close near range extreme | 96 | 95.8% | +1.48 |
| Failed auction / wick >60% | 127 | 90.6% | +1.10 |
| Body break prev candle | 61 | 100% | +1.93 |
| Strong body >70% | 182 | 36.8% | — (hurts) |

## Session analysis

Best sessions for sweep + mtf5m: `ib_first` (80.5% WR), `silver_bullet` (78.6% WR), `power_hour` (breakevens).
Best sessions for BOS_FVG + mtf5m: `news_window` (70.5% WR), `ib_first` (69.2% WR). Power hour slightly negative — report recommends excluding.

## Direction analysis

Both longs and shorts profitable; shorts slightly stronger on BOS_FVG (68.6% WR vs 66.6%).

## R-target finding

Don't cap — edge comes from big R winners, not frequent small wins. Uncapped beats 0.75R / 1.5R / 3.0R cap versions.

## Report's own "key takeaways"

1. mtf_5m_aligned is mandatory
2. Body confirmation on BOS_FVG is the highest-edge entry technique
3. Failed auction (wick >60%) adds edge to both BOS and sweeps
4. Don't cap R targets
5. Avoid `strong_body (>70%)` on BOS entries — paradoxical
6. Drop power_hour for BOS_FVG
7. TIER_A+B hits the user's target: 71.2% WR, +1.8 R/day

## Why this summary is tagged "suspect-results"

The near-100% WR claims in TIER S and TIER A use the same single-bar `close > open` alignment pattern that [[v10i-look-ahead-bug|V10i was later shown to contaminate]]. When V2 re-ran the data with completed-bar alignment and a 1.5pt risk floor, BOS_FVG dropped from 97.9% WR (this report) to 63.5% WR ([[comprehensive-mining-report-v2|V2]]). The signal is still real and the top performer — just not superhuman. Treat headline numbers in this document as upper bounds, not targets.

## Source documents

- [[raw-sources/STRATEGY_MINING_REPORT]]
