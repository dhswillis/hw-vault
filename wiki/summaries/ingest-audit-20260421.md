---
created: 2026-04-21
updated: 2026-04-21
type: summary
sources:
  - raw-sources/imports/2026-04-11/ (recursive, 166 files)
related:
  - wiki/maps/import-triage-moc.md
  - wiki/lint-report.md
tags: [audit, ingest, triage]
---

# Ingest Audit — 2026-04-21

> Programmatic cross-reference of the 166-file import batch against `wiki/summaries/*.md` `sources:` frontmatter. Executed via `grep`/`awk` (not file-by-file LLM pass) after a first agent attempt hit the 8k output-token ceiling. See method notes at bottom.

## Summary

| Disposition | Count |
|---|---|
| **COVERED** — filename appears in a summary's `sources:` field | 117 |
| **MISSING** — no summary references this filename | 49 |
| **Total** | 166 |

### Missing files by extension

| Ext | Count | Disposition |
|---|---|---|
| `.md` | 21 | Boilerplate — CLAUDE_BOOTSTRAP, BOOTSTRAP, CONTEXT, QUICK_START, 13x readme duplicates. **SKIP.** |
| `.xlsx` | 15 | Data files. **2–3 worth ingesting** per 2026-04-18 lint. See below. |
| `.html` | 12 | Rendered dashboards/diagrams. **Low signal** — visualizations of data already in summaries. |
| `.pptx` | 1 | `Trading_Operator_Strategy_Roadmap.pptx` — **worth ingesting.** Unique format, likely novel content. |

## Headline finding

**Every PDF and DOCX in the 166-file batch already has a summary.** The priority Harrison set — "PDFs and docx that need text extraction" — is complete.

- 12 PDFs total: 8 legal/personal (SKIP per privacy boundary), 1 receipt (SKIP), 3 trading PDFs all covered by existing summaries (LumiTraders x2 → `lumi-strategy-spec`; IFVG TP Opt → `ifvg-tp-optimization-report`; Tempo Mobile → `tempo-methodology`).
- 17 DOCX total: all covered (Option4 Hybrid → `option4-hybrid-architecture`; Willis Holdings audit → `willis-holdings-strategy-audit`; etc.). Full mapping in [[import-triage-moc]].

## NEEDS-INGEST — priority 1

None by Harrison's stated priority (PDFs/DOCX all done).

## NEEDS-INGEST — priority 2 (bonus, non-PDF/DOCX formats worth extracting)

| File | Type | Proposed slug | MOC thread |
|---|---|---|---|
| `Trading_Operator_Strategy_Roadmap.pptx` | pptx | `trading-operator-strategy-roadmap` | [[wiki/maps/automation-moc]] |
| `NQ_Mining_Results_V2.xlsx` | xlsx | (may already be absorbed into `comprehensive-mining-report-v2` — verify before ingesting) | [[wiki/maps/audit-history-moc]] |
| `Tempo_Batch3_Results.xlsx` | xlsx | `tempo-batch3-results` | [[wiki/maps/tempo-moc]] |

## NEEDS-INGEST — priority 3 (low value, likely skip)

- `Tempo_Batch2_Winners.xlsx`, `Tempo_Inverse_Analysis.xlsx`, `tempo_portfolio_breakdown.xlsx`, `V6_Portfolio_Tracker.xlsx`, `Tempo_V8f_Strategy_A_Results.xlsx`, `NQ_Condition_Matrix.xlsx`, `fib_v2_results.xlsx`, `nq_fib_bug_tracker.xlsx`, `Test.xlsx` — mostly already-absorbed data tables.
- 12 HTML dashboards — all visualizations of data captured elsewhere.

## Boilerplate (SKIP)

21 MD files: `BOOTSTRAP.md`, `CLAUDE_BOOTSTRAP.md` (×2), `CONTEXT.md` (×2), `QUICK_START.md`, `.plan.md`, and 13x `readme.md` duplicates across subfolders.

## Recommended action

1. **Accept that the PDF/DOCX priority is complete.** No further ingest required to satisfy Harrison's stated priority.
2. **Optional**: ingest `Trading_Operator_Strategy_Roadmap.pptx` — only file in its format, likely has novel content.
3. **Defer** xlsx/html/md-boilerplate. These were already flagged by the 2026-04-18 lint; treatment unchanged.
4. **Pivot** remaining effort to closing the 30 rule-of-three violations the [[lint-report|2026-04-21 lint]] identified — i.e., add MOC cross-links to under-linked summaries.

## Method notes

- Approach: for each summary in `wiki/summaries/`, extracted the `sources:` frontmatter field (supports both inline `[a, b]` and multi-line `- a` forms). Cross-referenced every import filename against that combined set.
- Limitation: matches on basename substring in `sources:` value. A summary that paraphrases its source in the field (e.g. `sources: ["March 29 audit"]`) wouldn't match. Spot-check against [[import-triage-moc]] for cases like that.
- Coverage of the triage MOC's 79 "needing mapping" bucket: most have been ingested in the weeks since 2026-04-11 (the triage was written; the work happened). The residual 49 are the formats that were never the priority.
