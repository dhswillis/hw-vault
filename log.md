# Activity log

Append-only chronological record of vault operations. Entries use the format `## [YYYY-MM-DD HH:MM] <operation> | <description>` so they can be greppable:

```bash
grep "^## \[" log.md | tail -20
```

## [2026-04-10 15:27] setup | vault scaffolded

## [2026-04-10 17:12] ingest | COMPREHENSIVE_MINING_REPORT.md → wiki/summaries/comprehensive-mining-report-v2.md; concepts bos-fvg, mtf-alignment, v10i-look-ahead-bug, be-trail-mechanism

## [2026-04-10 17:12] ingest | STRATEGY_MINING_REPORT.md → wiki/summaries/strategy-mining-report-312d.md; flagged V1-era suspect-results (V10i contamination)

## [2026-04-10 17:12] ingest | V5_Strategy_Bug_Audit_2026-03-23.md → wiki/summaries/v5-strategy-bug-audit.md; concepts vwap-double-counting-bug; entity ninjatrader-v5

## [2026-04-10 17:12] ingest | Tempo_Context_Engine_Spec.docx → wiki/summaries/tempo-context-engine-spec.md; concepts volume-profile, session-type-taxonomy, bayesian-belief-engine

## [2026-04-10 17:12] ingest | Tempo_Quick_Start_Guide.docx → wiki/summaries/tempo-quick-start-guide.md; entity tempo-trading-system, quantconnect, databento

## [2026-04-10 17:12] synthesis | mining-reports-v1-v2-reconciliation.md — cross-source audit reconciling V1 97.9% WR vs V2 63.5% WR on BOS_FVG; three bias corrections documented

## [2026-04-10 19:30] audit | BOS_FVG paired bar-vs-tick trailing replay on 60-day Databento sample — bar +0.359 avgR, tick +0.001 avgR, delta +0.29 R/trade; confirms CLAUDE.md position that BOS_FVG DOES NOT WORK

## [2026-04-10 19:30] ingest | ~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md → wiki/summaries/bos-fvg-failure-consolidated.md; concept bar-sim-trailing-bug created; bos-fvg.md rewritten as dead-strategy

## [2026-04-11 08:30] ingest | tempo_rules_v3_complete.md → wiki/summaries/tempo-rules-v3.md; concepts smt, dol-framework, bpr, setup-quality-grading; entity tempo-methodology

## [2026-04-11 08:30] ingest | TEMPO_V14_CORRECTIONS.md → wiki/summaries/tempo-v14-corrections.md; concept tempo-v14-corrections

## [2026-04-11 08:30] ingest | TEMPO_PROJECT_STATE.md → wiki/summaries/tempo-project-state.md

## [2026-04-11 08:30] ingest | TEMPO_IFVG_RESEARCH.md → wiki/summaries/tempo-ifvg-research.md

## [2026-04-11 08:30] ingest | LUMI_STRATEGY_SPEC.md → wiki/summaries/lumi-strategy-spec.md

## [2026-04-11 08:30] ingest | NQ_PLAYBOOK.md → wiki/summaries/nq-playbook.md (tagged suspect-results, contaminated by bar-sim-trailing-bug)

## [2026-04-11 08:30] ingest | RESEARCH_JOURNAL.md → wiki/summaries/research-journal.md (tagged suspect-results)

## [2026-04-11 08:30] ingest | LOOK_AHEAD_AUDIT_2026-02-18.md → wiki/summaries/look-ahead-audit.md

## [2026-04-11 08:30] ingest | BOS_FVG_10PT_AUDIT.md + BOS_FVG_AUDIT_FINDINGS.md → wiki/summaries/bos-fvg-10pt-audit.md

## [2026-04-11 08:30] ingest | MINING_ANALYSIS.md → wiki/summaries/mining-analysis.md (tagged suspect-results)

## [2026-04-11 08:30] ingest | The_Unified_Brain_Architecture.docx → wiki/summaries/unified-brain-architecture.md (stub — needs docx conversion); entity nova-sable-brains

## [2026-04-11 08:30] ingest | sf-portfolio/*.md → wiki/summaries/sf-portfolio-cluster.md (cluster summary, 6 source files)

## [2026-04-11 08:30] ingest | sweep/*.md → wiki/summaries/sweep-cluster.md (cluster summary, 8 source files)

## [2026-04-11 08:30] ingest | NAUTILUS_SPEC.md → wiki/summaries/nautilus-spec.md

## [2026-04-11 08:30] ingest | TRANSCRIPT_MINING_FINDINGS.md → wiki/summaries/transcript-mining-findings.md

## [2026-04-11 08:30] ingest | V8_Full_Audit_Report.md + V9_Full_Audit_Report.md → wiki/summaries/v8-v9-audit-reports.md (tagged suspect-results)

## [2026-04-11 08:30] synthesis | bos-fvg-claim-vs-reality.md — BOS_FVG ≠ IFVG, two-layer mistake (wrong strategy AND bar-sim trailing)

## [2026-04-11 08:30] synthesis | tempo-three-layers.md — framework placing every Tempo claim into Layer 1 canonical / Layer 2 mining / Layer 3 implementation

## [2026-04-11 08:30] synthesis | research-arc-map.md — master map of five phases, five bug waves, three parallel tracks, Unified Brain pivot

## [2026-04-11 14:30] vault-map | scanned home folder (336 docs); 166 unmapped copied to raw-sources/imports/2026-04-11/ with full wiki-link manifest; 2 content-mismatched duplicates flagged without overwrite

## [2026-04-11 14:45] schema | CLAUDE.md upgraded — Obsidian best practices integrated: new operations /inbox /weekly /moc /capture; new folders wiki/maps/, weekly/, personal/legal/; reserved tags extended (dead-strategy, canonical, draft, in-flight, needs-reingest); authority hierarchy codified; plugin recommendations (Templater, Dataview, Periodic Notes, Tasks, Calendar, QuickAdd)

## [2026-04-11 14:45] templates | added templates/weekly.md, templates/moc.md, templates/decision.md, templates/person.md

## [2026-04-11 14:45] templates | added templates/prompts/ reusable prompt library — weekly-review.md, inbox-processing.md, ingest.md, moc.md

## [2026-04-11 14:45] concepts | wiki/concepts/maps-of-content.md + wiki/concepts/inbox-processing.md created to back the new /moc and /inbox operations

## [2026-04-11 14:45] ingest | audits cluster cataloged → wiki/summaries/audits-cluster.md (7-file contamination chronology view)

## [2026-04-11 14:45] memory | saved user_role_haven_park.md from Claude data export — Harrison is Director of AM at Haven Park Communities (PE-backed MHC, 30k→60-80k units); institutional RE is primary context, trading is side project

## [2026-04-11 14:45] memory | saved tempo_strategy_codified_rules.md — Harrison's own canonical Tempo entry rules (FVG on dynamic TF → close through → HTF DOL sweep → inverse entry)

## [2026-04-11 14:45] memory | saved unified_brain_pivot.md — Nova + Sable AI researchers, BOS_FVG DEAD confirmed, v24 IFVG MTF cascade only sim strategy, 7 dead signals listed, anti-stupidity rules hard-coded

## [2026-04-11 14:45] memory | saved bar_sim_trailing_bug_rule.md — DO NOT TRAIL ON BARS; use tick-level, fixed R, or close-through soft stops

## [2026-04-11 14:45] memory | saved feedback_user_corrections_authoritative.md — user mid-session edits are ground truth; re-thread downstream rather than argue

## [2026-04-11 14:45] privacy | extended .gitignore to exclude personal/legal/ and raw-sources/claude-data/ from git tracking

## [2026-04-11 16:00] ingest | The_Unified_Brain_Architecture.docx → wiki/summaries/unified-brain-architecture.md (full 440-line conversion via textutil); nova-sable-brains entity fully expanded from stub

## [2026-04-11 16:00] ingest | 15S_Wick_Fade_Findings.docx → wiki/summaries/15s-wick-fade.md (tick-validated, 260d, 43% WR, flip dead)

## [2026-04-11 16:00] ingest | WickFade_Complete_Findings.docx → wiki/summaries/wickfade-complete.md (tick-validated 5m variant, 10/10 walk-forward OOS, identified bar-sim trailing bug in March 2026)

## [2026-04-11 16:00] concept | wiki/concepts/wick-fade.md — family concept page for both wick-fade variants; linked from sweep-cluster and research-arc-map

## [2026-04-11 16:00] audit | SF portfolio simulator code-read confirms tick-level trade management (trade_f/trade_be5/trade_es iterate individual ticks, not OHLC bars). 22-day sanity check: −0.428 R/day vs published +1.384 R/day — gap entirely in RUNNER leg (0/12 wins vs expected ~2 at 16% WR, p=11.5%). Conclusion: fat-tail variance, not a bug. sf-portfolio-cluster.md updated with corrected tick-level assessment. Sanity script saved to results/archive/sf-portfolio-tick-audit-2026-04-11/
