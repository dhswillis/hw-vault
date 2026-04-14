---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/smt.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/summaries/tempo-rules-v3.md
tags: [trading, tempo, structure]
---

# BPR — Balanced Price Range

"FVG on steroids." A [[tempo-methodology|Tempo]] structural concept recognized as standard methodology by November 2025.

## Definition

A **BPR** is the overlapping zone between a previous bullish FVG and a previous bearish FVG in the same price region. Because both imbalances are present, the zone acts simultaneously as support and resistance — price tends to reject crisply in either direction when it re-enters the range, which is what "balanced" refers to.

Visually: stack a bullish FVG and a bearish FVG on the same chart, find the price interval where they overlap, and that interval is the BPR.

## Why it's stronger than a regular FVG

A single FVG reflects one-sided pressure — buyers (bullish FVG) or sellers (bearish FVG) moved price aggressively enough to leave a gap. A BPR reflects *both* sides having had aggressive moves that crossed the same level. When price returns to the BPR, whichever side is currently in control encounters the unfilled imbalance from the opposite side, and the rejection tends to be sharper and more reliable than at a plain FVG.

`tempo_rules_v3_complete.md` describes BPRs as more powerful than regular FVGs and explicitly elevates them as a standard tool in the entry stack.

## How it's used in setup grading

BPRs are a confluence factor in the [[setup-quality-grading]] system. The A+ formula is:

**Major sweep + SMT + singular IFVG 8–15pts + daily bias alignment + 50% rejection with wick + clean momentum + BPR support + session level alignment + multiple HTF confluences**

BPR is listed alongside [[smt|SMT]] and HTF alignment as one of the top-quality confluence factors. BPR + SMT + sweep is explicitly called out as highest-conviction.

## Failure mode: tight BPR without SMT

Tempo's anti-pattern list: **avoid going long or short into tight BPRs without SMT.** A BPR that looks like a clean double-rejection zone can trap you if the SMT doesn't confirm — the "balanced" appearance is structural, but the actual move through will be fast, and you'll get stopped with no reversal to catch. The fix is always the same: require SMT to confirm any BPR entry.

## What BPR is NOT

- Not an FVG itself — it's the *overlap* of two FVGs of opposite direction
- Not a sweep zone — BPRs are structural; sweeps happen at liquidity, not necessarily at BPRs
- Not a stop level — stops for BPR-origin trades still go at the [[ifvg|IFVG]] edge (soft stop), per Tempo's standard rule

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/trading-system/batch-runner/SYSTEM_STATE|SYSTEM STATE]]
- [[raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_from_videos|tempo rules from videos]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/docs/brains/TEMPO|TEMPO]]
