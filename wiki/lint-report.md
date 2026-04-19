---
created: 2026-04-18
updated: 2026-04-18
type: report
related:
  - wiki/dedup-report.md
tags: [maintenance, lint, audit]
---

# Vault Lint Report — 2026-04-18

Automated weekly lint + dedup scan. Previous report: 2026-04-14 (superseded entirely by this one).

Scanned: 128 wiki pages (85 summaries, 21 concepts, 6 entities, 4 syntheses, 10 MOCs, 2 meta-reports), 224 raw sources, 318 total vault markdown files.

## Orphans

Two pages have no inbound `[[wikilinks]]`:

- `wiki/dedup-report.md` — meta-report, expected orphan
- `wiki/lint-report.md` — meta-report, expected orphan (this file)

**Status:** acceptable. Meta-reports are entry points, not graph nodes. Consider adding both to `index.md` under a `## Maintenance` section so they appear in the flat catalog.

## Broken links

Four unresolved `[[link]]` targets found; three are false positives. One real broken link:

| Source | Link | Status |
|---|---|---|
| `wiki/summaries/tempo-portfolio-v26.md` (lines 96, 131) | `[[ifvg-two-leg-portfolio-2026-04-12]]` | **Real — page does not exist.** Describes 2026-04-12 autonomous research cited as "wrong baseline" in v26. |
| `wiki/concepts/ifvg.md`, `wiki/entities/ninjatrader-v5.md`, `wiki/entities/tempo-methodology.md` | `[[raw-sources/imports/2026-04-11/Documents/.plan]]` | False positive — `.plan.md` exists (dot-prefix hidden from default scans). |
| `wiki/concepts/ifvg.md`, `wiki/summaries/comprehensive-mining-report-v2.md`, `wiki/maps/audit-history-moc.md` | `[[bos-fvg\|...]]`, `[[invalidation-rules\|...]]` | False positives — backslash-escaped pipes inside Markdown tables; targets resolve correctly in Obsidian. |

**Action:** Create stub `wiki/syntheses/ifvg-two-leg-portfolio-2026-04-12.md` or remove the forward references in `tempo-portfolio-v26.md`.

## Stale pages (>90d)

None. Every wiki page with a parseable `updated:` date is ≤ 90 days old. Oldest wiki page: 2026-04-10 (8 days).

## Missing summaries

52 raw-source files have no corresponding `wiki/summaries/*.md`:

- 22 markdown (mostly `CONTEXT.md`, `CLAUDE_BOOTSTRAP.md`, `BOOTSTRAP.md` config files and QuantConnect `readme.md` data-folder stubs — low ingestion value)
- 15 `.xlsx` (data dumps: `NQ_Mining_Results_V2.xlsx`, `Tempo_V8f_Strategy_A_Results.xlsx`, `V6_Portfolio_Tracker.xlsx`, etc. — some referenced indirectly through cluster summaries)
- 12 `.html` (operator dashboards, fib diagrams, architecture diagrams)
- 2 `.json`, 1 `.pptx`

Candidates worth ingesting:

- `raw-sources/operations/2026-04-11-vault-map-report.md` — vault self-analysis, likely relevant metadata
- `raw-sources/operations/2026-04-11-import-manifest.md` — original import provenance
- `raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/QUICK_START.md` — may duplicate `tempo-quick-start-guide.md` summary

The rest (CONTEXT/BOOTSTRAP/readme stubs, spreadsheet data dumps, HTML dashboards) can be left unsummarized — they are referenced/subsumed by the existing cluster summaries.

## Duplicate / near-duplicate pages

44 pairwise findings across 85 summary files. Most are **healthy cluster → child relationships** (expected per vault design):

### Known issues (carried over from 2026-04-15 dedup report)

1. **`ifvg-full-year-verification-memo.md` ↔ `ifvg-full-year-verification.md`** (ratio 0.92, same source) — **EXACT DUPE.** Memo is a strict subset. Recommendation unchanged: delete the memo.
2. **`tempo-v14-corrections.md` (concept) ↔ `tempo-v14-corrections.md` (summary)** — same filename, two file types, ~70% text overlap. Recommendation unchanged: merge summary into concept.
3. **`sf-portfolio-cluster.md` ↔ `sf-portfolio-state.md`** (sources-subset) — superseded; tagged.
4. **`sf-portfolio-cluster.md` ↔ `v8-mining-synthesis.md`** (sources-subset) — superseded; tagged.
5. **`strategy-mining-report-312d.md` ↔ `comprehensive-mining-report-v2.md`** — superseded; tagged.

### Near-duplicate titles flagged (new or reconfirmed)

| Pair | Ratio | Verdict |
|---|---|---|
| `ifvg-optimization-report.md` ↔ `ifvg-optimization-report-v2.md` | 0.94 | Versioned — v2 supersedes v1. Consider tagging v1 `superseded`. |
| `ifvg-optimization-report-v2.md` ↔ `ifvg-tp-optimization-report.md` | 0.89 | Distinct focus (general optimization vs. TP-specific). Keep both; cross-link. |
| `lumi-es-strategy-v2-2026-04-10.md` ↔ `lumi-strategy-v2-2026-04-10.md` | 0.95 | Possible near-duplicate from a re-ingest. Verify and merge if same source. |
| `lumi-strategy-v2-2026-04-10.md` ↔ `lumi-strategy-v3-2026-04-18.md` | 0.93 | Versioned (v2 and v3 ingested 8 days apart). Tag v2 `superseded`; forward-link to v3. |
| `nq-sf-engulfing-strategy.md` ↔ `nq-sf-engulfing-strategy-v2.md` | 0.94 | Versioned. Confirm v1 is `superseded`. |
| `tempo-context-engine-spec-v1-1-addendum.md` ↔ `tempo-context-engine-spec-v1-2-addendum.md` | 0.97 | Two addenda to same spec. Keep both; link from parent spec. |
| `tempo-portfolio-v15.md` ↔ `tempo-portfolio-v26.md` | 0.89 | Distinct versions (v15 = 6-strategy, v26 = latest). Keep both; confirm v15 has `superseded` tag if retired. |

### Cluster-subset relationships (expected, no action)

33 `sources-subset` findings where an individual summary's sources are a subset of a cluster summary (e.g., `tempo-cluster.md` contains 7 children, `sweep-cluster.md` contains 5, `sf-portfolio-cluster.md` contains 3, `audits-cluster.md` contains 4). Per dedup-report §7-10, this is **intentional architecture** — clusters are parent aggregators, children are focused views. No action.

## Superseded pages

Four pages currently carry the `superseded` tag:

- `wiki/summaries/bos-fvg-failure-consolidated.md`
- `wiki/summaries/lumi-strategy-spec.md`
- `wiki/summaries/lumi-strategy-v2-2026-04-10.md`
- `wiki/summaries/nq-playbook.md`

**Additional candidates** (from dupe analysis above) that should receive the tag:

- `wiki/summaries/ifvg-optimization-report.md` (v1 → v2)
- `wiki/summaries/nq-sf-engulfing-strategy.md` (v1 → v2; confirm status)
- `wiki/summaries/tempo-portfolio-v15.md` (v15 → v26; confirm whether v15 is retired or still active)
- `wiki/summaries/sf-portfolio-state.md`, `deep-mine-findings.md`, `v8-mining-synthesis.md` (per dedup-report, pending tag application)
- `wiki/summaries/strategy-mining-report-312d.md` (per dedup-report §6)

## Recommendations

Priority actions for the next wrap-up or dedup session:

1. **Create or remove the missing `ifvg-two-leg-portfolio-2026-04-12` reference.** Either stub the page or edit `tempo-portfolio-v26.md` to drop the broken links.
2. **Execute the carried-over dedup actions** from `wiki/dedup-report.md`: delete `ifvg-full-year-verification-memo.md`, merge `tempo-v14-corrections` concept + summary, apply `superseded` tag to the five pages listed under "additional candidates" above.
3. **Resolve SWEEP_FADE_RESEARCH headline contradiction** (+0.29 vs +10.43 R/day) — still unresolved from 2026-04-14 dedup scan. Re-read `raw-sources/trading/sweep/SWEEP_FADE_RESEARCH.md`.
4. **Investigate `lumi-es-strategy-v2-2026-04-10.md` vs `lumi-strategy-v2-2026-04-10.md`** — ratio 0.95 title similarity suggests a re-ingest duplicate. Diff the two and merge if they cover the same source.
5. **Clarify v15 vs v26 relationship** in the tempo portfolio — if v15 is retired, tag it `superseded` and forward-link to v26; if both are active (different portfolios), add a clarifying note to both headers.
6. **Add lint + dedup reports to `index.md`** under a `## Maintenance` section so the meta-reports become discoverable from the flat catalog.
7. **Ingest `raw-sources/operations/2026-04-11-vault-map-report.md` and `2026-04-11-import-manifest.md`** if they contain substantive content beyond import provenance.

## Scan metadata

- Wiki pages scanned: 128
- Summary pages: 85
- Concept pages: 21
- Entity pages: 6
- Synthesis pages: 4
- MOC pages: 10
- Raw sources: 224
- Stale threshold: 90 days
- Title similarity threshold: 0.85 (Levenshtein ratio)
- First-500-char text overlap threshold: 0.50
- Previous scan: 2026-04-14 (reported 0 orphans, 0 broken links, 0 stale; this scan adds 1 real broken link and 7 near-duplicate title flags the prior scan did not enumerate)
