---
created: 2026-04-10
updated: 2026-04-10
type: synthesis
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
  - raw-sources/STRATEGY_MINING_REPORT.md
related:
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/summaries/strategy-mining-report-312d.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/be-trail-mechanism.md
tags: [trading, synthesis, audit, v1-vs-v2, nq]
---

# Mining Reports V1 vs V2 — Reconciliation

Two NQ mining reports exist in [[tempo-trading-system|Tempo]]'s raw sources. Their headline numbers disagree by an order of magnitude on the same core signal. This page reconciles them.

## The two reports

| | [[strategy-mining-report-312d|STRATEGY_MINING_REPORT (V1-era)]] | [[comprehensive-mining-report-v2|COMPREHENSIVE_MINING_REPORT V2]] |
|---|---|---|
| Dataset | 312 days, 8,358 trades | 258 days, 34,323 trades |
| Slippage | none | 0.4pt |
| Min risk | none | 1.5pt |
| Look-ahead | **V10i bug** (unclosed HTF bars) | **fixed** (completed bars only) |
| Alignment formula | Single bar `close > open` on HTF | 3-bar HH/HL on closed bars |
| Simulation start | `idx` (entry bar itself) | `idx+1` (next bar after entry) |

## The headline disagreement

Same signal, same symbol, ~same instrument-years of data:

| Signal | V1 headline | V2 corrected |
|---|---|---|
| BOS_FVG + body confirms | **97.9% WR**, n=140, +220R | — |
| BOS_FVG + body break + close near extreme | **98.8% WR**, n=82 | — |
| BOS_FVG (all, BE+Trail) | — | **63.5% WR**, n=11,348, +15,368R |
| BOS_FVG + strong_body > 70% | 36.8% WR (paradoxical) | — |
| `fvg_exit_body_stop` | +15,623R total, R/T +3.68 (V1) | **+502R total, R/T +0.27** (V2) |

The most dramatic collapse is `fvg_exit_body_stop`: **V2 is 1/30th of V1**. The V2 report explicitly identifies this as an artifact of V1's lack of a minimum-risk floor — 53% of `body_stop` trades had <3pt risk, producing artificial 10R+ winners on micro-stops.

## The three corrections that did the work

V2 pinned down three separate biases, each of which contributed independently:

### 1. Look-ahead at entry (`idx` → `idx+1`)
V1 checked the entry bar's own high/low against targets. But since entry happens at the close, the high/low of that bar already happened in the past. Moving the simulation start to `idx+1` (next bar after entry) removes this.

### 2. Minimum risk floor (1.5pt)
V1 had no risk floor. Tiny-stop trades (<3pt risk) produced artificial 10R+ winners when price kept moving. The 1.5pt floor — about 6 NQ ticks — is realistic for actual execution and eliminates the inflation.

### 3. [[v10i-look-ahead-bug|V10i alignment look-ahead]]
V1 used `get_alignment()` with a single-bar `close > open` check on unclosed higher-TF bars. The `close` field of an unclosed bar is future data. Fixing this is what collapsed the 91.1% WR "sniper" tier to 37.6% WR with a NEGATIVE R total. **0 of 13 months were green after the fix.**

## What survived

Despite the collapse, the headline finding of both reports agrees: **[[bos-fvg|BOS_FVG]] is the best single signal**, and **[[mtf-alignment|MTF alignment]] is the most important filter**. The direction is right; only the magnitude was wrong.

- V1: BOS_FVG + mtf_5m + body confirms → 97.9% WR (inflated)
- V2: BOS_FVG with BE+Trail → 63.5% WR (honest)

63.5% WR on 11,348 trades with walk-forward stability (IS 63.6% → OOS 63.4%) is **still a strong edge**. The 7-strategy V2 portfolio produces +20,662R on 22,896 trades with Calmar 425. The system is real — just not superhuman.

## What died

The entire "sniper tier" of 100% WR configurations from V1. The audit found that 533 "Sniper" configs (WR ≥ 70%, n ≥ 50, WF pass) survive V2's cleanup — but the TIER_S claims of n=61 at 100% WR do not. Most of the near-perfect configs were alignment-look-ahead artifacts or tiny-risk R-multiplication artifacts, not real edges.

## The load-bearing insight: [[be-trail-mechanism|BE+Trail is the edge]]

V2 made one thing unambiguously clear: BOS_FVG's edge **does not exist at fixed R targets**.

- BOS_FVG, R=1.0 fixed target: **−0.260 R/T** (loser)
- BOS_FVG, BE+Trail: **+1.354 R/T** (best signal in the dataset)

29.5% of trades are 2R+ runners producing +17,461R. The other 70.5% combined produce −2,094R. Without BE+Trail converting losers to breakevens and letting winners run, the edge disappears. This is a classic fat-tail distribution — a small fraction of trades does all the work.

The V1 report didn't make this framing explicit because the V1 alignment bug was producing enough spurious "wins" at fixed targets that the mechanism distinction got lost in the noise.

## Practical rules from the reconciliation

1. **Treat any V1-era mining WR claim as an upper bound.** Real numbers are 30-50% lower.
2. **Trust the V2 portfolio construction.** +80.1R/day average, Calmar 425, 13/13 green months, avg correlation 0.134 — those numbers survive all three bias corrections.
3. **BOS_FVG + BE+Trail + MTF alignment** is the core setup. Everything else is elaboration.
4. **Don't test alignment formulas that consult unclosed HTF bars.** Ever. Use `mask = tf_c.index + TF_DURATION[tf_label] <= entry_time` as the standard.
5. **Always impose a minimum risk floor** (≥ 1.5pt for NQ) before computing R multiples. Otherwise tiny stops inflate winners.
6. **Report both BE+Trail and fixed-R numbers** on every new signal audit, so the mechanism contribution is visible.
7. **Audit any signal claiming > 90% WR on n ≥ 50.** That magnitude is a red flag for look-ahead, not a celebration.

## Open questions

- The sweep-family signals (sweep_wick, sweep_close_through, swing_failure) all survive V2 with positive R at 5.0R fixed targets and `al≥2` filtering. But their V1 numbers were also inflated, and the V2 WRs are 24.9%–35.4%. At those WRs, sample size matters a lot — are the survivors genuinely stable or noise-protected? The walk-forward numbers say stable, but the effective sample size per cell is small.
- The `fvg_exit_body_stop` collapse from +15,623R to +502R was explained by the risk floor, but **some** of the drop could also be the look-ahead fix. V2 doesn't decompose which correction contributed how much — a useful future audit.
- V1 claimed `strong_body > 70%` on BOS entries "paradoxically hurts" performance (36.8% WR). V2 doesn't revisit this specific filter. Did the strong-body filter also benefit from look-ahead, which is why it now looks bad? Worth re-running.
