---
created: 2026-04-14
updated: 2026-04-14
type: report
related: []
tags: [maintenance, dedup]
---

# Dedup Report — 2026-04-14

Scanned: `wiki/summaries/` (75 files), `wiki/concepts/` (17 files), `wiki/entities/` (5 files), `wiki/syntheses/` (4 files). Total: 101 wiki pages.

---

## Exact Duplicates

**None found.** No two pages have both identical `sources:` arrays and >80% body text overlap.

---

## Near Duplicates

### 1. `wiki/summaries/tempo-v14-corrections.md` ↔ `wiki/concepts/tempo-v14-corrections.md`

**Similarity: ~70% — SAME FILENAME in different directories.**

Both pages cover the four critical v14 C# implementation bugs and the 30-day corrected backtest. Both reproduce the same bug table, the same `| Slice | Trades | WR | Pts/day | Calmar |` results table, the same London-problem analysis, and the same Lumi verdict. The cross-links confirm awareness: the summary says "see [[tempo-v14-corrections]] concept page for the full explanation."

Differences: the concept page adds ~500 words of deeper technical prose on each bug mechanism (e.g. the `max(wick+1, entry+3)` fix logic, the "inverted FVG" naming etymology) and an explicit comparison to the [[bar-sim-trailing-bug]] bug class. The summary front-loads the source context and the 2026-04-10 "historical artifact" status.

**Recommendation:** The split is intentional but the shared filename is confusing — Obsidian searches for `tempo-v14-corrections` will return both, and wikilinks are ambiguous unless path-qualified. Consider:
- Option A (preferred): Rename the summary to `tempo-v14-corrections-findings.md` and update all inbound wikilinks (7 pages reference it).
- Option B: Merge the two into the concept page with a "Source summary" section at the top, and redirect the summary to a stub.

---

## Superseded Pages

### 2. `wiki/summaries/sf-portfolio-state.md` → superseded by `wiki/summaries/sf-portfolio-cluster.md`

`sf-portfolio-state.md` (created 2026-04-12) is a brief summary of `raw-sources/trading/sf-portfolio/PORTFOLIO_STATE_20260320.md`. The cluster summary `sf-portfolio-cluster.md` (created 2026-04-11) covers that SAME source file as one of its six cluster documents — and provides significantly more detail, including the tick-level code verification, the 8-bug list, and the updated R/day assessment.

`sf-portfolio-state.md` already acknowledges this: "The SF Portfolio Cluster has the updated assessment after tick-level verification." The individual summary adds nothing the cluster does not cover more accurately.

**Recommendation:** Tag `sf-portfolio-state.md` as `superseded`, add `related: [wiki/summaries/sf-portfolio-cluster.md]`, and add a note pointing forward to the cluster. Do NOT delete — the cluster's `PORTFOLIO_STATE` section can be reached more directly from search via the individual summary.

---

### 3. `wiki/summaries/deep-mine-findings.md` → superseded by `wiki/summaries/sf-portfolio-cluster.md`

`deep-mine-findings.md` (2026-04-12) covers `raw-sources/trading/sf-portfolio/DEEP_MINE_FINDINGS.md`. The cluster summary `sf-portfolio-cluster.md` explicitly covers this same file under "Deep Mine Findings (Fib retracement — dead strategy confirmation)" with equivalent content (the 96,552 fib trade count, the dead-strategy verdict, the usable negative confirmation).

The individual summary is marginally more specific (names the three breakthrough filters: No-Poison Candle, EMA21 Direction, Minimum Leg Size; gives the +7.26 R/day top config). The cluster treats it more briefly and reaches the same conclusion.

**Recommendation:** Tag `deep-mine-findings.md` as `superseded`, forward-link to `sf-portfolio-cluster.md`. The unique detail (three filter names and the suspect-results caveat on B/E management) could be preserved by adding one bullet to the cluster summary's DEEP_MINE section before tagging.

---

### 4. `wiki/summaries/v8-mining-synthesis.md` → partially superseded by `wiki/summaries/sf-portfolio-cluster.md`

`v8-mining-synthesis.md` (2026-04-12) covers `raw-sources/trading/sf-portfolio/V8_MINING_SYNTHESIS.md`. The cluster `sf-portfolio-cluster.md` also covers this file, under "V8 Mining Synthesis (separately tagged suspect)," with substantively the same verdict (dead, V10i look-ahead invalidates everything). The cluster's section is actually more concise than the individual summary.

However, the individual summary adds unique value: the specific breakdown of three execution model tiers (Tier 1/2/3, Tier 2 at 90% weekly WR), the Monte Carlo methodology (10K runs), and the 174-train/75-test walk-forward split.

**Recommendation:** NOT fully superseded — the tier breakdown and methodology detail are worth keeping. Tag `v8-mining-synthesis.md` with `superseded` frontmatter but add a note that the tier detail is unique. Add forward-link to cluster.

---

### 5. `wiki/summaries/nq-sf-engulfing-strategy-v2.md` — overlaps with `wiki/summaries/sf-portfolio-cluster.md`

`nq-sf-engulfing-strategy-v2.md` covers `raw-sources/trading/sf-portfolio/NQ_SF_Engulfing_Strategy_V2.docx`, which is listed as one of the six sf-portfolio cluster source files. The individual summary (172 words) is notably more detailed on the Leppyrd ICT framework mechanics than the cluster's brief mention. The cluster does not summarize V2 in depth — it lists the file but marks it "(binary)" pending conversion.

**Recommendation:** NOT superseded — the individual summary provides detail the cluster does not. Update the cluster to note that V2 is now individually summarized at `wiki/summaries/nq-sf-engulfing-strategy-v2.md`.

---

## Split Candidates

### 6. `wiki/summaries/tempo-cluster.md` (14 sources) — parent of multiple individual summaries

The `tempo-cluster.md` covers 14 source files from `raw-sources/trading/tempo/`. At least seven of those source files now have their own dedicated individual summaries, created in a subsequent ingest session (2026-04-12):

| Cluster source file | Individual summary |
|---|---|
| `TEMPO_V14_CORRECTIONS.md` | `wiki/summaries/tempo-v14-corrections.md` |
| `IFVG_TP_Optimization_Report.docx` | `wiki/summaries/ifvg-tp-optimization-report.md` |
| `The_Unified_Brain_Architecture.docx` | `wiki/summaries/unified-brain-architecture.md` |
| `Unified_Brain_Options_and_Growth.docx` | `wiki/summaries/unified-brain-options-and-growth.md` |
| `Tempo_IBKR_Migration_Plan.docx` | `wiki/summaries/tempo-ibkr-migration-plan.md` |
| `LUMI_STRATEGY_SPEC.md` | `wiki/summaries/lumi-strategy-spec.md` |
| `TEMPO_PROJECT_STATE.md` | `wiki/summaries/tempo-project-state.md` |

The cluster covers these files at summary level; individual summaries cover them more deeply. Both layers are useful — the cluster gives the "14-file master view" while individual summaries let readers go deep on one file. This is healthy architecture, not a bug.

**Recommendation:** Do NOT merge. Instead, update `tempo-cluster.md` to add wikilinks pointing to the individual summaries that now exist (e.g. add a "See [[unified-brain-architecture]]" note next to each cluster file entry that has a dedicated page). This makes the cluster a true navigation index rather than a self-contained doc.

---

### 7. `wiki/summaries/sweep-cluster.md` — stale for three of its source files

`sweep-cluster.md` covers eight sources including:
- `15S_Wick_Fade_Findings.docx` — since ingested individually as `wiki/summaries/15s-wick-fade.md`
- `WickFade_Complete_Findings.docx` — since ingested individually as `wiki/summaries/wickfade-complete.md`
- `SWEEP_FADE_RESEARCH.md` — since ingested individually as `wiki/summaries/sweep-fade-research.md`

The cluster's treatment of the two wick-fade docx files explicitly says "treat wick fade claims as 'unread, not ingested'" — which was accurate when written (2026-04-11) but is now stale (both were ingested 2026-04-12 and found to be tick-level validated, canonical results). The cluster's narrative about these files is materially incorrect.

**Recommendation:** Update `sweep-cluster.md` to add wikilinks to the three individual summaries and revise the wick-fade paragraphs to reflect that they are now ingested and validated. No merging needed — the cluster's cross-file narrative value (comparing sweep variants) is distinct from the individual summaries.

---

### 8. `wiki/summaries/audits-cluster.md` — parent of `look-ahead-audit.md`, `backtest-integrity-audit-v2.md`, and `v8-v9-audit-reports.md`

`audits-cluster.md` covers 7 sources. Three of those sources now have their own dedicated individual summaries that are substantively more detailed:

| Cluster source | Individual summary |
|---|---|
| `LOOK_AHEAD_AUDIT_2026-02-18.md` | `wiki/summaries/look-ahead-audit.md` (3 bugs, methodology lessons, 80 lines) |
| `BACKTEST_INTEGRITY_AUDIT_V2.docx` | `wiki/summaries/backtest-integrity-audit-v2.md` (13 findings by severity) |
| `V8_Full_Audit_Report.md` + `V9_Full_Audit_Report.md` | `wiki/summaries/v8-v9-audit-reports.md` (bug taxonomy, what-not-to-cite list) |

The individual summaries add significant value beyond the cluster. Healthy split-candidate architecture.

**Recommendation:** Do NOT merge. Update `audits-cluster.md` chronology table to add wikilinks pointing to the individual summaries where they exist. The cluster's "chronology of contamination findings" table is unique value not replicated in any individual summary — keep it.

---

### 9. `wiki/summaries/portfolio-audit-v2-sweep.md` — split with `wiki/summaries/sweep-cluster.md`

`portfolio-audit-v2-sweep.md` covers `raw-sources/trading/sweep/PORTFOLIO_AUDIT_V2.md`, listed in `sweep-cluster.md` as "Second-pass portfolio audit. Overlaps with SF portfolio audit work." The individual summary (code verification of 7 bugs in TempoPortfoliov1.cs) is substantively different from the brief mention in the cluster — they cover different aspects of the same source file.

**Recommendation:** Keep both. The individual summary's content (specific BIP routing, 7 fixed bugs) is not in the cluster. Update `sweep-cluster.md` to add `[[portfolio-audit-v2-sweep]]` as a link next to `PORTFOLIO_AUDIT_V2.md`.

---

## Contradictions Requiring Investigation

### 10. ⚠️ SWEEP_FADE_RESEARCH.md headline number discrepancy — HIGHEST PRIORITY

Two summaries of the SAME source file (`raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md`) report completely different headline numbers:

| Page | Created | Headline result |
|---|---|---|
| `wiki/summaries/sweep-cluster.md` | 2026-04-11 | "+0.29 R/day, marginal — not dead but not tradeable" |
| `wiki/summaries/sweep-fade-research.md` | 2026-04-12 | "+10.43 R/day, Calmar 361.8, 100% winning weeks" |

These are irreconcilable without re-reading the source. Possible explanations:
- The cluster was written before the file was fully read (the cluster was an 8-file ingest; the individual summary a single-file ingest the next day)
- The source file has a complex structure with multiple strategies; the cluster may have cited an earlier, less-optimized configuration
- The April 12 individual summary may have misidentified the headline result

Given the authority hierarchy in CLAUDE.md (most recent ingest > earlier cluster), `sweep-fade-research.md` is likely more accurate. But given the magnitude of the discrepancy (+0.29 vs +10.43 R/day), manual verification is warranted before acting on either number.

**Recommendation:** Re-read `raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md` directly and verify which number is correct. Update whichever summary is wrong. Add a note to `sweep-cluster.md` pointing to `[[sweep-fade-research]]` as the more detailed individual summary.

---

### 11. ⚠️ Source path mismatch — `BOS_FVG_FAILURE_CONSOLIDATED.md` and `BOS_FVG_10PT_AUDIT.md`

These two files appear with two different path prefixes across the wiki:

| Summary | Path used |
|---|---|
| `wiki/summaries/bos-fvg-failure-consolidated.md` | `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md` |
| `wiki/summaries/audits-cluster.md` | `raw-sources/trading/audits/BOS_FVG_FAILURE_CONSOLIDATED.md` |
| `wiki/syntheses/research-arc-map.md` | `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md` |
| `wiki/summaries/audits-cluster.md` | `raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md` |
| `wiki/summaries/bos-fvg-10pt-audit.md` | `raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md` |
| `wiki/syntheses/bos-fvg-claim-vs-reality.md` | `raw-sources/trading/audits/BOS_FVG_10PT_AUDIT.md` |

The `BOS_FVG_FAILURE_CONSOLIDATED.md` path is inconsistent. Either there are copies in both locations (duplication in `raw-sources/`), or `bos-fvg-failure-consolidated.md` and `research-arc-map.md` point to a different copy than `audits-cluster.md`. This would not affect wiki quality directly but could cause stale reads if the two copies diverge.

**Recommendation:** Check whether `raw-sources/trading/audits/BOS_FVG_FAILURE_CONSOLIDATED.md` and `~/Documents/trading-system/results/BOS_FVG_FAILURE_CONSOLIDATED.md` are the same file (symlink) or different copies. If different, determine which is canonical and standardize all `sources:` references.

---

## Additional Notes

### 12. `context-engine-cluster.md` — internal source duplication

The cluster's `sources:` array lists `QUICK_REFERENCE.md` at two different paths:
- `raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/QUICK_REFERENCE.md`
- `raw-sources/imports/2026-04-11/Documents/trading-system/context-engine/context-engine/QUICK_REFERENCE.md`

The summary acknowledges this ("×2, duplicates across paths"). Likely the same file copied during an import pass.

**Recommendation:** Check whether the two are identical. If yes, remove the duplicate from `raw-sources/` and from the `sources:` frontmatter.

---

### 13. `wiki/summaries/tempo-pipeline-cluster.md` — self-flagged staleness

The cluster explicitly notes its V2 extraction intermediates (`tempo_rules_summary.md`, `tempo_rules_from_videos.md`, `V2_IMPLEMENTATION_SUMMARY.md`, `README_V2.md`) are "superseded by V3." No action needed on the wiki pages themselves (the individual V2 intermediates don't have their own summaries), but the cluster's `tags:` could gain `superseded` for those specific source files to match the tag vocabulary.

---

## Summary Count

| Category | Count |
|---|---|
| Exact duplicates | 0 |
| Near duplicates | 1 |
| Superseded pages | 3 (confirmed) + 1 (partial) |
| Split candidates (healthy, no merge needed) | 4 |
| Contradictions requiring investigation | 2 |
| Minor flags (path issues, internal dupe) | 2 |
