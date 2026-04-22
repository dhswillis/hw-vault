---
created: 2026-04-21
updated: 2026-04-21
type: report
related:
  - wiki/maintenance/dedup-report.md
tags: [maintenance, lint, audit]
---

# Vault Lint Report — 2026-04-21

Automated weekly lint sweep. Supersedes the 2026-04-18 report.

Scanned: 131 wiki pages (87 summaries, 22 concepts, 6 entities, 4 syntheses, 10 MOCs, 2 meta-reports), 224 raw-source files (excluding `assets/`, `claude-data/`, `personal/legal/`).

## 1. Orphans

Zero real orphans. Both meta-reports are expected non-graph nodes.

| Page | Status |
|---|---|
| `wiki/maintenance/lint-report.md` | expected orphan (this file) |
| `wiki/maintenance/dedup-report.md` | expected orphan (meta-report) |

No action. Both are intentionally excluded from the orphan rule per BRAIN.md §hard-rule-3 convention.

## 2. Broken links

Three real broken wiki-links (excluding this report's own citations). All three are carry-overs or edge-cases.

| Source | Line | Link | Suggested fix |
|---|---|---|---|
| `wiki/summaries/tempo-portfolio-v26.md` | 96 | `[[ifvg-two-leg-portfolio-2026-04-12\|2026-04-12 research]]` | Target page does not exist. Either stub `wiki/syntheses/ifvg-two-leg-portfolio-2026-04-12.md` or drop the reference. Carry-over from 2026-04-18 report. |
| `wiki/summaries/tempo-portfolio-v26.md` | 131 | `[[ifvg-two-leg-portfolio-2026-04-12]]` | Same as above. |
| `wiki/summaries/tempo-quick-start-guide.md` | 61 | `[[\|github_ssh_identity]]` | Malformed wikilink — empty target, alias-only. `github_ssh_identity` lives in auto-memory (`~/.claude/projects/-Users-harrisonwillis/memory/`), not the vault. Rewrite as plain text or remove the link syntax. |

False-positives confirmed not broken (handled via pipe-escape parsing): `[[bos-fvg\|BOS_FVG]]`, `[[invalidation-rules\|Rule N]]` × 15 in `audit-history-moc.md` — these are GFM table-cell pipe escapes; targets resolve correctly in Obsidian.

## 3. Stale pages (>90d)

None. Oldest `updated:` date in wiki is within 90 days of 2026-04-21. Vault origin is 2026-04-10.

## 4. Stub / underdeveloped pages

None. All 28 concept/entity pages have bodies ≥ 1800 bytes. Smallest: `wiki/concepts/doji-break.md` at 1871 bytes total (frontmatter included), well above the 500-byte stub threshold.

## 5. Rule-of-three violations

30 summary pages have exactly one inbound link — and in every case that inbound is from `index.md` only. No MOC, concept, entity, or other summary cross-links to them. Per BRAIN.md §hard-rule-6, these should either earn a second inbound or be absorbed into a parent.

| Page | Note |
|---|---|
| `wiki/summaries/ai-trading-show-blueprint.md` | Consider absorbing into `entities/nova-sable-brains.md` or a new `ai-trading-show-moc`. |
| `wiki/summaries/autonomous-trading-system-progress-summary.md` | Pre-Unified-Brain snapshot; thread from `unified-brain-architecture.md`. |
| `wiki/summaries/batch-runner-cluster.md` | Add to `tempo-moc.md` under infrastructure section. |
| `wiki/summaries/big-ride-research.md` | Thread from `research-arc-map.md`. |
| `wiki/summaries/brain-architecture.md` | Thread from `unified-brain-architecture.md` or BRAIN.md/root MOC. |
| `wiki/summaries/conglomerate-ceo.md` | Thread from `unified-brain-architecture.md`. |
| `wiki/summaries/daily-range-research.md` | Thread from `sweep-cluster.md` or `research-arc-map.md`. |
| `wiki/summaries/deep-mine-findings.md` | Thread from `sf-portfolio-cluster.md` (already marked superseded — confirm tag). |
| `wiki/summaries/final-portfolio-spec.md` | Thread from `sf-portfolio-cluster.md`. |
| `wiki/summaries/fvg-entry-timing-analysis.md` | Thread from `concepts/ifvg.md` or `tempo-cluster.md`. |
| `wiki/summaries/ifvg-full-year-verification.md` | Thread from `tempo-ifvg-research.md` (memo variant already flagged for deletion). |
| `wiki/summaries/ifvg-optimization-report.md` | v1 — link from v2 summary as `superseded` forward-link. |
| `wiki/summaries/infrastructure-boilerplate-cluster.md` | Thread from `tempo-project-state.md`. |
| `wiki/summaries/keylevel-sweep-cluster.md` | Thread from `strategies-moc.md` or `sweep-cluster.md`. |
| `wiki/summaries/new-signal-classes-research.md` | Thread from `research-arc-map.md`. |
| `wiki/summaries/nq-portfolio-trading-system.md` | Thread from `sf-portfolio-cluster.md`. |
| `wiki/summaries/nq-sf-engulfing-strategy.md` | v1 — link from v2 as `superseded` forward-link. |
| `wiki/summaries/option4-hybrid-architecture.md` | Thread from `tempo-project-state.md` (flagged as master arch doc in import-triage). |
| `wiki/summaries/personal-legal-cluster.md` | Thread from `personal-moc.md`. |
| `wiki/summaries/remaining-imports-cluster.md` | Thread from `import-triage-moc.md`. |
| `wiki/summaries/sf-portfolio-state.md` | Thread from `sf-portfolio-cluster.md` (already superseded). |
| `wiki/summaries/tempo-ibkr-migration-plan.md` | Thread from `tempo-moc.md` under deployment section. |
| `wiki/summaries/tempo-ifvg-build-spec.md` | Thread from `tempo-ifvg-research.md`. |
| `wiki/summaries/tempo-pipeline-cluster.md` | Thread from `tempo-moc.md`. |
| `wiki/summaries/tempo-strategy-analysis-2026-02-16.md` | Thread from `tempo-moc.md` under pre-Unified-Brain era. |
| `wiki/summaries/trading-system-operations-cluster.md` | Thread from `tempo-project-state.md`. |
| `wiki/summaries/unified-brain-options-and-growth.md` | Thread from `unified-brain-architecture.md`. |
| `wiki/summaries/video-ingestion-pipeline-guide.md` | Thread from `tempo-project-state.md`. |
| `wiki/summaries/wickfade-strategy-findings.md` | Thread from `wick-fade.md` or `wickfade-complete.md`. |
| `wiki/summaries/willis-holdings-strategy-audit.md` | Thread from `unified-brain-architecture.md`. |

**Pattern:** many of these are cluster summaries or superseded precursors that are load-bearing by design — they legitimately serve as single points of aggregation. The cleanest fix is to add one cross-link from the appropriate MOC; creating a third reference for each is unnecessary if the page is clearly a hub.

## 6. Raw sources without summaries

224 raw-source files total. 179 summarized (via `sources:` frontmatter or body references). 1 threaded via `import-triage-moc.md` (`.plan.md`). **44 truly unsummarized and untriaged.**

### Truly unsummarized — actionable

| Folder | Count | Files |
|---|---|---|
| `raw-sources/operations/` | 3 | `2026-04-11-import-manifest.md`, `2026-04-11-import-manifest-inbox-stub.md`, `2026-04-11-vault-map-report.md` |
| `raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/` | 1 | `QUICK_START.md` |
| `raw-sources/imports/2026-04-11/Documents/` (xlsx/html) | 13 | `NQ Test.xlsx`, `NQ_Condition_Matrix.xlsx`, `Tempo_Batch3_Results.xlsx`, `Tempo_Inverse_Analysis.xlsx`, `Tempo_V8f_Strategy_A_Results.xlsx`, `fib_786_strategy_report.html`, `fib_retracement_diagram.html`, `lumi_original_vs_ours.html`, `lumi_vs_ifvg_comparison.html`, `market_vs_limit_dashboard.html`, `nq_fib_bug_tracker.xlsx`, `operator_architecture.html`, `willis_holdings_architecture.html` |
| `raw-sources/imports/2026-04-11/Documents/strategies/tempo/` | 1 | `tempo_portfolio_breakdown.xlsx` |
| `raw-sources/imports/2026-04-11/Documents/trading-system/` | 5 | `NQ Test.xlsx`, `NQ_Condition_Matrix.xlsx`, `Tempo_V8f_Strategy_A_Results.xlsx`, `V6_Portfolio_Tracker.xlsx`, `fib_v2_results.xlsx` |
| `raw-sources/imports/2026-04-11/Downloads/` | 5 | `Trading_Operator_Strategy_Roadmap.pptx`, `architecture_comparison.html`, `infrastructure_architecture.html`, `tempo_progress_dashboard.html`, `tempo_uploader.html` |
| `raw-sources/imports/2026-04-11/Downloads/data/**/margins/` | 13 | 13× `readme.md` stubs under QuantConnect data-folder boilerplate |
| `raw-sources/imports/2026-04-11/trading-system/` | 2 | `NQ Test.xlsx`, `Willis_Holdings_CEO_Schematic.html` |

### Disposition guidance

**High-value — worth ingesting:**
- `raw-sources/operations/2026-04-11-vault-map-report.md` — vault self-analysis.
- `raw-sources/operations/2026-04-11-import-manifest.md` — import provenance.
- `raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/QUICK_START.md` — may duplicate `tempo-quick-start-guide.md`; diff first.
- `raw-sources/imports/2026-04-11/Documents/willis_holdings_architecture.html` + `Willis_Holdings_CEO_Schematic.html` — load-bearing per memory (Haven Park / Harrison's role). Consider promoting.

**Low-value — safe to leave uncovered:**
- 13× QuantConnect `readme.md` margin-data stubs under `Downloads/data/**/margins/` — auto-generated boilerplate.
- Spreadsheet data dumps (`.xlsx`) — numbers are already captured in cluster summaries (e.g., `sf-portfolio-cluster.md`, `audits-cluster.md`).
- HTML dashboard/diagram exports — content already described in parent summary text.
- `tempo_uploader.html`, `tempo_progress_dashboard.html`, `architecture_comparison.html` — ephemeral dashboards.

Carry-over from 2026-04-18: previous report counted 52 unsummarized; today's scan finds 44, suggesting 8 have been summarized in the last 3 days (or resolution differed — this scan catches triage via `import-triage-moc.md` threading).

## 7. Frontmatter hygiene

All 131 wiki pages have well-formed frontmatter with required fields (`created`, `updated`, `type`). No missing-field issues detected.

## 8. Log append-only check

`log.md`: 218 lines, 104 structured headers (`## [YYYY-MM-DD HH:MM]`).

Verification that prior lint report's cited entries remain present:
- `## [2026-04-18 23:30] lint | 14 issues found …` — **present.**
- `## [2026-04-15 17:03] dedup | 1 exact, 1 near, 4 superseded …` — **present.**
- `## [2026-04-14 23:30] dedup | 0 exact, 1 near …` — **present.**
- `## [2026-04-14 11:00] audit | full vault audit — strategy statuses verified …` — **present.**

No evidence of history rewrite. Append-only rule holds.

## 9. Summary — top 5 actions

1. **Fix or stub the broken `ifvg-two-leg-portfolio-2026-04-12` references** at `wiki/summaries/tempo-portfolio-v26.md:96` and `:131`. Either create `wiki/syntheses/ifvg-two-leg-portfolio-2026-04-12.md` or drop the two forward references. Carry-over from 2026-04-18.
2. **Repair the malformed wikilink** at `wiki/summaries/tempo-quick-start-guide.md:61` — `[[|github_ssh_identity]]` has an empty target. Rewrite as plain text since `github_ssh_identity` is an auto-memory file, not a vault page.
3. **Close the rule-of-three gap on 30 summary pages** by adding at least one cross-link from the appropriate MOC (most mapped to `tempo-moc.md`, `tempo-project-state.md`, `unified-brain-architecture.md`, `sf-portfolio-cluster.md`, or `research-arc-map.md`). See §5 table for per-page targets.
4. **Promote the two operations-folder sources** into summaries: `raw-sources/operations/2026-04-11-vault-map-report.md` and `2026-04-11-import-manifest.md`. These capture vault-level state that does not yet have a wiki page.
5. **Execute carry-over dedup actions** from `wiki/maintenance/dedup-report.md`: delete `ifvg-full-year-verification-memo.md`, merge `tempo-v14-corrections` concept + summary, apply `superseded` tag to `ifvg-optimization-report.md`, `nq-sf-engulfing-strategy.md`, `tempo-portfolio-v15.md`, `sf-portfolio-state.md`, `deep-mine-findings.md`, `v8-mining-synthesis.md`, `strategy-mining-report-312d.md`.

## Scan metadata

- Wiki pages scanned: 131
- Summaries: 87 | Concepts: 22 | Entities: 6 | Syntheses: 4 | MOCs: 10 | Meta-reports: 2
- Raw sources scanned: 224
- Stale threshold: 90 days
- Stub threshold: 500 bytes body content
- Previous report: 2026-04-18 (14 issues: 2 orphans, 1 broken, 0 stale, 11 dupes). Delta: orphans 0 (meta-reports now classified as expected), broken 3 (1 carry-over + 1 new malformed link at `tempo-quick-start-guide.md:61` + companion line), stale 0, rule-of-three 30 (not tracked in prior report), unsummarized raw 44 (down from 52).
