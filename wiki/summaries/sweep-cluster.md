---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md
  - raw-sources/trading/sweep/BIG_RIDE_RESEARCH.md
  - raw-sources/trading/sweep/DAILY_RANGE_RESEARCH.md
  - raw-sources/trading/sweep/DOJI_BREAK_STRATEGY.md
  - raw-sources/trading/sweep/PORTFOLIO_AUDIT_V2.md
  - raw-sources/trading/sweep/TEMPO_PORTFOLIO_BUILD_SPEC.md
  - raw-sources/trading/sweep/15S_Wick_Fade_Findings.docx
  - raw-sources/trading/sweep/WickFade_Complete_Findings.docx
related:
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/bos-fvg.md
  - wiki/summaries/nq-playbook.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/summaries/sf-portfolio-cluster.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, sweep, cluster-summary, suspect-results]
---

# Sweep Cluster — Summary

## What the cluster is

`~/HW/raw-sources/trading/sweep/` — **8 documents** covering the various "sweep-and-fade" strategy variants tested over 2025–2026: the wide sweep, the big-ride continuation, daily range sweeps, doji breaks, the IFVG-style sweep fade, and the wick fade scalper. This cluster overlaps with both [[sf-portfolio-cluster]] (which includes sweep-fail as a signal family) and the Tempo IFVG research (which uses sweep as the catalyst for a liquidity fade), but the sweep cluster specifically focuses on different entry mechanics than either — it's the experimental ground for "given a sweep happened, what's the best way to trade the reaction?"

## The cluster files

| File | Lines | Disposition |
|---|---|---|
| `SWEEP_FADE_RESEARCH.md` | 618 | Primary sweep fade research. Headline: "+0.29 R/day, marginal" — see RESEARCH_LOG_20260304 for the dead-end verdict |
| `BIG_RIDE_RESEARCH.md` | 322 | Continuation after sweep reversal fails. Tests "ride the big move" theory |
| `DAILY_RANGE_RESEARCH.md` | 285 | Sweeps within the daily range vs sweeps of daily range. Range-bound vs trending regimes |
| `DOJI_BREAK_STRATEGY.md` | 300 | Doji reversal after sweep. **Already confirmed dead** in auto-memory (phantom B/E + bad tick data created false edge; clean sim −0.171 avgR) |
| `PORTFOLIO_AUDIT_V2.md` | 307 | Second-pass portfolio audit. Overlaps with SF portfolio audit work |
| `TEMPO_PORTFOLIO_BUILD_SPEC.md` | 329 | Build spec for a Tempo-style portfolio using sweep signals |
| `15S_Wick_Fade_Findings.docx` | (binary) | 15-second wick fade scalper findings. Requires `textutil` |
| `WickFade_Complete_Findings.docx` | (binary) | Complete wick-fade research. Requires `textutil` |

## The pattern this cluster is trying to capture

All of these are variations on the same core idea: **price sweeps a liquidity level, then reverses.** The variants differ in:

- **What kind of sweep** — session H/L, daily range, swing points, psychological levels, 1H bar H/L
- **What kind of reaction** — wick rejection (wick fade), doji (doji break), momentum through (big ride), fade back to structure (sweep fade)
- **What timeframe** — 15-second (wick fade), 1-minute (most), 5-minute (big ride), 15-minute (sweep-fail)
- **What exit** — trailing, fixed R target, session flatten, close-through

The core problem with the cluster as a whole is that **most variants use trailing stops or close-based management on bar data**, which means the [[bar-sim-trailing-bug]] caveat applies to anything with a trailing exit. And because the cluster is experimentally dense (dozens of variants), it's exactly the kind of multiple-comparison environment where Bonferroni correction is essential — and none of these docs apply it rigorously.

## Known dead within the cluster

- **Doji Break** — phantom B/E fills + bad tick data (MNQ-bleed) created a false edge. Clean sim: **−0.171 avgR everywhere**. Bug fix: symbol filter + B/E fix. Already in auto-memory as DEAD.
- **Wide Structure Sweep** (from `RESEARCH_LOG_20260304.md`) — **+0.29 R/day**, marginal. Not dead but not tradeable.
- **Wick Reversion Scalper** — DEAD per RESEARCH_LOG_20260304
- **Candle Fade** — DEAD per RESEARCH_LOG_20260304
- **9:00 AM Candle Behavior** — **+0.04 R/day max**, not tradeable
- **Open Fakeout** — DEAD per RESEARCH_LOG_20260304

## Big Ride Research (TBD)

`BIG_RIDE_RESEARCH.md` tests the opposite pattern: **what if you ride the continuation after a sweep fails to reverse?** The hypothesis is that a failed sweep-fade creates a trapped-short (or trapped-long) population that fuels the continuation. Results need to be read from the source directly — the short summary here is that this is an *opposite-direction* variant of the fade, and the meta-question is whether the two together form a coin-flip (one wins when the other loses) or whether they independently each don't work.

## Daily Range Research (TBD)

`DAILY_RANGE_RESEARCH.md` asks whether the daily range (high/low of the previous day, current day, or a rolling window) is a valid sweep target. Conceptually sound, but whether it produces edge depends on whether the sweeps of daily range are clustered with other confluence (DOL stack, SMT, etc.) — and the cluster's research didn't typically include those filters.

## Wick Fade (15S scalper)

The two binary files — `15S_Wick_Fade_Findings.docx` and `WickFade_Complete_Findings.docx` — document the 15-second wick fade scalper. This is a particularly suspicious variant for the [[bar-sim-trailing-bug]] reason: **15-second bars compress even less path than 1-minute bars, and the scalper has very tight risk, so intra-bar path ordering matters a lot**. If the wick fade's claimed edge is from a trailing stop on 15S bars, it's almost certainly contaminated. If it's from a fixed-R target with exits on the same 15S bar as the entry, that's structurally more defensible but still needs Bonferroni-style review on the parameter sweep.

These two docs need `textutil -convert txt` to become readable. Until then, treat wick fade claims as "unread, not ingested."

## Tempo Portfolio Build Spec

`TEMPO_PORTFOLIO_BUILD_SPEC.md` is a spec for combining several of the cluster's variants into a Tempo-style portfolio. It predates the [[tempo-v14-corrections|v14 corrections]] and therefore may be based on an incorrect signal understanding — the same pattern that the BOS_FVG mining corpus fell into. Read with the three-layer framework in mind and verify that the sweep mechanics being specified match Tempo's canonical methodology before citing.

## Open questions / what to read next

1. **Sweep Fade "real" number** — is the +0.29 R/day figure in SWEEP_FADE_RESEARCH.md robust across parameter sweeps, or does Bonferroni kill it?
2. **Big Ride vs Sweep Fade correlation** — are these genuinely anti-correlated (one wins when the other loses) or just independently bad?
3. **Wick Fade contents** — what's actually in the two docx files? Is this a 15S trailing bug or a 15S fixed-target strategy?
4. **Tempo Portfolio Build Spec vs v14 corrections** — does the build spec match the corrected mechanics, or does it need an update pass?

## Cross-references

- [[bar-sim-trailing-bug]] — applies to any variant using trailing exits
- [[sf-portfolio-cluster]] — sibling cluster; SF uses sweep-fail signals with fixed targets and is comparatively cleaner
- [[ifvg]] — the Tempo canonical version of a sweep-reaction strategy (fade into close-through FVG)
- [[nq-playbook]] — the V10 flagship consolidation that references several of these variants
- [[tempo-three-layers]] — Layer 2 framework; most of this cluster is Layer 2 suspect
