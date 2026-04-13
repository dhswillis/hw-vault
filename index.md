> Maintained by Claude per `CLAUDE.md`.

# Index

Catalog of every wiki page, one line each. Updated on every `ingest` and any time a wiki page is created or renamed.

## Concepts

- [[wiki/concepts/bos-fvg]] — **DEAD (2026-04-10).** BOS_FVG is NOT validated. Bar-sim trailing inflated prior results by +0.29 R/trade; tick replay shows +0.001 avgR.
- [[wiki/concepts/wick-fade]] — **TICK-VALIDATED family.** 5m and 15s fade variants. 10/10 walk-forward OOS positive. Fixed stop + target, bar-sim-safe. Rare Layer 2 survivor.
- [[wiki/concepts/bar-sim-trailing-bug]] — Structural reason trailing stops can't be backtested on 1-minute OHLC. Caused the BOS_FVG illusion. Wick-fade docs flagged the same bug in March.
- [[wiki/concepts/maps-of-content]] — Curated editorial index notes for topic clusters; the Obsidian-community layer on top of Karpathy's flat `index.md`.
- [[wiki/concepts/inbox-processing]] — Morning ritual for converting raw captures into structured wiki notes. Implemented as `/inbox`.
- [[wiki/concepts/invalidation-rules]] — **Canonical.** The "do not forget" ledger. 15 rules distilled from every bug found across V7-V11 research. Read before citing any pre-April 2026 number.
- [[wiki/concepts/ifvg]] — Inverted Fair Value Gap, Tempo's canonical entry signal. Close-through FVG on a pre-existing gap after a sweep.
- [[wiki/concepts/smt]] — Smart Money Technique: NQ vs ES divergence. Primary Tempo confluence since Nov 2025.
- [[wiki/concepts/dol-framework]] — Draw on Liquidity. Hierarchy of structural magnets — London H/L → data wicks → hourly → 50% range → PDH/PDL.
- [[wiki/concepts/bpr]] — Balanced Price Range. Overlap of opposite-direction FVGs; "FVG on steroids."
- [[wiki/concepts/setup-quality-grading]] — A+/A/B+/B tier system. 3+ confluences required on funded accounts (Jan 2026 rule change).
- [[wiki/concepts/tempo-v14-corrections]] — Four bugs in the v14 C# NinjaTrader implementation. Code was running a different strategy than Tempo teaches.
- [[wiki/concepts/mtf-alignment]] — Multi-timeframe alignment. Prior WR claims contaminated by V10i look-ahead; re-test after tick-level rework.
- [[wiki/concepts/be-trail-mechanism]] — Break-even + trailing stop exit. The mechanism whose apparent edge was the [[bar-sim-trailing-bug]] illusion on BOS_FVG.
- [[wiki/concepts/v10i-look-ahead-bug]] — Critical alignment look-ahead that contaminated all V1-era mining research. Fixed in V2 (but V2 still has the bar-sim bug).
- [[wiki/concepts/volume-profile]] — POC, VA, HVN/LVN + footprint analysis. Layer 2/3 of the Context Engine.
- [[wiki/concepts/session-type-taxonomy]] — TREND / RANGE / EXPANSION / COMPRESSION / NEWS-SHOCK classification used by the Context Engine.
- [[wiki/concepts/bayesian-belief-engine]] — Multinomial naive Bayes classifier with checkpoint-driven posterior updates, heart of the Context Engine.
- [[wiki/concepts/vwap-double-counting-bug]] — NinjaTrader `Calculate.OnEachTick` bug that broke live/backtest parity in the V5 SF portfolio.

## Entities

- [[wiki/entities/tempo-methodology]] — Tempo the educator. 281 transcribed recaps, 88% WR on 400+ trades, 30+ copy-trade accounts, $42k single day (2026-01-29).
- [[wiki/entities/nova-sable-brains]] — Nova (methodical) + Sable (aggressive), DeepSeek R1 researchers in the Unified Brain. Spec only, not yet running.
- [[wiki/entities/tempo-trading-system]] — The overall NQ futures trading system at `~/Documents/trading-system/`. Research + backtest + planned Context Engine.
- [[wiki/entities/quantconnect]] — Backtest/execution platform. Project `28083727`. Runs the Tempo batch variants.
- [[wiki/entities/databento]] — Tick-level data vendor. Source for volume profile + footprint. Key saved on VPS, not yet integrated live.
- [[wiki/entities/ninjatrader-v5]] — The V5 SF portfolio (NQ VWPM, ES VWPM, NQ LF1170). Live-execution stack with 7 open bugs.

## Summaries

- [[wiki/summaries/audits-cluster]] — Cluster summary of 7 audit documents. Contamination chronology from V10i look-ahead (Feb) → V8 phantom B/E → bar-sim trailing (Apr).
- [[wiki/summaries/bos-fvg-failure-consolidated]] — **2026-04-10 failure report.** Paired bar-vs-tick replay: bar +0.359 avgR, tick +0.001 avgR. Invalidates every prior BOS_FVG "validated" claim.
- [[wiki/summaries/bos-fvg-10pt-audit]] — 2026-03-08 tick-level audit of BOS_FVG 10+pt static T3R. Baseline +0.015 avgR, nothing survives Bonferroni correction.
- [[wiki/summaries/research-arc-map]] *(also synthesis)* — **Master map** of the V7 → V11 research arc and the five-phase crisis cycle.
- [[wiki/summaries/tempo-cluster]] — 14-source Tempo cluster summary. Three-layer framework (canonical / mining / implementation).
- [[wiki/summaries/tempo-rules-v3]] — Master 281-transcript extract of what Tempo actually teaches. The Layer 1 canonical source.
- [[wiki/summaries/tempo-v14-corrections]] — 2026-03-20 audit of the v14 NinjaTrader C# code. 4 bugs, post-correction 70.5% WR US sessions.
- [[wiki/summaries/tempo-project-state]] — 2026-02-09 project state master doc. File inventory, architecture, standing corrections.
- [[wiki/summaries/tempo-ifvg-research]] — 2026-03-08 tick-level IFVG research. 74.7% WR baseline, 83% WR with SMT + reaction-quality. The cleanest Layer 2 doc in the cluster.
- [[wiki/summaries/lumi-strategy-spec]] — Lumi engine from @LumiTraders. v14 audit tested at 20% WR, excluded from production.
- [[wiki/summaries/unified-brain-architecture]] — The 2026-04 Willis Holdings pivot doc. Nova + Sable AI researchers, Mem0 memory, weekly tournaments, hard-coded Risk Officer, phase-gated progression. Replaces the entire mining-as-validation strategy.
- [[wiki/summaries/nq-playbook]] — V10 flagship findings doc. **Suspect** — headline +1.141 avgR / Cal 263.6 is bar-sim artifact per [[bos-fvg-failure-consolidated]].
- [[wiki/summaries/research-journal]] — V7 → V11 narrative journal. **Suspect** for same reasons as NQ Playbook.
- [[wiki/summaries/mining-analysis]] — V7aa_b cross-tab of 2,600 signal × pattern × session × DOW × alignment combos. **Suspect** (V10i alignment bug).
- [[wiki/summaries/look-ahead-audit]] — 2026-02-18 foundation audit. Swing confirmation + footprint absorption look-ahead bugs.
- [[wiki/summaries/v8-v9-audit-reports]] — V8 and V9 phase audits. **Suspect** — pre-dates V10o alignment bug discovery.
- [[wiki/summaries/comprehensive-mining-report-v2]] — V2 mining (34,323 trades, 258 days). BOS_FVG numbers now suspect due to [[bar-sim-trailing-bug]].
- [[wiki/summaries/strategy-mining-report-312d]] — V1 mining (312d, 8,358 trades). V10i-contaminated "sniper tiers" flagged suspect.
- [[wiki/summaries/nautilus-spec]] — NautilusTrader BOS_FVG spec. Published at +0.286 avgR — early red flag ignored until April audit.
- [[wiki/summaries/sf-portfolio-cluster]] — SF (Sweep-Fail) portfolio track. +3.34 R/day on 260d. **Tick-level simulator confirmed 2026-04-11.** RUNNER leg is fat-tailed (16% WR); needs full 260d for edge to show.
- [[wiki/summaries/15s-wick-fade]] — 15-second wick fade, tick-level validated on 260d. 43% WR, $30.89/trade. Flip variant dead (0/216 configs); fade alone is real.
- [[wiki/summaries/wickfade-complete]] — 5-minute WickFade, tick-level. +77–94 R/day, Calmar 2.12, **10/10 walk-forward OOS positive**. Strongest anti-overfit evidence in the corpus. Flagged the bar-sim trailing bug a month before the BOS_FVG audit.
- [[wiki/summaries/sweep-cluster]] — Sweep-and-fade experimental cluster. Most variants dead or marginal.
- [[wiki/summaries/transcript-mining-findings]] — Orderflow methodology synthesis from 7 educators, 30 transcripts. CLC framework, volume cluster pullbacks, POC reversal, footprint absorption.
- [[wiki/summaries/v5-strategy-bug-audit]] — 2026-03-23 audit of V5 NinjaTrader strategies. Seven bugs, Bug 1 (VWAP double-counting) is the primary cause of ES VWPM live/backtest divergence.
- [[wiki/summaries/tempo-context-engine-spec]] — Engineering spec v1.0 for the Tempo Context Engine. Session classifier, five feature layers, nine checkpoints, Bayesian belief updates, 5-phase build plan.
- [[wiki/summaries/tempo-quick-start-guide]] — How to start a Tempo session. `tempo` alias, file locations (Mac/VPS/GitHub/QC), credentials reference, memory document chain.
- [[wiki/summaries/option4-hybrid-architecture]] — Option 4: Hybrid Full Stack architecture document. Links QC, NinjaTrader, and Context Engine into one pipeline.

## Syntheses

- [[wiki/syntheses/research-arc-map]] — **Master map** of the research arc. Five phases, five bug waves, three parallel tracks (BOS_FVG / SF portfolio / Tempo IFVG), current Unified Brain pivot.
- [[wiki/syntheses/tempo-three-layers]] — The three-layer framework (canonical / mining / implementation) that every Tempo claim must be placed into before it can be evaluated.
- [[wiki/syntheses/bos-fvg-claim-vs-reality]] — Deep dive on why BOS_FVG is not Tempo's IFVG and why "BOS_FVG is the core signal of Tempo" was a two-layer mistake.
- [[wiki/syntheses/mining-reports-v1-v2-reconciliation]] — Cross-source analysis: why V1 claimed 97.9% WR while V2 shows 63.5%. Three corrections documented. **Note 2026-04-10**: even the V2 "honest" 63.5% WR is now invalidated for BOS_FVG — see [[bos-fvg-failure-consolidated]].

## Maps

- [[wiki/maps/root]] — **Start here.** Root Map of Content. Single entry point to the entire brain. Links every domain, top-10 concepts, vault health stats.
- [[wiki/maps/tempo-moc]] — Full Tempo research arc. Layer 1 (canonical methodology) → Layer 2 (mining/backtests — mostly suspect) → Layer 3 (current implementation). Evolution timeline from Oct 2024 to present.
- [[wiki/maps/bos-fvg-saga-moc]] — Case study: how BOS_FVG went from "97.9% WR sniper tier" to "63.5% WR core signal" to "DEAD at +0.001 avgR". Three contamination layers, three audits, four rules produced.
- [[wiki/maps/audit-history-moc]] — **Canonical.** Every audit sorted by outcome (bugs found vs missed). Each audit links to what it invalidated and the rules it produced. Chronological invalidation chain.
- [[wiki/maps/context-engine-moc]] — Bayesian session classifier. Five feature layers, nine checkpoints. Not yet built.
- [[wiki/maps/strategies-moc]] — All signal families: IFVG, Wick Fade (validated), Sweep Fade (dead), SF Portfolio (tick-confirmed), Lumi (excluded). Live status tracker.
- [[wiki/maps/personal-moc]] — Personal domain. Legal (mediation docs), security (Proton recovery), cross-links to trading as personal project.
- [[wiki/maps/work-moc]] — Haven Park Communities. Director of Asset Management. PE-backed MHC platform. Planned structure for when work docs arrive.
- [[wiki/maps/automation-moc]] — Scheduled tasks, ingest pipelines, lint sweeps. How the brain maintains itself. Four live tasks: daily-note, weekly-rollup, weekly-lint, inbox-triage.
- [[wiki/maps/import-triage-moc]] — Maps all 166 files in `raw-sources/imports/2026-04-11/` to their closest existing wiki page.

## Architecture

- [[BRAIN]] — Vault operating architecture. Six-layer knowledge pyramid, four core pipelines, automation schedule, hard rules.

## Projects (work)

*(none yet — see `user_role_haven_park.md` in auto-memory for the Haven Park context that should populate this section)*

## Projects (personal)

*(none yet — `personal/projects/tempo-trading/` is the expected first entry)*

## Inbox — pending ingest

- [[inbox/2026-04-11-vault-map-report|2026-04-11 Vault Map Report]] — full inventory of home-folder documents cross-referenced against `raw-sources/`. 336 scanned, 166 external unmapped, 52 duplicates (50 byte-identical, 2 content mismatches).
- [[inbox/2026-04-11-import-manifest|2026-04-11 Import Manifest]] — 166 files copied into `raw-sources/imports/2026-04-11/` mirroring original paths. Every file wiki-linked from the manifest so the graph shows them connected. Ready for Claude Code to run Ingest.
