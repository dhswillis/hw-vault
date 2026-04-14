---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
related:
  - wiki/concepts/tempo-v14-corrections.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/tempo-cluster.md
  - wiki/entities/tempo-methodology.md
tags: [trading, tempo, implementation, bug, ninjatrader]
---

# Tempo V14 Corrections — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md` — **385 lines**, dated 2026-03-20. An audit of the v14 C# NinjaTrader implementation of the Tempo IFVG strategy. Identified four critical implementation bugs where the C# code was doing a *different* strategy than the one described in `tempo_rules_v3_complete.md`, and documented the fixes and a 30-day corrected backtest.

## The four bugs (high-level; see [[tempo-v14-corrections]] concept page for the full explanation)

1. **Signal sequence reversed** — v14 waited for a sweep then scanned for a new FVG. Correct: FVG forms before the sweep (momentum gap), sweep is the catalyst, entry is on price trading back through the *pre-existing* FVG.
2. **Stop placement wrong** — v14 used sweep-wick ± 1pt (20–30pt stops on NQ). Correct: opposite FVG edge (= gap size, typically 5–15pt). Sweep wick is the *emergency* hard stop only.
3. **Entry price wrong** — v14 entered at bar close when the close crossed back through the FVG. On 1-minute bars, closes landed 13+ points past the FVG edge. Correct: enter at the FVG edge exactly, using 30-second bars.
4. **Emergency stop on wrong side for fades** — for 30 fade-short trades, v14's emergency hard stop was placed *above* entry (profit side), effectively disabling it. Fixed with `max(wick+1, entry+3)` for shorts, `min(wick−1, entry−3)` for longs.

## 30-day corrected backtest (2025-03-03 → 2025-04-04)

Using Databento NQ tick data, 30 trading days, post-corrections:

| Slice | Trades | WR | Pts/day | Calmar |
|---|---|---|---|---|
| **US sessions (NY AM/Lunch/PM, with M60 bias filter)** | 105 | **70.5%** | **+24.5** | **5.7** |
| London only | 58 | 46.6% | +10.9 | — |
| **Combined (US + London)** | **163** | **62.0%** | **+31.9** | **4.8** |

## The London problem

London's 46.6% WR hides a structural failure: **247% of London's total PnL came from 3 end-of-day runner trades.** Non-EOD London runs at **−12.7 pts/day** — the session is deeply negative when you strip out the session-end flatten. The audit diagnoses this as the interaction between:

- London's "100% runner to B/E at +17" session rule from `tempo_rules_v3_complete.md`
- Small typical gap sizes during London (5–7pt), which produce soft stops that catch every trade before it can run to +17

Practical recommendation from the audit: **skip London in production**, or re-engineer the session rule to accept smaller targets that match the structural room available.

## The Lumi problem

The v14 implementation also shipped a second strategy — the **Lumi engine** (sweep → order block → FVG cascade, spec from @LumiTraders on X/Twitter). v14 audit results on Lumi:

- 35 trades over 29 days
- 20% WR
- −3.4 pts/day
- Calmar −0.4

Lumi targets 3R and needs ~25% WR for break-even. At 20% it's slightly below. Only HTF_OB sources and the London session showed any small edge; all other sources negative. The audit's Twitter-spec expected ~65% WR on ~200 backtested trades — the gap is likely from bar construction differences (NinjaTrader bars vs Python tick resampling), overfitting, or regime change.

**Recommendation from audit: exclude Lumi from production, or test London-only / HTF_OB-only subsets in isolation.**

Adding Lumi to the IFVG portfolio hurts it: IFVG alone = +26.4 pts/day, IFVG + Lumi = +23.0 pts/day.

## Status as of 2026-04-10

The v14 corrections are a historical artifact. Harrison's `CLAUDE.md` at `~/Documents/trading-system/` states that **v24 (IFVG MTF cascade) is the current active sim strategy**, not v14. v14's corrected numbers (70.5% WR US, Calmar 5.7) are not being pursued as a production target — they're documentation of what the code was doing wrong and what the correction produced in-sample. The active question is what v24 produces in forward sim.

## Why these numbers are not suspect in the same way as BOS_FVG

The v14 corrected backtest uses **close-based soft stops** (candle close past the opposite FVG edge), not trailing stops. This makes it bar-sim-safe by construction — the bar's close is unambiguous regardless of intra-bar path. The [[bar-sim-trailing-bug]] that invalidates every BOS_FVG trailing number does NOT apply here.

However, the in-sample caveat still holds: 30 days is a small sample, no walk-forward OOS validation, and no Bonferroni correction across the variations tried. Treat the +24.5 pts/day US-session number as "the fix worked, this is roughly what the corrected spec produces," not as "this is the expected live edge."

## Cross-references

- [[tempo-v14-corrections]] — concept page with full technical breakdown
- [[ifvg]] — the signal the corrections re-align v14 with
- [[bar-sim-trailing-bug]] — why these numbers are not suspect the way BOS_FVG's are
- [[tempo-ifvg-research]] — the companion research doc that runs the corrected spec with SMT and reaction-quality filters (83% WR)
- [[tempo-three-layers]] — Layer 3 context for v14's role in the overall arc

## Source documents

- [[raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS]]
