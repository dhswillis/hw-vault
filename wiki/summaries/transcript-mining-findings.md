---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/context-engine/TRANSCRIPT_MINING_FINDINGS.md
related:
  - wiki/concepts/volume-profile.md
  - wiki/concepts/ifvg.md
  - wiki/entities/tempo-methodology.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/tempo-cluster.md
tags: [trading, orderflow, methodology, background]
---

# Transcript Mining Findings — Summary

## What the source is

`~/HW/raw-sources/trading/context-engine/TRANSCRIPT_MINING_FINDINGS.md` — **556 lines**. A synthesis of orderflow-trading methodology extracted from transcripts of **7 educators across 30 transcripts** (not Tempo specifically — a broader survey of the orderflow/ICT/SMC universe). Written to identify *codeable rules* that could be extracted from educator transcripts and turned into backtestable signals.

This doc is "background material" — it predates both the V10 cleanup and the Unified Brain pivot. Its value is in documenting the *ideas* being mined, regardless of whether the resulting backtests worked.

## The core rules extracted

### CLC — Context, Location, Confirmation

The organizing framework of the doc. Every trade should satisfy three layers:

1. **Context** — what regime is the market in? TREND, RANGE, EXPANSION, COMPRESSION, NEWS-SHOCK (this is the basis for [[session-type-taxonomy]])
2. **Location** — is the trade happening at a structural level? (key levels, session H/L, FVG, order block, liquidity pool)
3. **Confirmation** — is there a pattern confirming the intended direction? (rejection wick, death candle, three consecutive candles)

All three must be present. Missing any one is "not a trade" territory regardless of how appealing the other two look.

### Volume cluster pullbacks (VCP)

After a strong directional move, price pulls back and tests the **high-volume node** (HVN) that formed during the move. The theory: the HVN represents where the move paused for absorption, and a pullback that tests that node and reacts produces a strong continuation entry. This overlaps with standard [[volume-profile]] analysis but frames it in terms of a specific intra-session pattern rather than profile geometry.

### POC reversal (fail on one side → trade opposite)

Price approaches the **Point of Control** (session POC) from one side, fails to break through, reverses. Trade in the direction of the reversal. The POC acts as magnetic support/resistance — strong failures at POC produce reliable opposite-direction moves.

### Entry-in-the-now

The doctrine that **you do not plan entries in advance** — you watch price action at the level and commit when the confirmation prints. Planning entries (e.g. "if price reaches X I'll enter") leads to taking entries on paper that wouldn't have been taken in real time because the reaction quality was weak.

### Footprint absorption

Heavy volume at a specific price without further movement → absorption (the opposite side is soaking up the pressure). Typically precedes a reversal. This was the concept that [[look-ahead-audit|Bug 2 in the foundation audit]] identified as implemented incorrectly (the absorption detector was reading ahead into uncompleted bar data). After the fix, V8-era absorption-as-filter was "too rare for meaningful signal."

### Rejection / confirmation patterns

- **Wick rejection** — long wick into a level with close back inside. Strong rejection.
- **Death candle** — single 3+ point (or 46+ point in Tempo's version) candle through a level with zero retrace. Strong momentum confirmation.
- **Three consecutive directional candles** — momentum filter after a sweep. Used by Tempo as part of reaction-quality in the [[smt|SMT + reaction-quality]] formula.

### Session bias rules

- **London** sets the initial bias for the day
- **NY AM** is the main profit window
- **NY Lunch** is low-participation and chop-prone
- **NY PM** is for profit-taking and positioning for close, not initiation
- **Overnight Asia** is for scalping news-driven moves

These map closely to the session rules in [[tempo-rules-v3|Tempo v3]] and to the session observations in [[nq-playbook|NQ Playbook]] — educator consensus is high on session taxonomy even when individual setups differ.

## Cross-educator patterns

The doc notes convergent rules across the 7 sources:

- All 7 use some form of FVG or imbalance zone
- 6/7 use liquidity sweeps as entry triggers
- 5/7 use "soft stop" mechanics (close-based exits)
- 7/7 emphasize session timing
- 6/7 use higher-timeframe bias as a gating filter
- 4/7 use volume profile elements
- 3/7 explicitly use footprint data

The convergence suggests the ideas are real orderflow structure — not just one educator's eccentricity — but the magnitude of the edge is open.

## What this doc is NOT

**It is not validated research.** It's a taxonomy of ideas drawn from educators who make self-interested claims. Nothing in this doc has been statistically validated at the level of [[bos-fvg-10pt-audit|BOS_FVG 10+pt]] or even [[tempo-ifvg-research|TEMPO_IFVG_RESEARCH]]. It's the *hypothesis generation* layer — useful as input to structured backtests but not as output.

## Relationship to the Tempo cluster

Tempo is *one* of the 7 educators surveyed here. [[tempo-rules-v3|Tempo's specific methodology]] (IFVG + SMT + sweep + close-through entry + session-aware TPs) is a specific instantiation of the CLC pattern — context (session), location (FVG at a swept level), confirmation (SMT + reaction quality). The transcript mining doc is broader; it surveys the orderflow universe to identify *shared patterns*, while [[tempo-cluster]] narrows to Tempo specifically.

If a future project wanted to build "orderflow strategy from scratch but not committed to Tempo specifically," the transcript mining doc is the starting inventory.

## Cross-references

- [[tempo-rules-v3]] — Tempo's specific instantiation of these patterns
- [[tempo-methodology]] — entity page
- [[ifvg]] — IFVG is the shared primitive most educators use
- [[volume-profile]] — VCP and POC reversal map onto standard volume profile
- [[session-type-taxonomy]] — CLC's "Context" layer maps onto this taxonomy
- [[look-ahead-audit]] — the footprint absorption bug fixed in 2026-02
