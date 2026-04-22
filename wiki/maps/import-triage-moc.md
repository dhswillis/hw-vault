---
created: 2026-04-11
updated: 2026-04-21
type: moc
topic: Import Triage
tags: [moc, imports, triage, methodology]
related:
  - wiki/maps/tempo-moc.md
  - wiki/maps/bos-fvg-saga-moc.md
  - wiki/maps/audit-history-moc.md
---

# Import Triage — Map of Content

> Maps every file in `raw-sources/imports/2026-04-11/` to its closest existing wiki page. 138 files triaged: 6 already covered, 53 low-signal boilerplate, 79 needing mapping. This MOC threads the unmapped files to the vault's existing graph.

## Triage summary

| Category | Count | Action |
|---|---|---|
| Already covered by existing summary | 6 | No action — existing page handles the content |
| Low-signal boilerplate | 53 | Skip — CLAUDE.md, SETUP.md, README.md, etc. |
| Legal/personal (sensitive) | 8 | Catalog only — do not ingest content, gitignored |
| IFVG family (extends existing) | 6 | Thread to [[tempo-ifvg-research]], [[tempo-cluster]] |
| BOS_FVG family (dead) | 2 | Thread to [[bos-fvg-failure-consolidated]], [[bos-fvg-saga-moc]] |
| Wick-fade family (extends existing) | 3 | Thread to [[wick-fade]], [[wickfade-complete]], [[15s-wick-fade]] |
| SF Portfolio family | 3 | Thread to [[sf-portfolio-cluster]] |
| Keylevel sweep family | 4 | Thread to [[sweep-cluster]] |
| Tempo core docs | 8 | Thread to [[tempo-cluster]], [[tempo-moc]] |
| Research/results archives | 7 | Thread to [[research-journal]], [[research-arc-map]] |
| System architecture/infra | 10 | Thread to [[tempo-project-state]], [[unified-brain-architecture]] |
| Context engine | 1 | Thread to [[tempo-context-engine-spec]] |
| Pipeline/misc | 8 | Thread to [[tempo-quick-start-guide]] or skip |

## Legal/Personal — CATALOG ONLY (gitignored)

These exist in `raw-sources/imports/` but are gitignored via `personal/legal/`. Content is NOT ingested. This catalog confirms they are accounted for.

| File | Subject | Cross-reference |
|---|---|---|
| `2026_0309_Compromise_Settlement...pdf` | Jarmon partition settlement | Estate of Dan Alvin Willis (see user_role_haven_park memory) |
| `2026_0310_Settlement_Agreement...pdf` | Jarmon settlement agreement | Same |
| `AGREEMENT FOR MEDIATION.pdf` | Mediation agreement | Same |
| `AGREEMENT FOR MEDIATION (1).pdf` | Duplicate | Same |
| `Here_is_your_signed_document...pdf` | Signed mediation | Same |
| `Hwillis_Mediation_Signed.pdf` | Harrison's signed mediation | Same |
| `Item Fulfillment_IF356977.pdf` | Fulfillment receipt | Unknown — personal purchase |
| `proton-recovery-kit.pdf` | **DO NOT INGEST** — credential recovery kit | Security-sensitive |

## IFVG Family → thread to existing summaries

These extend the content already covered by [[tempo-ifvg-research]] and [[tempo-cluster]].

| File | Maps to | Why |
|---|---|---|
| `results/IFVG_AUDIT_RESULTS.md` | [[tempo-ifvg-research]] | V1 IFVG audit — superseded by V2 |
| `results/IFVG_AUDIT_RESULTS_V2.md` | [[tempo-ifvg-research]] | V2 IFVG audit — this IS the doc that tempo-ifvg-research summarizes |
| `results/IFVG_OPTIMIZATION_REPORT.md` | [[tempo-cluster]] | IFVG optimization — covered by cluster summary |
| `results/IFVG_Full_Year_Verification_Memo.docx` (×2) | [[tempo-ifvg-research]] | Full-year verification — extends IFVG research |
| `results/IFVG_Optimization_Report_v2.docx` | [[tempo-cluster]] | V2 of optimization report |
| `IFVG_TP_Optimization_Summary.pdf` | [[tempo-cluster]] | TP optimization summary (PDF version of docx already in cluster) |

## BOS_FVG Family → thread to bos-fvg-saga

| File | Maps to | Why |
|---|---|---|
| `results/BOS_FVG_RESEARCH.md` | [[bos-fvg-failure-consolidated]] | The original research doc that [[bos-fvg-failure-consolidated]] supersedes |
| `MINING_SIGNAL_SPEC.md` (×2) | [[bos-fvg]] concept page | Signal specification — the definition that generated the dead signal |

## Wick-Fade Family → thread to existing summaries

| File | Maps to | Why |
|---|---|---|
| `WickFade_Strategy_Findings.docx` | [[wickfade-complete]] | Earlier findings — superseded by complete findings |
| `WickFade_Strategy_Spec.md` | [[wick-fade]] concept page | Strategy spec for the wick-fade family |
| `WickFade_5M_Research_Findings.docx` / `wickfade_5m_findings_20260303.md` | [[wickfade-complete]] | 5-minute variant findings — content reflected in the complete summary |

## SF Portfolio Family → thread to existing

| File | Maps to | Why |
|---|---|---|
| `NQ_SF_Engulfing_Strategy.docx` | [[sf-portfolio-cluster]] | V1 engulfing strategy — V2 is the canonical version |
| `BACKTEST_ISSUES_LOG.md` | [[sf-portfolio-cluster]] | Issues log — historical debugging trail |
| `NT_AUDIT.md` | [[v5-strategy-bug-audit]] | NinjaTrader audit — content reflected in V5 bug audit summary |

## Keylevel Sweep Family → thread to sweep-cluster

| File | Maps to | Why |
|---|---|---|
| `NINJATRADER_BUILD_SPEC.md` | [[sweep-cluster]] | Build spec for NinjaTrader keylevel sweep strategies |
| `NINJATRADER_BUILD_PROMPT.md` | [[sweep-cluster]] | Build prompt — lower signal than spec |
| `PORTFOLIO_AUDIT_V1.md` | [[sweep-cluster]] | V1 of portfolio audit — V2 already in sweep cluster |
| `WORKING_LOG.md` | [[sweep-cluster]] | Working log — historical session trail |

## Tempo Core Docs → thread to tempo-cluster and tempo-moc

| File | Maps to | Why |
|---|---|---|
| `TEMPO_Carmine_Strategy_2026-02-16.docx` | [[tempo-methodology]] | Carmine's strategy input — educator-family content |
| `TEMPO_Strategy_Analysis_2026-02-16.docx` | [[tempo-moc]] | Strategy analysis doc — pre-Unified-Brain era |
| `Tempo_Portfolio_V15.docx` | [[tempo-cluster]] | Portfolio V15 spec — superseded by V14 corrections and Unified Brain |
| `Tempo_Batch_Cheatsheet.docx` | [[tempo-quick-start-guide]] | Quick-reference for batch runner |
| `V17_AUDIT_PROMPT.md` | [[tempo-moc]] | V17 audit prompt — NinjaTrader SweepBreak v17 |
| `AUDIT_PROMPT.md` + `AUDIT_RESULTS_20260329.md` | [[audits-cluster]] | March 29 audit prompt + results |
| `Willis_Holdings_Strategy_Audit.docx` | [[unified-brain-architecture]] | Willis Holdings strategy audit — feeds the Unified Brain pivot narrative |

## Research/Results Archives → thread to research-arc-map

| File | Maps to | Why |
|---|---|---|
| `RESEARCH_LOG_20260304.md` | [[research-journal]] | March 4 research log — session-level entry |
| `FINAL_SUMMARY.md` | [[research-arc-map]] | Summary doc from a research phase |
| `VALIDATION_SUMMARY.md` | [[research-arc-map]] | Validation summary from a mining run |
| `BACKTESTING_AUDIT.md` | [[audits-cluster]] | Backtesting audit — pre-April era |
| `CONGLOMERATE_SESSION_LOG.md` | [[unified-brain-architecture]] | Session log for the "Conglomerate" (Unified Brain predecessor) |
| `SESSION_NOTES_FEB18.md` | [[research-journal]] | Session notes — Feb 18 snapshot |
| `KNOWLEDGE_BASE.md` | [[tempo-project-state]] | Knowledge base doc — project infra |

## System Architecture/Infrastructure

| File | Maps to | Why |
|---|---|---|
| `Option4_Hybrid_Handoff_Document.docx` | [[tempo-project-state]] | **THE master architecture doc** — Option 4 Hybrid Full Stack. Harrison has said "read this" multiple times. |
| `Autonomous_Trading_System_Progress_Summary.docx` | [[unified-brain-architecture]] | Progress summary — pre-Unified-Brain snapshot |
| `Video_Ingestion_Pipeline_Guide.docx` | [[tempo-project-state]] | How to extract from Tempo videos |
| `OpenClaw_VPS_Setup_Guide.docx` | [[tempo-project-state]] | VPS setup for OpenClaw agent |
| `Tempo_Strategy_Mobile_Version_with_Claude_Memory.pdf` | [[tempo-methodology]] | Mobile-friendly strategy ref |
| `implementation_review.docx` / `implementation_review_1.docx` | [[tempo-project-state]] | Implementation review (duplicate pair) |
| `START_HERE.md` (×2 — Documents and trading-system) | [[tempo-project-state]] | Already summarized as the master state doc |
| `CLAUDE_BOOTSTRAP.md` | [[tempo-project-state]] | Technical bootstrap — credentials + integrations |

## Context Engine

| File | Maps to | Why |
|---|---|---|
| `TRADE_TYPES_VISUAL_GUIDE.md` | [[tempo-context-engine-spec]] | Visual guide to trade types — reference for the Context Engine session classifier |

## Pipeline/Infrastructure/Misc

| File | Maps to | Disposition |
|---|---|---|
| `LumiTraders_Strategy.pdf` / `LumiTraders_Strategy_v2.pdf` | [[lumi-strategy-spec]] | PDFs of the Lumi strategy. V2 is canonical. |
| `NY_Session_Reversal_System.md` | [[sweep-cluster]] | NY session reversal — experimental, likely dead |
| `README_V2.md` / `V2_IMPLEMENTATION_SUMMARY.md` | [[tempo-rules-v3]] | V2 extraction implementation — superseded by V3 |
| `upload_report.md` | skip | Pipeline report — ephemeral |
| `DATA_SETUP_GUIDE.md` / `QC_SETUP_GUIDE.md` / `README_QUANTCONNECT_IMPLEMENTATION.md` | [[quantconnect]] | QC setup guides — infra reference |
| `HEARTBEAT.md` + `USER.md` (openclaw) | skip | Agent heartbeat files — ephemeral |
| `EXCEL_STRATEGY_OVERVIEW.md` / `README_EXCEL_ANALYSIS.md` | [[research-arc-map]] | Excel analysis of strategy results — historical reference |
| `.plan.md` | skip | Ephemeral plan file |
| `health_check.md` files | skip | Trading operator health checks — ephemeral |

## What should actually be individually summarized (prioritized)

If you want to promote any file from "threaded to existing page" to "has its own wiki summary," these are the highest-value candidates:

1. **`Option4_Hybrid_Handoff_Document.docx`** — THE master architecture doc that Harrison cites as the canonical system design. Currently only threaded to tempo-project-state. Worth its own summary.
2. **`Willis_Holdings_Strategy_Audit.docx`** — Willis Holdings (= Harrison's entity) strategy audit. Feeds the Unified Brain pivot narrative.
3. **`AUDIT_RESULTS_20260329.md`** — March 29 audit results. Could extend the audits-cluster or audit-history-moc.
4. **`NINJATRADER_BUILD_SPEC.md`** (26KB) — big build spec for the keylevel sweep NinjaTrader strategy.
5. **`BACKTESTING_AUDIT.md`** — could extend audits-cluster if it contains findings not already captured.
6. **`IFVG_Full_Year_Verification_Memo.docx`** — full-year verification of the IFVG strategy.

Everything else is adequately served by the thread-to-existing-page mapping above.

## Related summaries

- [[wiki/summaries/remaining-imports-cluster]] — 14 miscellaneous import files (identity files, operational docs, trading PDFs) grouped into one cluster summary.
- [[wiki/summaries/ingest-audit-20260421]] — 2026-04-21 programmatic audit of which of the 166 imports have/haven't earned a summary. All PDFs/DOCX complete; 49 residual lower-format files triaged.
