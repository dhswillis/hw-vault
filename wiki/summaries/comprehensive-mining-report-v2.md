---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources: [raw-sources/COMPREHENSIVE_MINING_REPORT.md]
related:
  - wiki/summaries/research-journal
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/syntheses/mining-reports-v1-v2-reconciliation.md
tags: [trading, nq, mining, v2, research]
---

# NQ Futures Mining Report V2 (Corrected)

**Source:** `raw-sources/COMPREHENSIVE_MINING_REPORT.md`
**Dataset:** 34,323 trades across 258 trading days (Feb 2025 – Feb 2026)
**Status:** Cleaned version of V1 — three major bias sources fixed.

## What V2 corrects from V1

1. **Look-ahead bias fixed** — simulation now starts at `idx+1` (next bar after entry), not `idx`. V1 was checking the entry bar's own high/low against targets; since entry happens at the close, that was already in the past.
2. **Slippage added** — 0.4pt on all market entries.
3. **Minimum risk floor** — 1.5pt floor prevents tiny-stop trades from inflating R multiples. V1 had no floor, and 53% of `body_stop` trades had &lt;3pt risk producing artificial 10R+ winners.

These corrections dropped `fvg_exit_body_stop` from V1's +15,623R / R/T +3.68 down to V2's +502R / R/T +0.27 — the edge was almost entirely spurious.

## Signal ranking (V2, corrected)

| # | Signal | n | BE+Trail WR | Total R | R/T | WF stability |
|---|---|---|---|---|---|---|
| 1 | [[bos-fvg\|BOS_FVG]] | 11,348 | 63.5% | +15,368 | +1.354 | 63.6%→63.4% stable |
| 2 | fvg_exit_fvg_stop | 3,767 | 67.2% | +2,219 | +0.589 | 67.8%→66.6% stable |
| 3 | fvg_exit_body_stop | 1,892 | 60.5% | +502 | +0.265 | 63.5%→57.3% slight drift |
| 4 | sweep_wick_sweep (al≥2 co≤2 @ 5.0R) | 3,045 | 26.4% | +1,126 | +0.370 | stable |
| 5 | sweep_close_through (al≥2 @ 5.0R) | 1,216 | 35.4% | +883 | +0.726 | 70.3%→72.0% stable |
| 6 | swing_failure (al≥2 @ 5.0R) | 1,087 | 24.9% | +342 | +0.315 | stable |
| 7 | sweep_fvg_combo | 541 | 59.1% | +222 | +0.411 | stable |

[[bos-fvg|BOS_FVG]] is the core signal by total R and walk-forward stability.

## MTF alignment is the top filter

[[mtf-alignment|MTF stacking]] preserves perfect monotonic WR improvement in V2 — from 40.8% WR at 0 TFs aligned to 81.0% at 6 TFs aligned. This is the most important filter in the system.

## V10i alignment bug — critical finding

The V1-era `get_alignment()` function suffered [[v10i-look-ahead-bug|look-ahead bias]]: it checked `close > open` on unclosed higher-TF bars, using future close prices. A config that showed 91.1% WR / 2,035 trades dropped to 37.6% WR / 340 trades after the fix. **The entire edge was look-ahead.**

V2's alignment uses 3-bar HH/HL trend pattern on completed bars only — no look-ahead.

## BE+Trail mechanism

The [[be-trail-mechanism|BE+Trail exit]] converts losers into near-zero breakevens and lets winners run. BOS_FVG's edge is entirely in the trail: 29.5% of trades are 2R+ runners generating +17,461R; the remaining 70.5% combined for -2,094R. Classic fat-tail distribution.

## Recommended portfolio

Combined 7-strategy portfolio:
- **22,896 trades** across 258 days
- **+20,662R** total
- **211/258 green days (81.8%)**
- **13/13 green months**
- **Daily avg +80.1R**
- **Max cumulative DD -48.6R**
- **Calmar 425**
- Average pairwise correlation 0.134

Two distinct books, nearly uncorrelated:
1. **FVG Trend-Following** — BOS_FVG + fvg_exit variants, BE+Trail, captures runners
2. **Sweep Mean-Reversion** — wick/close/swing, 5.0R fixed targets, low WR big payoffs

## Sniper/Sharpshooter/Gunner tiers

- 533 Snipers (WR ≥ 70%, n ≥ 50, WF pass) — mostly BOS_FVG + body_confirms
- 307 Sharpshooters (WR ≥ 60%, n ≥ 200, WF pass)
- 30 Gunners (WR ≥ 50%, n ≥ 500, WF pass)

## Raw data artifacts

- `NQ_Mining_Results_V2.xlsx` — 8-sheet workbook
- `full_mine_v2_batches/batch_0-5.json` — 34,323 trades
- `replicate_v10i_batches/` + `replicate_fixed_batches/` — V10i bug replication
- Scripts: `build_portfolio.py`, `correlation_analysis.py`, `stress_test_v10i.py`
