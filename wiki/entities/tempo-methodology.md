---
created: 2026-04-11
updated: 2026-04-11
type: entity
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/bpr.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/tempo-cluster.md
  - wiki/syntheses/tempo-three-layers.md
  - wiki/entities/tempo-trading-system.md
tags: [trading, tempo, entity, methodology]
---

# Tempo Methodology

**Tempo** is the trading educator whose methodology Harrison subscribes to and whose rules are the canonical source for the [[tempo-trading-system|Tempo NQ system]] at `~/Documents/trading-system/`. Harrison is a *subscriber*, not the educator — this is explicitly flagged in `~/Documents/trading-system/CLAUDE.md`.

## What Tempo is

An NQ (E-mini Nasdaq) day-trading methodology built on:
- **[[ifvg|IFVG]]** — Inverted Fair Value Gap entries
- **[[dol-framework|Draw on Liquidity]]** targets
- **[[smt|SMT]]** — intermarket divergence filter (primary since Nov 2025)
- **[[bpr|BPR]]** — balanced price range confluence
- **[[setup-quality-grading|A+/A/B+/B tiers]]** — 3+ confluence minimum on funded accounts
- **Soft stops** — close past gap edge, never wick-touch
- **Session-aware trim rules** — London 100% runner, NY AM 30/70, Lunch+PM 55/35/10
- **0.4–0.6% per-trade risk** — $200–300 on $50k accounts
- **Copy-trading at scale** — 30+ prop firm accounts by January 2026

## The five-phase evolution

`tempo_rules_v3_complete.md` traces the methodology across five phases from October 2024 through January 2026:

1. **Foundation (Oct 2024 – Jan 2025)** — IFVG discovery, basic sweep framework
2. **Integration (Feb – May 2025)** — session rules established, soft stops default, hard stops deprecated
3. **Adaptation (Jun – Aug 2025)** — DOL hierarchy formalized, data-wick setup refined
4. **Refinement (Sep – Nov 2025)** — SMT elevated to primary, BPR recognized as standard
5. **Mastery (Dec 2025 – Jan 2026)** — 3+ confluence minimum, base-hit mentality, copy-trading at 30+ accounts

## Reported performance

From `tempo_rules_v3_complete.md` (Tempo's own statements, not Harrison's backtests):

- **88% win rate across 400+ trades** (Nov 2025 – Jan 2026)
- Average win **30–50 points NQ**; average loss **15–20 points**
- Single-day record: **$42,000** on 2026-01-29 (across 30+ copy-trade accounts)
- Primary session window: **9:30–11:00 AM ET**; standard exit at 11:00 AM
- Copy-trading scaled from 5–10 accounts (April 2025) to 30+ accounts (January 2026)

These numbers are Tempo-reported and unaudited. When Harrison's own backtesting machinery has been pointed at the methodology under tight methodology (`TEMPO_IFVG_RESEARCH.md`, Tempo V2 corrected per [[tempo-v14-corrections]]), the US-session backtest results come in at **70.5% WR** with no SMT filter and **83% WR** with SMT + reaction-quality. These are lower than Tempo's live claim but high enough to be consistent with the rest of the system's discipline layer (SMT gate, setup grading, funded account locks).

## Source corpus (324 videos → 281 transcribed)

Tempo teaches primarily through Discord trade recap videos. Harrison's project state (`TEMPO_PROJECT_STATE.md`, 2026-02-09) inventories:

- **324 trade recap videos** totaling hundreds of hours
- **281 transcribed** (257,335 words) into `tempo_rules_v3_complete.md` and the earlier v2 doc
- **43 remaining untranscribed** as of Feb 2026
- **42 premium education videos** (not trade recaps) in the Discord education channel
- **18 YouTube classes** — public, lower-signal
- Transcription pipeline: **Twelve Labs + Whisper**

The transcribed recap corpus is the authoritative source. Written rule extracts (like the 23-rule skeleton earlier in the project) are "skeleton only" — they don't carry the nuance of the live explanations. `TEMPO_PROJECT_STATE.md` has an explicit rule: "Do NOT invent rules. Only use what Tempo actually teaches."

## Distinction from [[bos-fvg|BOS_FVG]]

BOS_FVG is Harrison's own mining-era interpretation and does NOT match Tempo's IFVG. See [[bos-fvg-claim-vs-reality]] for the reconciliation. The distinction matters because every piece of mining research on BOS_FVG with a "Tempo" label in it (build specs, optimization reports, portfolio audits) is **not testing what Tempo actually teaches** — it's testing a momentum-continuation strategy that happens to use FVG terminology.

## Cross-reference

- Methodology rules: `~/HW/raw-sources/trading/tempo/tempo_rules_v3_complete.md` (654 lines, 281 recaps)
- Project state: `~/HW/raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md` (543 lines)
- V14 implementation audit: [[tempo-v14-corrections]]
- V2 mining rerun on corrected logic: `~/HW/raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md`
