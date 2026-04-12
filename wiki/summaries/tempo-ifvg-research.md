---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
related:
  - wiki/summaries/tempo-cluster
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/tempo-v14-corrections.md
  - wiki/concepts/bos-fvg.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/tempo-v14-corrections.md
  - wiki/syntheses/bos-fvg-claim-vs-reality.md
tags: [trading, tempo, ifvg, backtest, smt]
---

# Tempo IFVG Research — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md` — **664 lines**, dated 2026-03-08. The single most important Layer 2 research doc in the Tempo cluster because it is the only one that runs a backtest against **the correct IFVG spec** (close-through-FVG on a corrected signal generator, tick-level fills, soft stops). It implements the mechanics from `tempo_rules_v3_complete.md` rather than Harrison's earlier BOS_FVG mining interpretation, and runs them against 312 days of Databento NQ tick data.

## The headline finding

Baseline IFVG (no SMT, no reaction-quality filter): **74.7% WR on 269 trades (1-minute timeframe)**. This is dramatically better than any BOS_FVG trailing number, and critically it is consistent with Tempo's self-reported 60–65% when SMT is absent. When SMT is added as a filter, the combined result jumps to **83% WR** — matching Tempo's A+ tier reported 80%+.

## The SMT reaction-quality finding

The research hit an unexpected result on SMT as a standalone filter:

- Baseline: 64.4% WR
- With SMT alone: **63.3% WR** (SMT does *not* improve standalone backtest WR)
- With SMT + reaction-quality (death candle ≥3pt OR 3+ consecutive directional candles): **83% WR**

This is an important epistemological moment for the project. Tempo explicitly calls SMT the *primary* confluence since November 2025. The backtest says SMT alone isn't the edge — the edge is SMT *plus* reaction-quality, which together probably encode "this is a real liquidity reversal happening right now" in a way that SMT alone (a structural filter) cannot. The working hypothesis: SMT identifies reversal *candidates*, reaction quality confirms the reversal is *actually occurring*. You need both.

This also explains why the mining corpus (which had no SMT anywhere) never stumbled onto high-WR configurations even when filter sweeps found spurious 80%+ tiers from look-ahead bias. The genuine 80%+ tier needs both SMT-style structural divergence AND reaction-quality momentum — neither of which was being measured.

## BOS_FVG vs IFVG (the explicit comparison)

The doc directly compares BOS_FVG and IFVG on the same dataset:

| Dimension | BOS_FVG | Tempo IFVG |
|---|---|---|
| Win rate | 47.7% | 74.7% (without SMT) |
| Direction | With the break (momentum) | Against the sweep (reversal) |
| Entry mechanism | Limit fill at FVG far edge | Close through a pre-existing FVG |
| Trades / day | 37–75 | 0.9–2 (A+ setups) |
| Edge source | Structural continuation | Liquidity fade |

The doc itself concludes: "These are fundamentally different strategies — BOS_FVG rides momentum after structure breaks, Tempo fades liquidity sweeps. They are likely uncorrelated and could combine well." That last clause is moot given the 2026-04-10 finding that BOS_FVG trailing has no edge at all, but the structural observation — that BOS_FVG and IFVG are different strategies — is what [[bos-fvg-claim-vs-reality]] now anchors on.

## Portfolio blend: IFVG + Lumi

The doc tests an IFVG + Lumi portfolio (the same Lumi that later tested at 20% WR in `TEMPO_V14_CORRECTIONS.md`):

- IFVG: 83% WR
- Lumi: 62% WR (this doc's number; v14 audit later lowered it to 20%)
- Combined: 67.4% WR
- Correlation: −0.004 (near-zero — diversifying)

The negative correlation is real and interesting — IFVG is a fade strategy and Lumi's HTF-OB-driven version can be a momentum strategy, so negative correlation is structurally plausible. But the Lumi number here (62%) disagrees with `TEMPO_V14_CORRECTIONS.md` (20% WR, 35 trades). The research doc's 62% is probably an earlier in-sample measurement that didn't survive the v14 audit's stricter testing.

## Dual-feed requirement (critical infrastructure note)

SMT as implemented in this research requires **simultaneous NQ and ES tick data** — the backtester has to check whether ES swept the structural equivalent of an NQ level in the same time window. Harrison's Databento subscription provides both, but this means any future IFVG research that claims to test SMT must confirm dual-feed data is in play. A single-instrument backtest cannot measure SMT.

## Is this research doc contaminated by the bar-sim trailing bug?

**No.** IFVG uses close-through soft stops, not trailing stops. The [[bar-sim-trailing-bug]] only affects trailing-stop simulations. IFVG's close-based exits are bar-sim-safe by construction because a bar's close is unambiguous regardless of intra-bar path ordering. This is one of the very few Layer 2 research docs that is NOT caught by the 2026-04-10 tick audit.

The in-sample caveats still apply: 269 trades is modest, no walk-forward validation reported, and multiple-comparison correction wasn't explicitly applied to the filter sweep. The 83% WR result is the most internally-consistent Layer 2 finding in the entire Tempo cluster, but it still needs OOS validation before being called "validated."

## Cross-references

- [[ifvg]] — concept page with full mechanics
- [[smt]] — the SMT + reaction-quality insight from this doc
- [[tempo-v14-corrections]] — companion doc that re-tests after the C# spec is fixed
- [[bos-fvg-claim-vs-reality]] — uses this doc's direct BOS_FVG vs IFVG comparison as primary evidence
- [[tempo-rules-v3]] — the Layer 1 methodology this research is implementing
- [[bar-sim-trailing-bug]] — why this doc's results are not invalidated by the 2026-04-10 audit
- [[tempo-three-layers]] — this is the canonical Layer 2 doc, the one that took Layer 1's rules and tested them correctly
