---
description: Ingest a raw source file — create summary, concepts, entities, cross-links
---

# /ingest

Ingest one or more files from `raw-sources/` into the wiki. This is the primary knowledge pipeline.

## Usage

```
/ingest raw-sources/trading/NEW_FILE.md
/ingest              (ingests all un-summarized files in raw-sources/)
```

## Steps

Follow `CLAUDE.md` § Operations § Ingest exactly. Key steps:

1. **Read every source in full** before writing anything.

2. **Write a summary** in `wiki/summaries/<source-slug>.md` with proper frontmatter.

3. **Apply rule of three**: create concept/entity pages for anything mentioned 3+ times across the ingest batch + existing wiki, OR once but load-bearing.

4. **Cross-link liberally** with `[[double brackets]]`. Every summary links to every concept/entity it introduces.

5. **Check for cross-source synthesis**: when sources contradict, complement, or update each other, write `wiki/syntheses/<slug>.md`.

6. **Check invalidation rules**: read `wiki/concepts/invalidation-rules.md` first. If the source makes claims about bar-sim trailing, V10i alignment, BE stops, or any other invalidated pattern, tag it `suspect-results` and add a "Why this is tagged suspect" section.

7. **Add source document wiki-link**: at the bottom of the summary, add:
   ```
   ## Source documents
   - [[raw-sources/path/to/SOURCE_FILE]]
   ```

8. **Update index.md** — add new entries under the right headers.

9. **Update MOCs** — if the new summary belongs in a MOC, add it there.

10. **Append to log.md** — one entry per source ingested.
