---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
related:
  - wiki/entities/tempo-methodology.md
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/tempo-cluster.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, tempo, project-state]
---

# Tempo Project State — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md` — **543 lines**, dated 2026-02-09. The project's "read this first" master context doc. Its job is to orient a new Claude session to the project before any work happens: what's known, what's unknown, what the architecture actually is, and what the hard constraints are. Originally written by Harrison and iteratively updated.

## The four warnings at the top

`TEMPO_PROJECT_STATE.md` leads with a set of non-negotiable corrections to common misreadings:

1. **Do NOT invent rules.** Only use what [[tempo-methodology|Tempo]] actually teaches. If it's not in the transcribed recap corpus or the education videos, it's not a rule.
2. **Follow Option 4 architecture** — Claude Code → Custom API Wrapper → OpenClaw → Quantower. NOT a monolithic QuantConnect algorithm. Early architecture attempts went monolithic and blew up.
3. **Real rules live in the 324 trade recap videos** — 281 transcribed as of Feb 2026, 43 remaining. The written 23-rule skeleton that predated v3 is "skeleton only" and does not carry the nuance.
4. **SMT = Smart Money Technique** (NQ vs ES divergence) — NOT "Supply Manipulation Test." This misreading appears in early notes and is explicitly corrected.

Additional standing corrections:
- **Risk is 0.4–0.6%** per trade ($200–300 on $50k). NOT 1%.
- **Copy-trading 30+ accounts** as of Jan 2026, up from 5–10 in April 2025.

## Source corpus inventory

As of 2026-02-09:

- **324 trade recap videos** (Oct 2, 2024 – Jan 29, 2026)
  - 281 transcribed via Twelve Labs + Whisper pipeline → `tempo_rules_v3_complete.md`
  - 43 remaining untranscribed
- **42 premium education videos** in the Discord education channel
- **18 YouTube classes** — public, lower-signal than the Discord recaps
- **23 written text rules** — the earlier "skeleton" extract, now superseded by v3

## Architecture (Option 4 — current)

```
Claude Code (research + strategy)
   ↓
Custom API Wrapper (tool interface)
   ↓
OpenClaw (execution agent)
   ↓
Quantower (broker adapter)
   ↓
Funded prop-firm accounts (30+)
```

The project state doc warns against collapsing these layers. A monolithic QC algo ("V1") was tried and blew up −92% in the 2022 bear market; "V2" added circuit breakers that fired permanently. Lessons learned:
- Circuit breakers must have recovery rules, not permanent lockouts
- Position sizing must be dynamic by market regime
- Signal confidence is not the same as execution quality

## The V1/V2 history (for context, not guidance)

- **V1** — monolithic QuantConnect algorithm, blew up −92% in 2022 bear market
- **V2** — added circuit breakers, but they fired permanently and locked the algo
- **Current** — Option 4 modular architecture, Claude-driven research, OpenClaw execution, Quantower brokerage

## What the doc says is known vs unknown

**Known (Feb 2026):**
- The full 5-phase methodology evolution from `tempo_rules_v3_complete.md`
- Session-aware TP rules
- Soft stops as default
- SMT as primary confluence
- 0.4–0.6% risk sizing, 3+ confluence minimum on funded

**Unknown / needing work (Feb 2026):**
- 43 untranscribed videos (potentially contain methodology tweaks not yet captured)
- The exact SMT measurement protocol (what counts as equivalent NQ/ES structural levels)
- Reaction-quality codification (what exactly constitutes a "death candle" quantitatively)
- How session-aware TP rules interact with small-gap trades (the London problem that later showed up in [[tempo-v14-corrections]])

## Cross-references

- [[tempo-methodology]] — entity page built partly from this source
- [[tempo-rules-v3]] — the v3 transcript corpus this doc indexes
- [[tempo-cluster]] — master cluster summary
- [[tempo-three-layers]] — how this project state doc fits into the three-layer framework (it's the closest thing to a Layer 1 / Layer 3 bridge — it describes canonical methodology and implementation architecture in one place)
- [[tempo-trading-system]] — the Mac-side directory this doc orients new sessions toward

## Source documents

- [[raw-sources/trading/tempo/TEMPO_PROJECT_STATE]]
