---
created: 2026-04-22
type: report
tags: [maintenance, dedup]
---

# Dedup Report — 2026-04-22

Automated scan of `wiki/summaries/`, `wiki/concepts/`, `wiki/entities/`, `wiki/syntheses/` (120 files total). Methodology: frontmatter `sources:` overlap, basename similarity, and fuzzy text matching on first 3,000 body characters. Findings below are grouped by severity. **No files were merged** — these are recommendations only.

## Exact duplicates

### 1. `summaries/ifvg-full-year-verification.md` vs `summaries/ifvg-full-year-verification-memo.md`

Two separate summaries describe the **same underlying source document**. The two `sources:` paths differ:

- `raw-sources/imports/2026-04-11/Documents/trading-system/ifvg-backtest/results/IFVG_Full_Year_Verification_Memo.docx`
- `raw-sources/imports/2026-04-11/Documents/strategies/tempo/results/IFVG_Full_Year_Verification_Memo.docx`

but the two `.docx` files are **binary identical** (MD5 `d57c022d0212a5bc4672e45a1c166b26`, 19,322 bytes). Both summaries cover the same 244-day IFVG MTF Cascade backtest memo, same bug descriptions, same audit findings.

Content differs slightly — the `-memo` version (2026-04-12) is terser and has tighter cross-linking; the un-suffixed version (2026-04-14) is more verbose on R-target findings.

**Recommendation:** Merge into one summary. Keep the un-suffixed `ifvg-full-year-verification.md` (has more quantitative detail on R-targets and the Bug 1/Bug 2 distinction), fold in the callout quote from the memo version, then delete `ifvg-full-year-verification-memo.md`. Update inbound links — only `wiki/maps/tempo-moc.md` links to the memo version; redirect it.

## Near duplicates

### 2. `concepts/tempo-v14-corrections.md` vs `summaries/tempo-v14-corrections.md`

Only pair in the vault with an **identical basename across two directories**. Both cover the same four C# NinjaTrader v14 bugs documented in `raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md`.

Content is **not redundant** — the concept page has standalone detail on each bug (sequence reversed, stop placement, entry price, emergency stop side), while the summary defers to the concept (`"see [[tempo-v14-corrections]] concept page for the full explanation"`) and adds the 30-day corrected backtest numbers. They serve different roles per CLAUDE.md conventions.

**Recommendation:** Keep both, but **rename** one to remove the basename collision. Options: rename the summary to `summaries/tempo-v14-corrections-audit-doc.md` (source-scoped), or rename the concept to `concepts/tempo-v14-bug-cluster.md`. Renaming the summary is lower blast-radius — one summary file, one forward link. This is cosmetic but the collision is confusing when grepping.

## Superseded pages

### 3. `summaries/lumi-strategy-spec.md` → already forward-links to `summaries/lumi-strategy-v2-2026-04-10.md`

The spec page already carries a `> **⚠️ SUPERSEDED 2026-04-16.**` callout pointing to v2. V2 is itself then superseded by `lumi-strategy-v3-2026-04-18.md` (also flagged in-line).

**Recommendation:** Add the `superseded` reserved tag (per CLAUDE.md tags vocabulary) to both `lumi-strategy-spec.md` and `lumi-strategy-v2-2026-04-10.md` frontmatter. Add `related: [lumi-strategy-v3-2026-04-18.md]` to the spec page so the chain is traversable in both directions. No content changes needed — the forward-links already exist in prose.

### 4. `summaries/tempo-portfolio-v15.md` → superseded by `summaries/tempo-portfolio-v26.md`

V26 states `> This is the current production NinjaTrader strategy. Supersedes [[tempo-portfolio-v15|v15]]`. Different source files, different research generations.

**Recommendation:** Add `superseded` tag to `tempo-portfolio-v15.md` frontmatter. No merge — v15's historical numbers are still useful for the audit trail. Purely a tag cleanup.

### 5. `summaries/nq-sf-engulfing-strategy.md` → `summaries/nq-sf-engulfing-strategy-v2.md`

V1 and V2 `.docx` files are distinct (different sizes and hashes). V2 refines the 9-strategy portfolio with explicit Leppyrd ICT framework integration.

**Recommendation:** Add a `> **Superseded by [[nq-sf-engulfing-strategy-v2]].**` callout at the top of the V1 summary, plus `superseded` tag. Cross-link is currently weak — neither page mentions the other in the body.

### 6. `summaries/ifvg-optimization-report.md` → superseded by `summaries/ifvg-optimization-report-v2.md` (and further by `summaries/ifvg-tp-optimization-report.md`)

V1 is a 312-day mining survey from March 9–10, 2026; V2 is a 30-day hard-stop-fix optimization that identified the FVG-edge hard stop and claims near-tripled $/day vs V1 baseline. V1's key claim (no hard stop) is directly contradicted by V2.

**Recommendation:** Add `superseded` tag to V1 and a `> **Superseded by [[ifvg-optimization-report-v2]]**` banner explaining the hard-stop finding. Link V1 from V2 in `related:`. Keep V1 — its 312-day baseline numbers are the reference for V2's gains, and the V8/V9 mining discussion belongs there.

## Split candidates

Pages that collectively cover the same topic from different angles and would benefit from a canonical page, a map of content, or a merge. **None require immediate merging** — each currently serves a distinct source — but the cluster is worth consolidating.

### 7. WickFade family (four summaries + one concept page)

Files:

- `concepts/wick-fade.md` (canonical concept)
- `summaries/15s-wick-fade.md` — 15-second variant, tick-validated
- `summaries/wickfade-complete.md` — "Complete Findings", backtest only, tagged unproven
- `summaries/wickfade-5m-research-findings.md` — 5-minute variant, profitable
- `summaries/wickfade-strategy-findings.md` — best validated config
- `summaries/sweep-fade-research.md` — key-level sweep fade, an adjacent variant

Source docs are distinct (five different files, five different MD5s), so these are not duplicates per se. But together they describe overlapping mean-reversion research with conflicting conclusions (one says $7,452/day profitable, another says "STATUS: UNPROVEN"). The concept page already tries to reconcile this.

**Recommendation:** Create `wiki/maps/wickfade-moc.md`. Structure: orientation paragraph → canonical concept link → four variants with one-line verdicts (tick-validated vs bar-only vs pending) → links to `sweep-cluster.md` and the `bar-sim-trailing-bug` concept. This satisfies the rule of thumb from CLAUDE.md ("create a MOC when you notice yourself typing the same 5 cross-links in multiple wiki pages"). No deletions — all five summaries have unique source material.

### 8. Mining report family (four summaries + one synthesis)

Files:

- `syntheses/mining-reports-v1-v2-reconciliation.md` (exists, does the reconciliation work)
- `summaries/comprehensive-mining-report-v2.md`
- `summaries/strategy-mining-report-312d.md` (the V1)
- `summaries/mining-analysis.md` (V7aa_b cross-tab, tagged contaminated)
- `summaries/v8-mining-synthesis.md`
- `summaries/deep-mine-findings.md` (SF portfolio fib, separate arc)
- `summaries/transcript-mining-findings.md` (not checked in detail, but name suggests overlap)

The V1/V2 reconciliation synthesis already exists, which is the right move. The V8 synthesis and V7 mining-analysis are distinct research generations.

**Recommendation:** No merges. Add all four mining summaries and both syntheses to a new `wiki/maps/mining-research-moc.md` so the research arc is navigable in one place. The `audit-history-moc.md` is adjacent but focuses on audits rather than mining reports. Separately, verify `transcript-mining-findings.md` is genuinely distinct (I did not compare it closely in this scan).

### 9. Brain architecture family

Files:

- `summaries/brain-architecture.md` — sources `raw-sources/trading/BRAIN_ARCHITECTURE.md`
- `summaries/unified-brain-architecture.md` — sources `The_Unified_Brain_Architecture.docx`
- `summaries/unified-brain-options-and-growth.md` — sources `Unified_Brain_Options_and_Growth.docx`

All three source files are distinct. The `brain-architecture.md` (OpenClaw + Ralph Loop + Berman Trifecta, engineering spec) vs `unified-brain-architecture.md` (Willis Holdings v2.0 April 2026) appear to be **parallel architectures** — possibly two competing designs for the same brain system. Without re-reading both in full it's unclear whether they're siblings or one supersedes the other.

**Recommendation:** Read both and write a 1-paragraph reconciliation at the top of the older one explaining the relationship. Likely candidate for a short synthesis page (`syntheses/brain-architectures-reconciliation.md`) if they truly disagree. Low priority — not actively harmful.

### 10. Portfolio family

Files:

- `summaries/final-portfolio-spec.md` — NQ+ES Final Portfolio Spec, +3.34 R/Day
- `summaries/nq-portfolio-trading-system.md` — NQ Portfolio Trading System, 9 strategies, ~987R/year
- `summaries/sf-portfolio-cluster.md` — cluster summary covering both
- `summaries/sf-portfolio-state.md` — March 2026 point-in-time state
- `summaries/tempo-portfolio-v15.md`, `tempo-portfolio-v26.md` — distinct Tempo strategy line (separate arc)

The first four are SF-portfolio arc; the Tempo portfolios are a different research line. The SF cluster already contains the individual summaries. `final-portfolio-spec.md` and `nq-portfolio-trading-system.md` describe different portfolio configurations (NQ+ES vs NQ-only) — not duplicates.

**Recommendation:** No merges. The `sf-portfolio-cluster.md` already plays the organizing role for this family. Confirm that `sf-portfolio-state.md` has a `superseded` tag if the numbers have been replaced by a more recent state file (check `work/projects/` or a newer portfolio doc before tagging).

### 11. Tempo IFVG research / build / audit chain

Files:

- `summaries/tempo-ifvg-research.md` — the foundational research doc (Mar 2026)
- `summaries/tempo-ifvg-build-spec.md` — implementation spec
- `summaries/tempo-ifvg-audit-report.md` — V14 post-emergency-fix validation
- `summaries/ifvg-composite-audit-20260329.md` — canonical late-March composite audit
- `summaries/ifvg-optimization-report.md` / `-v2.md` / `-tp-optimization-report.md` — optimization generations

These are **not duplicates** — each documents a distinct artifact in the IFVG research → build → audit → optimization pipeline. The chain is already structured reasonably well, and `ifvg-composite-audit-20260329.md` is explicitly marked canonical.

**Recommendation:** No merges. Ensure every page in this chain links forward to `ifvg-composite-audit-20260329.md` as the current canonical audit. `tempo-moc.md` probably already organizes this — verify on next MOC update.

## Cluster/individual-summary overlap (informational)

Not a dedup issue per CLAUDE.md conventions — clusters are editorial overviews, individual summaries are drill-downs — but worth noting where individual summary content may have been absorbed into a cluster such that the standalone value is low:

| Cluster | Individual summary it contains | Overlap |
|---|---|---|
| `sweep-cluster.md` | `sweep-fade-research.md` | Cluster lists SWEEP_FADE_RESEARCH.md as one of its 8 sources |
| `audits-cluster.md` | `v8-v9-audit-reports.md` | Cluster's 7 sources include both V8 and V9 audit reports |
| `tempo-cluster.md` | `tempo-ifvg-research.md`, `tempo-rules-v3.md`, `tempo-v14-corrections.md`, `unified-brain-architecture.md`, `tempo-ifvg-build-spec.md`, `tempo-ifvg-audit-report.md`, `tempo-project-state.md` | All contained — this is by design but the individual summaries carry the load-bearing detail |

**Recommendation:** No action. This is the intended pattern. If an individual summary ever becomes redundant with its cluster, prefer retaining the individual summary and trimming the cluster to a source-list-with-links.

## Summary of actions

- **1 merge** (pair #1 — exact duplicate, binary-identical source)
- **1 rename** (pair #2 — basename collision across dirs)
- **4 tag additions** (pairs #3, #4, #5, #6 — add `superseded` tag and forward-link callouts)
- **2 new MOCs recommended** (wickfade, mining-research)
- **0 file deletions** beyond the single merge target

No issues found that require urgent intervention. The vault's existing superseded-callouts-in-prose pattern is working; the main gap is that the `superseded` reserved tag (per CLAUDE.md) is not being applied in frontmatter where prose already marks a page as outdated.

## Scan caveats

- Automated text similarity (SequenceMatcher on normalized first 3,000 chars) was calibrated to flag `>60%` overlap but did not find any at that threshold — pages diverge in heading structure even when covering the same ground. The `split candidate` findings above are from manual review of suspect clusters, not from text-similarity alone.
- `transcript-mining-findings.md` was not read in full; re-scan if it shows up in the next `/lint`.
- Did not compare `work/` or `personal/` folders — scope was `wiki/` per the `/dedup` spec.
