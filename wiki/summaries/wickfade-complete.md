---
created: 2026-04-11
updated: 2026-04-21
type: summary
sources:
  - raw-sources/trading/sweep/WickFade_Complete_Findings.docx
related:
  - wiki/concepts/wick-fade.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/15s-wick-fade.md
  - wiki/summaries/sweep-cluster.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/syntheses/research-arc-map.md
tags: [trading, wick-fade, unproven, walk-forward]
---

# WickFade Complete Findings — Summary

> **STATUS: UNPROVEN.** Backtest-only strategy, never deployed live or to sim. The doc claims 10/10 walk-forward OOS positive with +77 R/day, but: (1) 68% of the edge comes from the flip leg, which is proven dead on 15S (0/216 configs profitable at tick level), (2) the Research Log (March 4, 2026) tested the same wick reversion concept independently and found it marginal (+0.29 R/day on full year, degrading 5x from quick test), (3) headline numbers have never been validated outside the original backtester. The doc's contribution is identifying the [[bar-sim-trailing-bug]] a month before the BOS_FVG audit.

## What the source is

`~/HW/raw-sources/trading/sweep/WickFade_Complete_Findings.docx` — **348 lines** (as converted via `textutil`), dated March 2026. Consolidated research findings for the WickFade NQ futures strategy across 258–312 trading days of tick-level Databento data. Covers multi-timeframe analysis (1m, 5m, 15m, 1H), exit management optimization, ATR-adaptive stop sizing, walk-forward validation, and position integrity verification.

This doc is a different strategy from [[15s-wick-fade|15S Wick Fade]] — they share the "fade the wick" family name but operate on different timeframes and entry definitions. 15S fades the break of the previous 15-second bar's wick; WickFade 5m fades the breakout of the previous 5-minute bar's high/low with an optional flip on stop-out.

## Strategy definition

**WickFade:** fade the breakout of the previous bar's high/low on NQ futures. Single-position architecture (no simultaneous long + short). Optional flip on stop-out (reverse the position when stopped). Tested on 1m, 5m, 15m, and 1H timeframes — the **5-minute timeframe is the sweet spot**.

## Headline result — 5m S=3 T=15

| Metric | Value |
|---|---|
| Timeframe | 5-minute |
| Stop | 3 pt ($60/contract) |
| Target | 15 pt ($300/contract) |
| R ratio | 5:1 |
| Trades | 17,093 (258 days, ~66/day) |
| WR | 39.8% |
| **R/day** | **+77.82** |
| Max DD | 36.7 R |
| **Calmar** | **2.12** |
| **Daily WR** | **97.3%** |
| Avg trade duration | 194 seconds |

## Alternative: 5m S=3 T=30 (higher R/day, higher variance)

- 14,303 trades (55/day), 26.7% WR
- **R/day: +94.48** (vs +77.82 for T=15)
- Max DD: 71.3 R
- Calmar: 1.32
- Daily WR: 92.6%

The T=30 variant produces more absolute R/day but with bigger drawdowns. T=15 has the higher Calmar and DWR — better risk-adjusted.

## The critical 1m trail finding (pre-discovered bar-sim trailing bug)

The doc reports a 1-minute trailing stop configuration that produces **+290.24 R/day with 44.5% WR** — absurd numbers, higher even than V10z BOS_FVG's +2.51 R/day flagship. But then it says this:

> "However, when stripping out the trail stop and using only stop+target exits, performance collapses to **−118.91 R/day**. This means the entire 1m edge comes from 37,784 trail stop exits that require instantaneous cancel/resubmit of StopLimit orders on every tick — unrealistic with 2pt stops in live NQ trading."

**This is the same [[bar-sim-trailing-bug|bar-sim trailing bug]] that killed BOS_FVG.** The doc identified it in March 2026 — about a month before the BOS_FVG audit formally named and characterized it — and correctly concluded that 1m trail results are "execution-assumption-dependent" (their framing) rather than real edge.

The honest response in the WickFade doc is:
> "The 5m results use pure stop+target with no trail dependency. This makes 5m the highest-conviction finding for live trading."

What the BOS_FVG project failed to generalize: if 1m wick fade's trail gains collapse to massively negative without the trail, **every 1m trailing strategy has the same problem**. The information was sitting in this doc for a month.

## Exit management research (5m, 21-day sample)

Eight exit strategies tested on 5-minute wick fade over 21 days. Key findings:

| Exit | WR | R/day | MaxDD | Calmar |
|---|---:|---:|---:|---:|
| **A) 1.5R fixed (baseline)** | 46.8% | +1.45 | 10.7 | 2.8 |
| B) 1.0R fixed | 57.5% | +1.32 | 11.3 | 2.5 |
| **C) 1.5R + BE at 1.0R** | 36.1% | +0.81 | 15.4 | 1.1 |
| D) Lock +0.25R at 1.0R | 56.9% | +1.14 | 13.9 | 1.7 |
| **G) Trail 0.5/0.15** | **73.1%** | **+1.35** | **5.5** | **5.2** |
| F) Trail only (0.5/1.0) | 44.2% | +0.55 | 11.2 | 1.0 |

**Winning exit strategy: Option G** — 1.5R target with trailing stop (0.5R trigger, 0.15R buffer). Calmar 5.2 with 73% WR and only 5.5R max DD.

**Important caveat:** Option G uses a trailing stop on 5-minute bars. Per the [[bar-sim-trailing-bug|bar-sim trailing bug]], trailing stops cannot be simulated on OHLC bars — the reported Calmar 5.2 may be inflated the same way BOS_FVG's was. The doc's own 1m vs 5m stop+target finding should apply: **if the 5m stop+target alone (Option A, +1.45 R/day, Calmar 2.8) is the "clean" result, Option G's +1.35 R/day Calmar 5.2 should be treated with the same skepticism** that BOS_FVG's trailing-stop headline got.

Recommendation: use **Option A (1.5R fixed target, no trailing, no B/E)** as the safe production config. Accept the lower Calmar for certainty that it's not a simulation artifact.

**Additional finding: BE stops hurt.** Option C (BE at 1.0R) drops WR from 46.8% to 36.1% and Calmar from 2.8 to 1.1. This matches the project-wide finding that [[be-trail-mechanism|B/E kills edge]] on Model 2 signals.

**Critical signal observation:** the SWING5M signal carries all the edge. IB_FAIL and OR_FAIL signals are net negative across all exit strategies and should be excluded from production.

## ATR-adaptive stops (don't help)

Tested whether scaling stops by ATR improves performance. Result:

| Config | R/day | Calmar |
|---|---:|---:|
| **Fixed S=3 T=15** | +6.82 | **6.36** |
| Fixed S=3 T=30 | +8.43 | 5.71 |
| ATR 0.5× R10 min2 | +3.98 | 1.88 |
| ATR 0.3× R10 min2 | +5.21 | 1.51 |
| ATR 0.7× R10 min3 | +2.34 | 0.48 |

**Fixed stops decisively beat every ATR variant.** The doc frames this as a positive finding: the edge is structural, not volatility-dependent. The strategy is robust across market conditions and doesn't need dynamic parameterization.

## Walk-forward validation — 10/10 positive

Rolling walk-forward with 3-month training / 1-month OOS test. Parameters selected on training data, applied unchanged to OOS:

| OOS Month | Config | OOS R/day | Result |
|---|---|---:|---|
| May 2025 | S=3 T=15 | +68.4 | POSITIVE |
| Jun 2025 | S=3 T=30 | +91.2 | POSITIVE |
| Jul 2025 | S=3 T=15 | +55.7 | POSITIVE |
| Aug 2025 | S=3 T=30 | +103.8 | POSITIVE |
| Sep 2025 | S=3 T=15 | +72.1 | POSITIVE |
| Oct 2025 | S=3 T=30 | +88.6 | POSITIVE |
| Nov 2025 | S=3 T=15 | +61.3 | POSITIVE |
| Dec 2025 | S=3 T=30 | +79.9 | POSITIVE |
| Jan 2026 | S=3 T=15 | +84.2 | POSITIVE |
| Feb 2026 | S=3 T=30 | +96.5 | POSITIVE |

**10/10 positive OOS months.** This is the strongest evidence against overfitting in the entire Layer 2 corpus.

## Position integrity verification

A critical concern for long-tail R strategies (10:1 reward:risk) is whether the architecture can accidentally open contradicting positions. Verified:

- **Zero overlapping positions** across ALL configurations and ALL 258 days
- Single-position architecture enforced via `t_on` flag / `_currentPos != null` check
- Positions are sequential: fade closes → flip opens on the same bar → never simultaneous
- Only 71 rapid fade→flip→fade cycles (within 60 seconds) out of 14,303 total trades — genuine sequential exits, not phantom overlaps

## Production recommendation (quoted from the doc)

| Parameter | Recommended Value |
|---|---|
| Timeframe | 5-minute |
| Instrument | NQ |
| Entry | Fade breakout of previous 5m bar high/low |
| Stop | 3 pt |
| Target | 15 pt (higher Calmar) or 30 pt (higher R/day) |
| Flip on stop | Yes |
| Trail stop | **Optional** — trigger 0.5R, buffer 0.15R (⚠ see caveat below) |
| Entry window | 8:30 AM – 2:00 PM CT |
| Flatten | 2:55 PM CT |
| Expected R/day | 77–94 R |
| Expected DWR | 92–97% |

**Caveat from this summary (not from the source doc):** the trail stop recommendation uses 5-minute bar data for trade management. Per [[bar-sim-trailing-bug]], trailing stops cannot be reliably simulated on bar data, even at 5m. The fixed-target (Option A, 1.5R) configuration is the bar-sim-safe version and is what should go to production first.

## What NOT to trade (from the doc)

- **IB_FAIL and OR_FAIL signals** — net negative across all exit strategies
- **1-minute wick fade with trail** — execution assumptions unrealistic for 2pt stops
- **ATR-adaptive stops** — underperform fixed stops

## NinjaTrader implementations built during research

- `WickFadeV14.cs` — 1-minute strategy with trailing stops. **Use with skepticism — per the bar-sim caveat, this is the contaminated variant.**
- `WickFade5mV1.cs` — 5-minute strategy with pure stop+target (no trail). **Clean — this is the production candidate.**

## BOS_FVG cross-reference (suspect claims from the doc)

The doc contains a section 7 that references a parallel BOS_FVG parameter sweep:

> "Composite Best config: 6,091 trades, 47.6% WR, +21.81 R/day, Calmar 335.3, 94.6% DWR, 100% weekly win rate. Swing lookback=2 was the single biggest improvement. **All 87 configurations tested were profitable — the BOS_FVG edge is structural.**"

**These BOS_FVG numbers are invalidated by the 2026-04-10 tick audit.** The "all 87 configurations profitable" claim is the signature pattern of the bar-sim trailing bug — when the simulation bug inflates every trailing-stop configuration uniformly, every parameter sweep looks like it's "structurally profitable." See [[bos-fvg-failure-consolidated]] for the paired bar-vs-tick replay that killed this claim.

The WickFade 5m stop+target results are NOT affected by this — they don't use trailing stops. The BOS_FVG sidebar in this doc should be treated as a suspect historical artifact.

## Why this doc stands up

1. **10/10 walk-forward OOS months positive** — strongest WF result in the corpus
2. **Flat parameter surface** — S=3 T=15 and S=3 T=30 both work; T=25, T=50 also work. No sharp peaks suggesting overfit
3. **Multi-timeframe consistency** — the same logic is profitable on 5m, 15m, AND 1H. A single-timeframe overfit would not generalize
4. **ATR-adaptive doesn't help** — if the edge were spurious, scaling by volatility should destroy it; it doesn't
5. **Zero overlapping positions verified** — no phantom simultaneous-long-and-short bugs
6. **Honest acknowledgment of the 1m trail problem** — the doc flags its own contamination rather than ignoring it
7. **Production recommendation uses pure stop+target, not trailing** — bar-sim-safe path to production

## Remaining risks (noted by the doc)

- **Execution slippage:** 3-point stops leave minimal room for slippage. Real fills may see 0.5–1pt per side.
- **Market regime change:** 258 days is one year — a prolonged low-vol regime could reduce edge
- **Commission impact:** 55–66 trades/day × $4.12 round-trip commissions adds up; R-multiples must stay positive net of costs
- **Technology risk:** unmanaged orders in NinjaTrader need robust connection handling

## Recommended next steps (from the doc)

1. Paper trade `WickFade5mV1.cs` on NinjaTrader for 2–4 weeks to validate fill quality
2. Add 1–2pt slippage simulation to the backtest engine
3. Test on MNQ for lower capital requirements
4. Investigate portfolio combination of WickFade 5m + (anything bar-sim-safe)
5. Build real-time monitoring dashboard

## Cross-references

- [[wick-fade]] — concept page for the wick-fade family
- [[15s-wick-fade]] — sibling strategy at 15-second timeframe, different entry definition
- [[bar-sim-trailing-bug]] — the bug this doc identified before the BOS_FVG audit did
- [[bos-fvg-failure-consolidated]] — the audit that invalidates this doc's BOS_FVG cross-reference
- [[be-trail-mechanism]] — B/E stops kill WickFade edge too (Option C result)
- [[sweep-cluster]] — the cluster this doc was stranded in
- [[sf-portfolio-cluster]] — another Layer 2 track that uses fixed targets and is comparatively clean
- [[research-arc-map]] — Layer 2 clean, one of the few defensible tracks
- [[wickfade-strategy-findings]] — earlier findings doc; provenance on which optimization ideas came first (trail-on-both-legs was the biggest improvement).
