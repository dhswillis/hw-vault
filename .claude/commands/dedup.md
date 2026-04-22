---
description: Scan wiki for duplicate or near-duplicate pages and flag them
---

# /dedup

Scan the wiki for duplicate or near-duplicate content and produce a dedup report.

## Steps

1. **Scan** all files in `wiki/summaries/`, `wiki/concepts/`, `wiki/entities/`, and `wiki/syntheses/`.

2. **For each pair of files**, compute similarity:
   - Compare titles (basename similarity)
   - Compare `sources:` frontmatter (overlapping source references = likely dupes)
   - Compare first 500 characters of body text (fuzzy match)
   - Compare `tags:` overlap

3. **Flag** pairs that meet any of these criteria:
   - **Exact dupe**: same `sources:` array AND >80% text overlap
   - **Near dupe**: >60% text overlap OR same basename with different paths
   - **Superseded**: one page's `sources:` is a strict subset of another's
   - **Split candidate**: two pages cover different aspects of the same source (should be merged)

4. **Write** the report to `wiki/maintenance/dedup-report.md`:
   ```markdown
   ---
   created: YYYY-MM-DD
   type: report
   tags: [maintenance, dedup]
   ---

   # Dedup Report — YYYY-MM-DD

   ## Exact duplicates
   (list with both paths and recommendation)

   ## Near duplicates
   (list with similarity score and merge recommendation)

   ## Superseded pages
   (list with forward-link recommendation)

   ## Split candidates
   (list with merge recommendation)
   ```

5. **Do NOT auto-merge** — only report. The user decides what to merge.

6. **Append** to `log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] dedup | N exact, N near, N superseded, N splits
   ```
