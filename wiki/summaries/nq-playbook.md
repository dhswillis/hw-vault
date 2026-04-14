---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/context-engine/NQ_PLAYBOOK.md
related:
  - wiki/summaries/strategy-mining-report-312d
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/research-journal.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, mining, backtest, suspect-results, superseded]
---

# NQ Playbook — Summary

> **Layer 2 — contaminated.** Almost every performance number in this doc is either (a) inflated by the [[bar-sim-trailing-bug]] for trailing-stop results, (b) inflated by the [[v10i-look-ahead-bug|V10i alignment look-ahead]] for MTF results, or (c) inflated by the phantom B/E bug in `micro_structure.py` for BE=1.0 variants. Read this summary for the *shape* of the research (what was tried, what the narrative was), not for the R-numbers. See [[tempo-three-layers]] for the framework and [[bos-fvg-failure-consolidated]] for the 2026-04-10 tick replay that killed the headline.

## What the source is

`~/HW/raw-sources/trading/context-engine/NQ_PLAYBOOK.md` — **780 lines**, the master findings document for the V10-series mining research. Originally titled "Strategy Bible," later renamed "NQ Playbook." It is Harrison's single biggest claim-to-fame document — the consolidated "what survived the V10 cleanup" reference — and almost everything in it is now suspect.

## What it claims (the headline numbers)

The doc's self-identified flagship findings:

| Claim | Metric |
|---|---|
| BOS_FVG 15S trailing (V10z conservative) | +1.141 avgR, Calmar 263.6, +2.51 R/day, WF 4/4 |
| 3-signal portfolio (BOS_FVG + vol_spike_FVG + ORB_breakout) | 887 trades, +1.008 avgR, Calmar 94.4 |
| 4-signal portfolio (+ pre_gap_fill) | 1008 trades, +0.880 avgR, Calmar 94.4 |
| NY AM session | 75% WR, +12.1 pts average |
| London session | 69.5% WR, +7.9 pts, 100% runner |
| Slippage stress test | survives 2pt exit slip at Calmar 145.6 |
| 1-minute delay test | kills 90% of Calmar — "edge is front-loaded, do NOT delay" |

Every one of these came from a bar-level trailing-stop simulation on 1-minute or 15-second OHLC data. The 2026-04-10 tick audit on the same signal generator showed the trailing-stop edge is a bar-reconstruction artifact — tick replay gives +0.001 avgR, not +1.141 avgR. Everything in this table downstream of BOS_FVG is affected proportionally.

## Why this doc is tagged suspect

1. **Trailing-stop results on OHLC bars** — see [[bar-sim-trailing-bug]]. The +1.141 avgR / Calmar 263.6 headline is what the bar sim reports; the tick-level replay on the same 60-day sample gives +0.001 avgR.
2. **"Conservative" intra-bar re-check fix only** — the doc claims V10z's "fixed intra-bar OHLC ordering bug" addressed the trailing inflation. It didn't. The re-check catches <1% of affected trades and gains ~+0.005 avgR. The full bar-vs-tick gap is +0.29 R/trade and requires actual tick replay.
3. **Walk-forward pass doesn't mean anything** when the underlying simulator is wrong. 4/4 OOS quarters positive on a bar-sim trailing test just means the bar-sim bug is consistent across time.
4. **The "3-signal" and "4-signal" portfolio composites** — every leg uses the same trailing simulator. vol_spike_FVG, ORB_breakout, and pre_gap_fill all inherit the bar-sim inflation proportionally to how much their edge lives in the trail versus the static target.
5. **Missing tick-level cross-validation** — the doc's only defense against the bar-sim bug was the conservative re-check, which was done at the bar level. A tick-level cross-check would have caught the problem in March 2026 instead of April.

## What the doc still gets right

- **Dead-strategy list** — london_breakout, VP_POC_retest, fib_retracement, failed_breakout, double_BOS_momentum, VWAP_mean_reversion. These were negative across every parameter set, and negative-across-every-parameter is robust to most contamination. The dead list stays.
- **Session observations** (qualitatively) — NY AM is the most active, London has structural issues on IFVG-style strategies, NY PM underperforms. These patterns hold up across tick- and bar-level analyses; it's only the *magnitude* of the edge in the "good" sessions that's inflated.
- **Swing lookback 2 beats 3** — this finding is about signal *density*, not trade management. Robust to the trailing bug.
- **BOS displacement 2.0 → 1.0 improves trade count** — same reasoning; a signal-count optimization, not a trade-management claim.
- **NautilusTrader cross-validation** — 9,188 trades, 55.6% WR, +0.286 avgR. NautilusTrader is engine-native (runs tick-level trade management internally), so the +0.286 avgR number is not contaminated the same way. But it's also lower than the NQ_PLAYBOOK's +1.141 flagship — this was in fact an early warning sign that should have been treated as a red flag before April.

## What to do with this doc

- **Do not cite any trailing-stop R-number from this doc as evidence of edge.** Every such number is bar-sim-contaminated.
- **Do cite the dead-strategy list, the signal-density findings, and the qualitative session observations.** Those are robust.
- **Do read this doc as the best available picture of the V10 research arc** — it's useful as a map of what was tried, just not as a scorecard of what worked.
- **Do pair it with `BOS_FVG_FAILURE_CONSOLIDATED.md`** — that's the 2026-04-10 correction to this doc's central claim.

## Cross-references

- [[bar-sim-trailing-bug]] — structural reason the headline numbers are wrong
- [[bos-fvg-failure-consolidated]] — 2026-04-10 tick audit
- [[research-journal]] — companion narrative doc for the same research arc
- [[be-trail-mechanism]] — the exit mechanism whose edge was the illusion
- [[v10i-look-ahead-bug]] — the earlier alignment bug that V10 partially fixed
- [[tempo-three-layers]] — this is Layer 2, contaminated by definition
- [[comprehensive-mining-report-v2]] — the V2 mining rerun that uses this doc's signal definitions and inherits its contamination

## Source documents

- [[raw-sources/trading/context-engine/NQ_PLAYBOOK]]
