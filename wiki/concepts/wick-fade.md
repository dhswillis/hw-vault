---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/sweep/WickFade_Complete_Findings.docx
  - raw-sources/trading/sweep/15S_Wick_Fade_Findings.docx
related:
  - wiki/concepts/ifvg
  - wiki/summaries/wickfade-complete.md
  - wiki/summaries/15s-wick-fade.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/syntheses/research-arc-map.md
tags: [trading, signal, nq, mean-reversion, unproven]
---

# Wick Fade

The mean-reversion family of NQ fade strategies that enter when price breaks a previous bar's wick (high/low) and bets on a snap-back. **Backtest-only — never deployed live or to sim.** The 15S flip variant is explicitly dead (0/216 configs profitable at tick level). The 5m variant claims 10/10 walk-forward OOS but 68% of its edge comes from the flip leg, and an independent test (Research Log March 4, 2026) found the same concept marginal at +0.29 R/day on full year data.

## The core bet

When the current bar *breaks* the previous bar's extreme, price tends to **snap back within the same bar or the next few bars**. The 15-second research doc puts a hard number on this: **92.4% snap-back rate** within the same 15S candle, with a median break excursion of only 3 points past the wick. The 5-minute version doesn't quantify snap-back rates but uses the same underlying logic and produces 97% daily win rate on fixed 5:1 R:R targets.

This is a pure mean-reversion play, not a continuation play. It works best in chop-prone sessions (NY morning, data releases) and not so well in strong trend days.

## Two production variants

The corpus has two distinct wick-fade strategies that share the family name but are different in mechanics:

### [[15s-wick-fade|15S Wick Fade]]

- 15-second bars
- Entry at the previous bar's wick (exact, 0 tick offset)
- 4.50pt stop, 10pt target (or 6.75pt for higher WR)
- Window: 08:30–10:00 CT (best 08:30–09:15)
- **Fade only** — the flip variant has no edge (0/216 configs profitable)
- 43% WR, $30.89/trade net, ~$1.27M gross P&L over 260 days

### [[wickfade-complete|WickFade 5m]]

- 5-minute bars
- Fade breakout of previous bar H/L
- 3pt stop, 15pt or 30pt target
- Window: 8:30 AM – 2:00 PM CT, flatten 2:55 PM
- **Flip allowed** — the 5m version has flip enabled and uses single-position architecture
- **+77 to +94 R/day**, 92–97% daily WR, Calmar 1.32–2.12
- **10/10 walk-forward OOS months positive**

## What makes this real (not suspect)

The wick fade family is one of the very few Layer 2 results that survives the project's multi-layered contamination patterns:

1. **Tick-level validated.** Both docs use Databento tick data. The 15S doc is explicit that Pine Script results went from 84% WR (v58) → 56% (v59) → 36% (v60) → 20.4% (tick-level ground truth) on the *flip* side — i.e., the bar-level sim was exactly as contaminated as BOS_FVG's was, and only tick replay showed reality.
2. **Fixed stops and fixed targets** (the primary production config). Bar-sim-safe by construction — see [[bar-sim-trailing-bug]]. The fade either hits the target or it doesn't; the path within the bar doesn't matter.
3. **Multi-timeframe robustness.** The WickFade logic works on 5m, 15m, AND 1H. A single-timeframe overfit wouldn't generalize.
4. **Flat parameter surface.** Performance is stable across stop/target combinations. No sharp peaks suggesting curve-fitting.
5. **ATR-adaptive stops don't help.** If the edge were spurious, scaling by vol should destroy it. It doesn't — which means the edge is structural, not vol-dependent.
6. **Walk-forward 10/10.** Parameters selected on training data perform in OOS months. This is the strongest anti-overfit evidence anywhere in the corpus.
7. **Position integrity verified.** Zero overlapping positions across all configurations, all 258 days. No phantom simultaneous-long-and-short bugs.

## What DOESN'T work (across both variants)

- **Trail stops on 1-minute data** — per [[bar-sim-trailing-bug]], this inflates by path-reconstruction artifacts. The WickFade doc explicitly documents 1m S=2 T=20 with trail at **+290 R/day** collapsing to **−118 R/day** when the trail is removed — an extreme reductio of the bar-sim trailing bug, documented in March 2026.
- **The flip** (on 15S) — 0 out of 216 configurations profitable
- **[[be-trail-mechanism|Break-even stops]]** — Option C (1.5R + BE at 1.0R) drops WR from 46.8% to 36.1% and Calmar from 2.8 to 1.1 on WickFade 5m. Same project-wide finding as Model 2.
- **ATR-adaptive stops** — fixed decisively beats adaptive on both variants
- **Tight entry offsets (2–4 ticks shy of wick)** — entering earlier reduces performance; the wick itself is the natural support/resistance
- **IB_FAIL and OR_FAIL signals** — on WickFade 5m, these are net negative across all exit strategies; exclude from production

## The SWING5M signal

The WickFade 5m doc identifies one specific signal — **SWING5M** — as carrying all the edge. IB_FAIL and OR_FAIL are separate signals in the same backtester that produce negative results. When reading the 5m headline numbers (+77 R/day, Calmar 2.12), it's specifically the SWING5M variant that's producing them, not the whole signal library.

## Lessons this family taught the corpus (that didn't propagate)

The wick fade docs hit two major findings that, if generalized, would have saved months of BOS_FVG work:

1. **Pine Script bar-level sim is systematically biased for path-dependent strategies** — the 15S doc walked through the v58 → v60 → tick journey, documenting that each bar-level fix still left selection bias because OHLC can't encode the sequence of high vs low within a bar. This is literally the [[bar-sim-trailing-bug]], identified in March 2026.

2. **Trail-stop P&L can be an artifact of execution-assumption violations** — the 5m doc's honest disclosure that 1m trailing gets +290 R/day while 1m stop+target gets −118 R/day is the cleanest statement of the same bug anywhere in the corpus. It's just that the BOS_FVG researchers didn't read this doc's disclaimer and apply it to their own 1m trailing work.

Both findings were sitting in the sweep cluster unreinged until this session. The Unified Brain's Mem0 seed should absolutely include them.

## Production status

**UNPROVEN — never deployed.** WickFade 5m `WickFade5mV1.cs` was built but never paper-traded or sim-tested. The headline numbers (+77 R/day, 10/10 walk-forward) are backtest-only and contradicted by:

1. The Research Log (March 4, 2026) tested the same wick reversion concept and found it **marginal** (+0.29 R/day on full year, degrading 5x from quick test)
2. 68% of the 5m edge comes from the **flip** leg, and the 15S doc proved the flip is dead at tick level on that timeframe
3. The strategy was never validated outside the original backtester

The project's current live portfolio is IFVG + Lumi (V15). Wick Fade remains a research artifact.

## Cross-references

- [[wickfade-complete]] — full summary of the 5m variant's research
- [[15s-wick-fade]] — full summary of the 15-second variant
- [[bar-sim-trailing-bug]] — the bug that killed BOS_FVG but that wick-fade identified first
- [[be-trail-mechanism]] — why BE stops hurt WickFade 5m
- [[sf-portfolio-cluster]] — another Layer 2 track with fixed targets; wick fade and SF portfolio are the two defensible survivors
- [[research-arc-map]] — where wick fade fits in the overall research arc

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec|WickFade Strategy Spec]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/WORKING_LOG|WORKING LOG]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/RESEARCH_LOG_20260304|RESEARCH LOG 20260304]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/wickfade_5m_findings_20260303|wickfade 5m findings 20260303]]
