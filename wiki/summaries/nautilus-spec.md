---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/context-engine/NAUTILUS_SPEC.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/nq-playbook.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/research-journal.md
tags: [trading, bos-fvg, nautilus, spec, suspect-results]
---

# NautilusTrader BOS_FVG Spec — Summary

> **Layer 2 — suspect.** This is an engineering spec for implementing BOS_FVG with 15-second trailing stops on NautilusTrader. It inherits the [[bar-sim-trailing-bug|bar-sim trailing contamination]] that the 2026-04-10 tick audit identified — the spec describes the strategy that `NQ_PLAYBOOK.md` claimed was validated at +1.141 avgR / Calmar 263.6, and that tick replay later showed to be +0.001 avgR. The spec is historically useful as a precise statement of what BOS_FVG V10z was supposed to do, but it is not a spec for a profitable strategy.

## What the source is

`~/HW/raw-sources/trading/context-engine/NAUTILUS_SPEC.md` — **454 lines**. An engineering specification for running BOS_FVG with 15-second trailing stop management on NautilusTrader (a Python-native event-driven backtest and execution engine). Written during the V10z era, when BOS_FVG was the flagship signal and 15-second trailing was the "breakthrough" exit mechanism.

## What it specifies

**Strategy:** BOS_FVG on NQ/MNQ futures using 15-second bars for trade management.

**Signal mechanics (from spec):**
- Swing detection: 3-bar confirmed swings
- BOS: close past confirmed swing with 2pt minimum displacement
- FVG: 3-bar gap within 15 bars of the BOS in the same direction
- Entry: limit at far edge of FVG (gap_low for longs, gap_high for shorts)
- Stop: opposite edge of FVG (risk = gap size)

**Trade management:**
- 15-second bar granularity throughout
- Trail trigger: 0.5R MFE
- Trail buffer: 0.1R behind high-water mark
- Session cut: 20:00 UTC flatten
- Minimum risk: 1.5pt
- Maximum risk: 50pt

**Data:** 312 days Databento tick data, Feb 2025 → Feb 2026. NautilusTrader ingests the tick data and rebuilds 15S bars internally.

## Why this is worth reading anyway

1. **It's the most precise statement of the V10z strategy anywhere in the corpus.** `NQ_PLAYBOOK.md` describes it prosaically; `RESEARCH_JOURNAL.md` describes it narratively; this spec describes it *operationally*, with parameter values you could type into a strategy class. If a future audit wants to verify exactly what V10z was doing, this is the reference.
2. **NautilusTrader cross-validation was published in the research journal at 9,188 trades, 55.6% WR, +0.286 avgR** — noticeably lower than NQ_PLAYBOOK's Python sim +1.141 avgR. This should have been the red flag that triggered a tick-level re-audit in March; it wasn't until April. The spec is the bridge between the two numbers — once you know the spec, you can see that NautilusTrader was running the same strategy with engine-native execution and producing a different answer.
3. **NautilusTrader's engine-native execution is NOT subject to the bar-sim trailing bug.** The platform processes ticks in order and manages trailing stops tick-by-tick internally. So the +0.286 avgR Nautilus number is *closer to tick-level ground truth* than the Python bar sim's +1.141. However, 55.6% WR and +0.286 avgR is still well above the +0.001 avgR that the April tick audit found — why?

## The Nautilus-vs-tick-audit discrepancy

Two possible explanations for why Nautilus reports +0.286 avgR but my tick audit reports +0.001 avgR on roughly the same strategy:

**A. Different trade populations.** Nautilus used 15-second bars as its primary signal timeframe and had 9,188 trades over 312 days. My tick audit used 1-minute bars for signal detection (with tick-level trade management) and had 842 trades on 60 days (~4,300 extrapolated to 312 days). The tighter 15S signal detection fires on a different (larger) population. Some fraction of that population may be a real edge, diluted by noise in my 1-minute sample.

**B. Entry slippage.** My tick audit used 0.25pt entry slippage. Nautilus may have used tick-exact fills without slippage. On BOS_FVG's typical gap sizes (5–15pt), 0.25pt slippage is 1.5–5% of risk — not a huge effect but non-zero over thousands of trades.

**C. Entry timing.** My tick audit waits for price to reach the FVG edge exactly on tick data. Nautilus enters on the 15S bar where the low crosses the FVG edge (still bar-level detection, even if the execution is tick-level). The entries may be slightly different.

None of these explanations are airtight. The honest answer is: **Nautilus gives a smaller positive result than the Python bar sim but a larger positive result than my 1-minute tick replay, and the gap deserves a follow-up tick-level run using 15S signal detection as a clean comparison.** The April tick audit was not designed to replicate the 15S signal timeframe that V10z used.

## Action items flagged by this summary

1. **Re-run the tick audit with 15-second signal detection** to get an apples-to-apples comparison with Nautilus's +0.286 avgR. If the tick audit still produces near-zero, Nautilus's result is an engine-level artifact. If it produces something closer to Nautilus's number, my earlier 1-minute result is under-counting the signal population and BOS_FVG 15S trailing may have a small residual edge after all.
2. **Cross-check Nautilus execution at the tick level** — does Nautilus actually process each tick, or does it process "events" at bar boundaries? If it's the latter, Nautilus has the same bar-sim bug.
3. **Re-read the 55.6% WR** in context — my 1-minute tick replay gave 66.9% WR on 842 trades. 55.6% on 9,188 trades and 66.9% on 842 trades is actually consistent with "high WR, tiny winners, net marginal" — both are consistent with "trailing stop catches most trades at or near break-even, no real edge."

## What to do with this spec

- **Do not use it as a production spec.** BOS_FVG trailing is dead per the 2026-04-10 audit and CLAUDE.md directive.
- **Do use it as a reference** for what the V10z era was actually doing, and as the starting point for any follow-up tick-level 15S audit.
- **Do flag the Nautilus +0.286 avgR number** as a data point that was inconsistent with the Python sim's +1.141 avgR but wasn't acted on in time.

## Cross-references

- [[bos-fvg]] — concept page (rewritten as dead-strategy)
- [[bar-sim-trailing-bug]] — the bug that the spec is a specimen of
- [[bos-fvg-failure-consolidated]] — the tick audit that closed the book on this strategy family
- [[nq-playbook]] — the doc that headlined V10z as +1.141 avgR / Calmar 263.6
- [[research-journal]] — the narrative doc that called Nautilus "different strategy surface" when it should have been called "red flag"
