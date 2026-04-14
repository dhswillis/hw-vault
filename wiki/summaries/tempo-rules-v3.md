---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
related:
  - wiki/summaries/unified-brain-architecture
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/bpr.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/entities/tempo-methodology.md
  - wiki/summaries/tempo-cluster.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, tempo, canonical, methodology]
---

# Tempo Rules V3 Complete — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/tempo_rules_v3_complete.md` — **654 lines, 257,335 words extracted from 281 Discord trade recap transcriptions** spanning 2024-10-02 → 2026-01-29. This is the most authoritative single document on what [[tempo-methodology|Tempo]] actually teaches, because it's built from the live recaps rather than from written rule summaries (which the project state doc explicitly warns are "skeleton only"). V3 supersedes an earlier v2 extract made from 90 transcriptions.

## Methodology arc (five phases)

The doc traces the methodology's evolution:

1. **Foundation (Oct 2024 – Jan 2025)** — IFVG discovery, basic sweep framework
2. **Integration (Feb – May 2025)** — session rules, soft stops default, hard stops deprecated ("market makers hunt hard stops")
3. **Adaptation (Jun – Aug 2025)** — DOL hierarchy formalized, data-wick setup refined
4. **Refinement (Sep – Nov 2025)** — [[smt|SMT]] elevated to primary confluence, [[bpr|BPR]] standardized
5. **Mastery (Dec 2025 – Jan 2026)** — 3+ confluence minimum for funded, base-hit mentality, 30+ copy-trading accounts

## Core mechanics (the doc is the definitive source for each)

**Entry** — [[ifvg|IFVG]] + liquidity sweep + [[smt|SMT]] divergence, candle CLOSE through the gap. Use 30-second bars for precise edge detection. Half-position on close, add second half on pullback/retest.

**Gap sizing (NQ)** — 8–15pt optimal, 5–6pt risky but valid with confluence, 20–30pt strong conviction, skip >30pt (stop too wide).

**Confirmation patterns** — V-shaped recovery with instant momentum; "death candles" (80–100pt single candles on NQ, or more typically 46+pt on the minute chart); wick rejection without close past level; three consecutive directional candles after sweep. "Reaction is everything" — the reaction to the sweep matters more than the sweep itself.

**Stops** — Soft default (~90% of trades): exit only on candle CLOSE past stop, never on wick. Hard stops only on funded accounts when away from screen. Hard stops explicitly rejected in Apr 2025 as unsuitable ("get hunted").

**Break-even** — Non-negotiable at **+17pts profit**. Never B/E below +10pts (too tight, gets wicked).

**Session-aware trade management** — see [[ifvg]] for the full table. Key rule: London is 100% runner with B/E at +17; NY AM is 30% at +17 + 70% runner; NY Lunch and PM are 55% at +17 + 35% at +35 + 10% runner.

**Primary window** — 9:30–11:00 AM ET. Standard exit at 11:00 AM. Wait 2–5 min after the open before taking any trade ("first 2–5 minutes after open = too volatile").

**Risk sizing** — 0.4–0.6% per trade = $200–300 on $50k account. This is NOT 1% (that's too aggressive for this methodology, per the FAQ rules). Contract sizing scales with account and volatility: 10–15 micros on $50k, 15–20 on $150k, half in pre-market/risky, quarter in extreme vol.

**Discipline limits** — Max 2 losses/day, then STOP. After daily target hit, lock all funded accounts. "I'm not going to now start spiraling out of control and give back all my weekly profits."

## Reported performance (Tempo's own numbers)

- **88% win rate across 400+ trades** (Nov 2025 – Jan 2026)
- **Average win 30–50 points NQ, average loss 15–20 points**
- **Single day record: $42,000** on 2026-01-29 across 30+ copy-trade accounts
- **Copy-trading scale** — from 5–10 accounts (April 2025) to 30+ accounts (January 2026)

These numbers are self-reported and not independently audited. They are consistent with what Harrison's corrected tick-level IFVG research produces when SMT and reaction-quality filters are applied (83% WR in `TEMPO_IFVG_RESEARCH.md`). They are **not consistent** with any BOS_FVG number anywhere in the mining corpus — see [[bos-fvg-claim-vs-reality]].

## Explicitly-dead patterns called out in the doc

- **Hard stops** — "market makers hunt hard stops"
- **Chasing pre-market** — "too unpredictable until official market open"
- **Trading at all-time highs without sell-side liquidity** — no targets above, shorts extremely dangerous
- **Stacked gaps** — multiple FVGs in the same zone = high failure rate
- **Trend day fading** — "DO NOT try to fade a trend day or catch the reversal"
- **Late runners** — "runners nearly eliminated — base hit mentality" as of Oct 2025
- **Trump/Powell speeches** — explicit avoid rule by Jan 2026
- **Election weeks and tariff days** — manipulation, chop, watch but don't overexpose

## Open questions flagged by the source

1. **Data wick high win rate claim** — Tempo says the 8:30 AM news data-wick setup is "HIGHEST WIN RATE setup type" but there's no dedicated backtest on it. Mentioned Apr 2025 and again Jan 2026 without numbers. Worth a targeted audit.
2. **SMT's exact measurement protocol** — the doc describes SMT as "NQ vs ES divergence at key levels" but the operational definition (what counts as equivalent levels on ES, what time window to compare) is implicit in the recap videos rather than stated algorithmically.
3. **Reaction-quality vs SMT as the real edge** — Tempo calls SMT primary, but Harrison's backtests show SMT alone doesn't improve WR. Only SMT + reaction-quality gets to 83%. Is Tempo's 88% live number driven by the *visual* reaction-quality judgment that isn't codifiable?

## Cross-references

- [[ifvg]] — entry signal concept page
- [[smt]] — the confluence primary since Nov 2025
- [[dol-framework]] — liquidity target hierarchy
- [[bpr]] — balanced price range
- [[setup-quality-grading]] — A+/A/B+/B tier system, 3+ confluence minimum
- [[tempo-methodology]] — overall entity
- [[tempo-three-layers]] — how this canonical Layer 1 source relates to the mining Layer 2 and implementation Layer 3
- [[bos-fvg-claim-vs-reality]] — why BOS_FVG in the mining corpus is NOT this strategy
- [[tempo-cluster]] — master cluster summary tying all Tempo raw-sources together

## Source documents

- [[raw-sources/trading/tempo/tempo_rules_v2_complete]]
- [[raw-sources/trading/tempo/tempo_rules_v3_complete]]
