---
created: 2026-04-11
updated: 2026-04-11
type: synthesis
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
  - raw-sources/trading/COMPREHENSIVE_MINING_REPORT.md
  - ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md
  - raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/summaries/tempo-rules-v3.md
tags: [trading, synthesis, bos-fvg, ifvg, dead-strategy, canonical]
---

# BOS_FVG Claim vs Reality

**Thesis:** BOS_FVG and Tempo's IFVG are **not the same signal**, were never the same signal, and the mining research corpus that labeled BOS_FVG as "the core signal of the Tempo system" was conflating two structurally opposite trade mechanics. When the two are measured side-by-side on the same 312-day Databento NQ dataset, BOS_FVG at 47.7% WR matches none of Tempo's published numbers, while corrected Tempo IFVG at 70.5–83% WR maps cleanly onto Tempo's self-reported 88% live WR. The gap is not a calibration issue; it's two different strategies.

## The concrete divergence

From the research journal (`TEMPO_IFVG_RESEARCH.md`) directly comparing both:

| Dimension | BOS_FVG (Harrison's mining) | Tempo IFVG (canonical) |
|---|---|---|
| **Reported WR** | 47.7% (bar-sim) / ~50% (tick) | 74.7% without SMT, 83% with SMT + reaction |
| **Entry direction** | *With* the break (momentum continuation) | *Against* the sweep (liquidity fade) |
| **Entry trigger** | Price fills the FVG (limit at the far edge of an FVG that formed in the direction of the BOS) | Price trades back *through* a pre-existing FVG after a liquidity sweep |
| **FVG timing** | FVG forms *after* the break of structure | FVG forms *before* the sweep; the sweep inverts it |
| **Stop** | FVG edge, fixed risk in points | FVG edge, but *soft* — close-through only, checked on bar closes |
| **Target mechanism** | Trailing stop (bar-sim broken — see [[bar-sim-trailing-bug]]) | Session-aware TPs: London 100% runner, NY 30/70 or 55/35/10 |
| **Trades/day** | 37–75 | 0.9–2 (A+ setups) |
| **Directional bias** | Follows break direction | Fades sweep direction |
| **Risk sizing** | 1R position, no confluence gate | 0.4–0.6% with 3+ confluences required (funded) |

These are not variations of one strategy. They are opposite structural plays.

## The "core signal of Tempo" mislabeling

Harrison's `COMPREHENSIVE_MINING_REPORT.md` V2 explicitly identifies BOS_FVG as "the best signal in the dataset by total R and walk-forward stability" and frames it as the core of the Tempo system. This framing was then inherited by:

- `BOS_FVG_RESEARCH.md` (+13.37 R/day, Calmar 151)
- `clean_backtest.py` results (+0.895 avgR, Calmar 237)
- V10z 15S trailing claims (+1.141 avgR, Calmar 263.6)
- The original HW vault `bos-fvg.md` (pre-2026-04-10)
- NinjaTrader build specs and portfolio compositions

But **at no point did Harrison's mining pipeline actually run Tempo's IFVG mechanics.** The mining corpus used:
- BOS (break of structure in the direction of the trade)
- Fill-the-FVG entry (limit at the far edge)
- Trailing stop (not close-past soft stop)

Tempo's IFVG uses the opposite setup:
- Sweep (not BOS — a sweep is a *failure* of structure, taking out a liquidity pool before reversing)
- Close-through-FVG entry (not fill-the-FVG)
- Soft stop and session-aware TPs (not trailing)

## How the conflation happened

The naming is part of the problem. "Inverted FVG" sounds like it could describe BOS_FVG — an FVG in the direction of the break, which "inverts" the prior structure. The actual technical meaning in Tempo's teaching is that the *price inverts the FVG*: it trades back through a gap it already formed earlier in the move. The inversion is of the price relationship, not of the structure.

Add to that:
- Both strategies use the term "FVG" with the same three-bar definition
- Both reference "break of structure" in the loose sense (a sweep is also a kind of break)
- Both were being researched by Harrison simultaneously, with shared terminology
- The early mining reports treated "BOS_FVG" as shorthand for "a Tempo-like signal"

The result: hundreds of hours of backtesting work aimed at optimizing a strategy that was never Tempo's. The V2 mining report's 63.5% WR headline is real in the sense that a BOS-direction FVG-fill entry with bar-sim trailing really does backtest to that number on 1-minute OHLC data. It has no relationship to Tempo's live 88% WR — they're measuring different populations of trades.

## Why the backtested BOS_FVG "edge" isn't real either

Even setting aside the name confusion, BOS_FVG as-defined does not have edge. On the same 2026-04-10 audit, a paired bar-vs-tick replay on 60 sampled days showed:

- **Bar-level trailing sim:** +0.3590 avgR, 41% WR, +5.24 R/day
- **Tick-level trailing sim:** **+0.0010 avgR**, 66.9% WR, +0.01 R/day

The +0.35 avgR headline was a [[bar-sim-trailing-bug|bar-reconstruction artifact]]. Ground-truth tick replay says the BOS_FVG trailing configuration is indistinguishable from zero. See [[bos-fvg-failure-consolidated]] for the full paired comparison.

For the static-target variant of BOS_FVG (10+pt FVGs, fixed T3R targets), the tick-level baseline is **+0.015 avgR** with no filter surviving Bonferroni multiple-comparison correction. Also dead — see `BOS_FVG_10PT_AUDIT.md`.

## What this means operationally

1. **Every backtest numbered on BOS_FVG is contaminated**, not by one bug but by two: (a) the signal is not what the project thought it was, and (b) the trailing-stop exit cannot be simulated on bar data. The first bug means the results don't map to Tempo's real strategy; the second means the results don't even map to BOS_FVG-with-trailing as a standalone strategy.
2. **IFVG research has to start from Tempo's rules, not from the mining corpus.** `TEMPO_IFVG_RESEARCH.md` is the correct foundation — it implements the close-through-FVG logic and runs at tick resolution.
3. **The "core signal of Tempo" label has to come off** every build spec, portfolio doc, and HW vault page that still carries it. CLAUDE.md at `~/Documents/trading-system/` has had the corrected position since March 2026; the mining research journal and parts of the context engine are still carrying the old language.
4. **The two strategies are likely uncorrelated, not related.** `TEMPO_IFVG_RESEARCH.md` actually points out that BOS_FVG (momentum continuation) and IFVG (liquidity fade) would combine well in a portfolio *if* BOS_FVG had any edge. It doesn't, so the portfolio argument is moot, but the structural observation stands.

## The one thing BOS_FVG still has going for it

The signal generator (swing detection → BOS ≥ 2pt → FVG within 15 bars → limit at far edge) is clean. It's a well-defined, deterministic, look-ahead-free signal. If a future investigation wanted to test an entirely different trade management layer on top of it — fixed targets only, no trailing, close-based stops — the signal layer could be reused without modification. That's the "signal is fine, the exit mechanism fails" framing that the [[bos-fvg|concept page]] now carries.

## Cross-references

- [[bos-fvg]] — concept page, rewritten as dead-strategy
- [[ifvg]] — the actually-correct Tempo signal
- [[bar-sim-trailing-bug]] — the structural backtest bug that inflated BOS_FVG's trailing results
- [[comprehensive-mining-report-v2]] — the mining report that propagated the "BOS_FVG is Tempo's core signal" framing
- [[bos-fvg-failure-consolidated]] — 2026-04-10 tick audit
- [[tempo-v14-corrections]] — the other side of the coin: when the C# implementation of IFVG got the mechanics wrong and produced a different (also wrong) signal
