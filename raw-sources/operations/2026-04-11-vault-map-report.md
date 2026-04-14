---
created: 2026-04-11
updated: 2026-04-11
type: synthesis
sources: []
related: []
tags: [inbox, vault-map, inventory]
---

# Vault Map Report

Full inventory of documents on the computer, cross-referenced against what's already
mapped into `HW/raw-sources/`. Generated 2026-04-11.

## Summary

| Bucket | Count |
|---|---|
| Total document files scanned (md/docx/pdf/xlsx/pptx/html) | 336 |
| Already represented in the vault (`raw-sources/`) by basename | 117 |
| External to vault, unmapped | 219 |
| External unmapped after dropping Claude session working dirs | 166 |

### Files by extension (after noise filter)

| Ext | Count |
|---|---|
| md | 230 |
| docx | 61 |
| xlsx | 15 |
| html | 15 |
| pdf | 14 |
| pptx | 1 |

## Scanning scope

The scan covered these roots under `/Users/harrisonwillis/`:

- `Documents/` (incl. `strategies/`, `trading-system/`, `research/`, `docs/`)
- `Downloads/`
- `Desktop/`
- `trading-system/`
- `trading-ceo/`
- `trading_operator/`
- `qc_operator_mvp/`

Excluded: `node_modules`, `.git`, `venv`, `.venv`, `__pycache__`, `site-packages`,
`dist`, `build`, Pine Script `.txt` source files, and Claude session working
directories under `Desktop/Claude/local-agent-mode-sessions/`.

## Already mapped (vault has a file with this basename)

See `docs_mapped.tsv` — 117 lines, format `<basename>\t<path>`.
Many files in `Documents/strategies/` are duplicates of files already in
`raw-sources/` (same basename, different location). Example:

- `Documents/strategies/tempo/TEMPO_IFVG_BUILD_SPEC.md` ⇔ `raw-sources/trading/tempo/TEMPO_IFVG_BUILD_SPEC.md`

These look like the original working copies that were later copied into `raw-sources/`.
Safe to leave in place; they are already in the vault under the same name.

## Unmapped — categorized

### A. Trading system — core knowledge (high priority)

Work-in-progress docs from `Documents/strategies/`, `Documents/trading-system/`, `trading-system/`.
These are exactly the kind of material the Ingest operation in `CLAUDE.md` is designed for.
Count: ~80 files.

Highlights:

- `Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Findings.docx`
- `Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec.md`
- `Documents/strategies/03-sf-portfolio/docs/BACKTEST_ISSUES_LOG.md`
- `Documents/strategies/03-sf-portfolio/docs/CONTEXT.md`
- `Documents/strategies/03-sf-portfolio/docs/NQ_SF_Engulfing_Strategy.docx`
- `Documents/strategies/03-sf-portfolio/docs/NT_AUDIT.md`
- `Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/CLAUDE.md`
- `Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/MINING_SIGNAL_SPEC.md`
- `Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/COWORK_PROMPT.md`
- `Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/NINJATRADER_BUILD_PROMPT.md`
- `Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/NINJATRADER_BUILD_SPEC.md`
- `Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/PORTFOLIO_AUDIT_V1.md`
- `Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/WORKING_LOG.md`
- `Documents/strategies/tempo/IFVG_TP_Optimization_Summary.pdf`
- `Documents/strategies/tempo/results/IFVG_Full_Year_Verification_Memo.docx`
- `Documents/strategies/tempo/results/IFVG_Optimization_Report_v2.docx`
- `Documents/strategies/tempo/scripts/AUDIT_PROMPT.md`
- `Documents/strategies/tempo/scripts/AUDIT_RESULTS_20260329.md`
- `Documents/strategies/tempo/tempo_portfolio_breakdown.xlsx`
- `Documents/trading-system/AGENTS.md`
- `Documents/trading-system/BOOTSTRAP.md`
- `Documents/trading-system/CLAUDE.md`
- `Documents/trading-system/CLAUDE_BOOTSTRAP.md`
- `Documents/trading-system/HEARTBEAT.md`
- `Documents/trading-system/IDENTITY.md`
- `Documents/trading-system/MINE_V2_PROMPT.md`
- `Documents/trading-system/MOTIVEWAVE_PORT_PROMPT.md`
- `Documents/trading-system/MOTIVEWAVE_SETUP.md`
- `Documents/trading-system/MW_CLAUDE_CODE_PROMPT.md`
- `Documents/trading-system/NQ Test.xlsx`
- `Documents/trading-system/NQ_Condition_Matrix.xlsx`
- `Documents/trading-system/OVERNIGHT_MINE_PROMPT.md`
- `Documents/trading-system/SETUP.md`
- `Documents/trading-system/SOUL.md`
- `Documents/trading-system/START_HERE.md`
- `Documents/trading-system/TOOLS.md`
- `Documents/trading-system/Tempo_V8f_Strategy_A_Results.xlsx`
- `Documents/trading-system/USER.md`
- `Documents/trading-system/V6_Portfolio_Tracker.xlsx`
- `Documents/trading-system/Willis_Holdings_Strategy_Audit.docx`
- `Documents/trading-system/batch-runner/CHECKLIST.md`
- `Documents/trading-system/batch-runner/DELIVERY_SUMMARY.md`
- `Documents/trading-system/batch-runner/EXAMPLES.md`
- `Documents/trading-system/batch-runner/IMPLEMENTATION_NOTES.md`
- `Documents/trading-system/batch-runner/INDEX.md`
- `Documents/trading-system/batch-runner/QUICKSTART.md`
- `Documents/trading-system/batch-runner/README.md`
- `Documents/trading-system/batch-runner/SYSTEM_STATE.md`
- `Documents/trading-system/context-engine/QUICK_REFERENCE.md`
- `Documents/trading-system/context-engine/TRADE_TYPES_VISUAL_GUIDE.md`
- `Documents/trading-system/context-engine/context-engine/QUICK_REFERENCE.md`
- `Documents/trading-system/context-engine/context-engine/data_ingestion/README.md`
- `Documents/trading-system/context-engine/data_ingestion/README.md`
- `Documents/trading-system/fib_v2_results.xlsx`
- `Documents/trading-system/ifvg-backtest/results/IFVG_Full_Year_Verification_Memo.docx`
- `Documents/trading-system/knowledge-base/master.md`
- `Documents/trading-system/nautilus-bt/CLAUDE.md`
- `Documents/trading-system/nautilus-bt/MINING_SIGNAL_SPEC.md`
- `Documents/trading-system/openclaw/IDENTITY.md`
- `Documents/trading-system/openclaw/skills/ceo-briefing/SKILL.md`
- `Documents/trading-system/openclaw/skills/claude-code-manager/SKILL.md`
- `Documents/trading-system/openclaw/skills/shell-runner/SKILL.md`
- `Documents/trading-system/openclaw/skills/trading-operator/SKILL.md`
- `Documents/trading-system/operator/CLAUDE.md`
- `Documents/trading-system/results/BOS_FVG_RESEARCH.md`
- `Documents/trading-system/results/FINAL_SUMMARY.md`
- `Documents/trading-system/results/IFVG_AUDIT_RESULTS.md`
- `Documents/trading-system/results/IFVG_AUDIT_RESULTS_V2.md`
- `Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT.md`
- `Documents/trading-system/results/RESEARCH_LOG_20260304.md`
- `Documents/trading-system/results/VALIDATION_SUMMARY.md`
- `Documents/trading-system/results/WickFade_5M_Research_Findings.docx`
- `Documents/trading-system/results/archive/bar-sim-pre-2026-04-10/README.md`
- `Documents/trading-system/results/wickfade_5m_findings_20260303.md`
- `Documents/trading-system/trading_operator/logs/20260217_045105_health_check.md`
- `Documents/trading-system/trading_operator/logs/health_check.md`
- `trading-system/BACKTESTING_AUDIT.md`
- `trading-system/CLAUDE.md`
- `trading-system/CLAUDE_BOOTSTRAP.md`
- `trading-system/CONGLOMERATE_SESSION_LOG.md`
- `trading-system/CONTEXT.md`
- `trading-system/LOCAL_SETUP.md`
- `trading-system/NQ Test.xlsx`
- `trading-system/SESSION_NOTES_FEB18.md`
- `trading-system/SETUP.md`
- `trading-system/START_HERE.md`
- `trading-system/TEMPO_Carmine_Strategy_2026-02-16.docx`
- `trading-system/TEMPO_Strategy_Analysis_2026-02-16.docx`
- `trading-system/Willis_Holdings_CEO_Schematic.html`
- `trading-system/repo_inventory.md`

### B. Tempo / pipeline docs in Downloads (medium priority)

Downloads containing Tempo system docs and pipeline guides. Probably originated as
handoff uploads. Count: ~15 files.

- `Downloads/Autonomous_Trading_System_Progress_Summary.docx`
- `Downloads/NY_Session_Reversal_System.md`
- `Downloads/OpenClaw_VPS_Setup_Guide.docx`
- `Downloads/Option4_Hybrid_Handoff_Document.docx`
- `Downloads/Tempo_Strategy_Mobile_Version_with_Claude_Memory.pdf`
- `Downloads/Trading_Operator_Strategy_Roadmap.pptx`
- `Downloads/Video_Ingestion_Pipeline_Guide.docx`
- `Downloads/tempo_extracted/README_V2.md`
- `Downloads/tempo_extracted/V2_IMPLEMENTATION_SUMMARY.md`
- `Downloads/tempo_extracted/tempo_rules_from_videos.md`
- `Downloads/tempo_extracted/tempo_rules_summary.md`
- `Downloads/tempo_pipeline/DATA_SETUP_GUIDE.md`
- `Downloads/tempo_pipeline/QC_SETUP_GUIDE.md`
- `Downloads/tempo_pipeline/QUICK_START.md`
- `Downloads/tempo_pipeline/README_QUANTCONNECT_IMPLEMENTATION.md`
- `Downloads/tempo_pipeline/TEMPO_RULES_IMPLEMENTATION.md`
- `Downloads/tempo_pipeline/upload_report.md`
- `Downloads/tempo_progress_dashboard.html`
- `Downloads/tempo_uploader.html`

### C. Implementation reviews and dashboards (low-medium priority)

Loose HTML dashboards, review docs, and architecture snapshots. Some may be
reference artifacts worth keeping, some may be disposable.

- `Documents/.plan.md`
- `Documents/EXCEL_STRATEGY_OVERVIEW.md`
- `Documents/LumiTraders_Strategy.pdf`
- `Documents/LumiTraders_Strategy_v2.pdf`
- `Documents/NQ Test.xlsx`
- `Documents/NQ_Condition_Matrix.xlsx`
- `Documents/NQ_Mining_Results_V2.xlsx`
- `Documents/README_EXCEL_ANALYSIS.md`
- `Documents/START_HERE.md`
- `Documents/Tempo_Batch2_Winners.xlsx`
- `Documents/Tempo_Batch3_Results.xlsx`
- `Documents/Tempo_Batch_Cheatsheet.docx`
- `Documents/Tempo_Inverse_Analysis.xlsx`
- `Documents/Tempo_Portfolio_V15.docx`
- `Documents/Tempo_V8f_Strategy_A_Results.xlsx`
- `Documents/V17_AUDIT_PROMPT.md`
- `Documents/fib_786_strategy_report.html`
- `Documents/fib_retracement_diagram.html`
- `Documents/lumi_original_vs_ours.html`
- `Documents/lumi_vs_ifvg_comparison.html`
- `Documents/market_vs_limit_dashboard.html`
- `Documents/nq_fib_bug_tracker.xlsx`
- `Documents/operator_architecture.html`
- `Documents/willis_holdings_architecture.html`
- `Downloads/architecture_comparison.html`
- `Downloads/implementation_review.docx`
- `Downloads/implementation_review_1.docx`
- `Downloads/infrastructure_architecture.html`
- `Downloads/tempo_progress_dashboard.html`
- `Downloads/tempo_uploader.html`

### D. Personal / legal / receipts (personal/ folder candidates)

These belong under `personal/` in the vault, not `raw-sources/trading/`.
Count: ~8 files. Includes mediation paperwork and an item fulfillment receipt.

- `Downloads/2026_0309_Compromise_Settlement_and_Agreement_for_Partition_(Jarmon)(8579879.1).docx.pdf`
- `Downloads/2026_0310_Settlement_Agreement_(Jarmon)(8581004.1).docx.pdf`
- `Downloads/AGREEMENT FOR MEDIATION (1).pdf`
- `Downloads/AGREEMENT FOR MEDIATION.pdf`
- `Downloads/Here_is_your_signed_document_AGREEMENT_FOR_M.pdf`
- `Downloads/Hwillis_Mediation_Signed.pdf`
- `Downloads/Item Fulfillment_IF356977.pdf`
- `Downloads/proton-recovery-kit.pdf`

### E. QuantConnect data folder readmes (skip)

These are read-only vendor/sample data READMEs. Not personal knowledge. Recommend skip.

- `Downloads/data/cfd/readme.md`
- `Downloads/data/crypto/readme.md`
- `Downloads/data/equity/readme.md`
- `Downloads/data/equity/usa/readme.md`
- `Downloads/data/forex/readme.md`
- `Downloads/data/future/cbot/margins/readme.md`
- `Downloads/data/future/cfe/margins/readme.md`
- `Downloads/data/future/cme/margins/readme.md`
- `Downloads/data/future/comex/margins/readme.md`
- `Downloads/data/future/ice/margins/readme.md`
- `Downloads/data/future/nymex/margins/readme.md`
- `Downloads/data/future/readme.md`
- `Downloads/data/option/readme.md`
- `Downloads/data/readme.md`

## Files currently in vault but no wiki summary yet

`raw-sources/` contains 57 files. `wiki/summaries/` has 18 markdown summaries. That
means Claude Code still has a backlog of raw sources to summarize. The summaries
that DO exist:

- `bos-fvg-10pt-audit.md`
- `bos-fvg-failure-consolidated.md`
- `comprehensive-mining-report-v2.md`
- `look-ahead-audit.md`
- `lumi-strategy-spec.md`
- `mining-analysis.md`
- `nq-playbook.md`
- `research-journal.md`
- `sf-portfolio-cluster.md`
- `strategy-mining-report-312d.md`
- `tempo-cluster.md`
- `tempo-context-engine-spec.md`
- `tempo-ifvg-research.md`
- `tempo-project-state.md`
- `tempo-quick-start-guide.md`
- `tempo-rules-v3.md`
- `tempo-v14-corrections.md`
- `unified-brain-architecture.md`
- `v5-strategy-bug-audit.md`

To see the raw sources without a matching summary, diff the two lists or run
`/lint` inside the vault.

## Recommended next steps

1. Review category A (trading core) and pick which files should be copied into
   `raw-sources/` for Claude Code to ingest. Most of `Documents/strategies/` looks
   like the parent copy of material already in the vault — likely already captured.
2. Copy category B (Tempo / pipeline docs) into `raw-sources/trading/tempo/` or
   `raw-sources/trading/pipeline/` for ingest.
3. Category C and D files should go to their own subfolders
   (`raw-sources/personal/`, `raw-sources/work/`) rather than defaulting to trading.
4. Category E — skip.

## Artifacts in this report

- `all_docs.txt` — raw scan output, 336 document files (md/docx/pdf/xlsx/pptx/html)
- `docs_mapped.tsv` — files whose basename is already in `raw-sources/`
- `docs_unmapped.tsv` — everything else
- `docs_unmapped_external.tsv` — the same, minus files already in the vault
- `unmapped_external_curated.txt` — 166 items, minus Claude session working dirs
- `raw_basenames.txt` — 57 basenames currently in `raw-sources/`
