---
created: 2026-04-14
updated: 2026-04-15
type: report
related: []
tags: [maintenance, dedup]
---

# Dedup Report — 2026-04-15

Scanned: `wiki/summaries/` (76 files), `wiki/concepts/` (21 files), `wiki/entities/` (6 files), `wiki/syntheses/` (5 files). Total: 108 wiki pages + 10 MOCs.

Previous scan: 2026-04-14. This report supersedes it entirely.

---

## Exact Duplicates

### 1. `wiki/summaries/ifvg-full-year-verification-memo.md` ↔ `wiki/summaries/ifvg-full-year-verification.md`

**Classification: EXACT DUPE (same source, memo is strict subset)**

Both reference the same raw source: `IFVG_Full_Year_Verification_Memo.docx`. The memo version (created 2026-04-12, 19 lines) is a 100-word teaser. The verification version (updated 2026-04-14, 31 lines) is a substantive summary with R-target findings and key takeaway sections. Every fact in the memo appears in the verification page.

**Recommendation:** Delete `ifvg-full-year-verification-memo.md`. Update any inbound links (currently `ifvg-optimization-report-v2.md` `related:` field) to point to `ifvg-full-year-verification.md`.

---

## Near Duplicates

### 2. `wiki/summaries/tempo-v14-corrections.md` ↔ `wiki/concepts/tempo-v14-corrections.md`

**Classification: NEAR DUPE — same filename, ~70% text overlap. Carried forward from 2026-04-14 report.**

Both cover the four v14 C# bugs and the 30-day corrected backtest. Both reproduce the same bug table and `| Slice | Trades | WR | Pts/day | Calmar |` results. The concept page adds ~500 words of deeper technical prose (fix logic, bug class comparison to [[bar-sim-trailing-bug]]). The summary front-loads source context and the "historical artifact" status.

**Recommendation (unchanged):** Merge into the concept page. The concept is more technically complete. Move the summary's status note into a "Status / caveats" section on the concept. Delete the summary. Update 7 inbound wikilinks.

---

## Superseded Pages

### 3. `wiki/summaries/sf-portfolio-state.md` → superseded by `wiki/summaries/sf-portfolio-cluster.md`

**Carried forward from 2026-04-14.** `sf-portfolio-state.md` (March 20, 12 legs, +2.26 R/day) is a point-in-time snapshot. The cluster covers the same source file with significantly more detail including the tick-level verification and updated R/day assessment.

**Recommendation:** Add `superseded` tag and a banner pointing to `sf-portfolio-cluster.md` and `final-portfolio-spec.md` (March 21 upgrade, +3.34 R/day). Do not delete — useful as a search entry point.

### 4. `wiki/summaries/deep-mine-findings.md` → superseded by `wiki/summaries/sf-portfolio-cluster.md`

**Carried forward from 2026-04-14.** The cluster covers the same source under "Deep Mine Findings" with equivalent content. Individual summary has marginally more detail on three breakthrough filters.

**Recommendation:** Tag `superseded`. Forward-link to cluster. Consider adding the three filter names to the cluster's DEEP_MINE section before tagging.

### 5. `wiki/summaries/v8-mining-synthesis.md` → partially superseded by `wiki/summaries/sf-portfolio-cluster.md`

**Carried forward from 2026-04-14.** Cluster covers the same verdict (dead, V10i invalidated). Individual summary has unique tier breakdown and Monte Carlo methodology detail.

**Recommendation:** Tag `superseded` but keep for methodology detail. Forward-link to cluster.

### 6. `wiki/summaries/strategy-mining-report-312d.md` → superseded by `wiki/summaries/comprehensive-mining-report-v2.md`

V1-era (312 days, V10i-contaminated) vs V2-era (258 days, corrected). The summary already self-references the V2 report and the reconciliation synthesis. Both tagged `suspect-results`.

**Recommendation:** Keep as historical artifact. Add explicit banner: "HISTORICAL ARTIFACT — V1 era. See [[comprehensive-mining-report-v2]] for corrected version and [[mining-reports-v1-v2-reconciliation]] for comparison." The supersession chain is the point of the page.

---

## Split Candidates (Healthy — No Merge Needed)

### 7. `wiki/summaries/tempo-cluster.md` — parent of 7 individual summaries

14-source cluster. Seven sources now have dedicated individual summaries (tempo-v14-corrections, ifvg-tp-optimization-report, unified-brain-architecture, unified-brain-options-and-growth, tempo-ibkr-migration-plan, lumi-strategy-spec, tempo-project-state). Healthy architecture.

**Recommendation:** Ensure cluster has wikilinks to all 7 individual summaries as navigation index.

### 8. `wiki/summaries/sweep-cluster.md` — stale on 3 of 8 sources

Three files now have individual summaries (15s-wick-fade, wickfade-complete, sweep-fade-research). **The cluster's wick-fade paragraphs are materially stale** — they say "treat as unread, not ingested" but both were ingested 2026-04-12 and found tick-level validated.

**Recommendation:** Update cluster to link individual summaries and correct the stale wick-fade narrative.

### 9. `wiki/summaries/audits-cluster.md` — parent of 3 individual summaries

Cluster covers 7 sources; three have dedicated summaries (look-ahead-audit, backtest-integrity-audit-v2, v8-v9-audit-reports). Cluster's contamination chronology table is unique value.

**Recommendation:** Add wikilinks from cluster entries to individual summaries. Keep both layers.

### 10. `wiki/summaries/portfolio-audit-v2-sweep.md` ↔ `wiki/summaries/sweep-cluster.md`

Individual summary covers BIP routing and 7 fixed bugs in TempoPortfoliov1.cs — detail not in the cluster.

**Recommendation:** Keep both. Add `[[portfolio-audit-v2-sweep]]` link in cluster.

---

## Contradictions Requiring Investigation

### 11. SWEEP_FADE_RESEARCH headline number discrepancy — HIGH PRIORITY

**Carried forward from 2026-04-14. Still unresolved.**

| Page | Created | Headline result |
|---|---|---|
| `wiki/summaries/sweep-cluster.md` | 2026-04-11 | "+0.29 R/day, marginal" |
| `wiki/summaries/sweep-fade-research.md` | 2026-04-12 | "+10.43 R/day, Calmar 361.8, 100% winning weeks" |

36x discrepancy on the same source file. Possible explanations: cluster was an 8-file batch (may have cited earlier/different configuration); individual summary was a focused single-file read. Per authority hierarchy, most recent ingest is higher-trust, but the magnitude warrants manual re-read of `raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md`.

**Recommendation:** Re-read the raw source. Update whichever summary is wrong. Add reconciliation note to both pages.

### 12. Source path mismatch — BOS_FVG_FAILURE_CONSOLIDATED.md

**Carried forward from 2026-04-14.**

`bos-fvg-failure-consolidated.md` and `research-arc-map.md` use `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md`. `audits-cluster.md` uses `raw-sources/trading/audits/BOS_FVG_FAILURE_CONSOLIDATED.md`. May be a copy vs symlink issue.

**Recommendation:** Verify whether both paths point to the same file. Standardize `sources:` references vault-wide.

---

## Minor Flags

### 13. `wiki/summaries/context-engine-cluster.md` — internal source duplication

`sources:` array lists `QUICK_REFERENCE.md` at two different paths under `raw-sources/imports/`. Acknowledged in the summary text. Likely an import artifact.

**Recommendation:** Verify the two files are identical. Remove duplicate from `raw-sources/` and `sources:` frontmatter.

### 14. `suspect-results` pages — verify "Why tagged suspect" sections

Six pages carry `suspect-results`: nq-playbook, research-journal, mining-analysis, strategy-mining-report-312d, wickfade-complete, sweep-fade-research. Each should have a clear "Why this is tagged suspect" section specifying which contamination applies, which numbers are affected, and which are safe to cite.

**Recommendation:** Audit all 6 pages. Add/improve the suspect explanation section where missing or unclear.

---

## Summary

| Category | Count | Change from 2026-04-14 |
|---|---|---|
| Exact duplicates | 1 | +1 (ifvg-full-year-verification-memo) |
| Near duplicates | 1 | 0 (tempo-v14-corrections, still unresolved) |
| Superseded pages | 4 | +1 (strategy-mining-report-312d explicitly flagged) |
| Split candidates (healthy) | 4 | 0 |
| Contradictions | 2 | 0 (both still unresolved from last scan) |
| Minor flags | 2 | 0 |
| **Total findings** | **14** | **+2 net new** |

**Priority actions for next session:**

1. Delete `ifvg-full-year-verification-memo.md` (exact dupe)
2. Merge `tempo-v14-corrections` summary into concept page
3. Re-read `SWEEP_FADE_RESEARCH.md` raw source to resolve +0.29 vs +10.43 contradiction
4. Update `sweep-cluster.md` stale wick-fade narrative
