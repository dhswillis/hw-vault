# CLAUDE.md — Vault Operating Manual

Instructions for Claude Code when operating in this Obsidian vault. Read this first every session.

## Purpose

Personal second-brain vault following the Karpathy LLM Wiki pattern:

- `raw-sources/` is **immutable**. You read from it; you never write to it.
- `wiki/` is LLM-maintained markdown. You own it.
- `CLAUDE.md` (this file) is the schema. Update it when conventions change.

## Folder conventions

- `work/projects/` — active work projects, one folder per project (kebab-case)
- `work/meetings/` — meeting notes, one file per meeting
- `work/decisions/` — decision log entries
- `work/people/` — colleagues, clients, stakeholders
- `personal/projects/` — personal side projects
- `personal/goals/` — goals, OKRs, reviews
- `personal/health/` — fitness, sleep, habits
- `personal/finance/` — personal finance notes (never store account numbers)
- `daily/` — daily notes, `YYYY-MM-DD.md`
- `wiki/concepts/` — concept pages (ideas, frameworks, patterns)
- `wiki/entities/` — people, companies, products, places
- `wiki/summaries/` — per-source summaries (one per raw source)
- `wiki/syntheses/` — cross-source analyses and answered queries worth keeping
- `raw-sources/` — **read-only**. Articles, PDFs, transcripts, clipped pages
- `raw-sources/assets/` — images (gitignored, may grow large)
- `templates/` — note templates
- `inbox/` — quick capture, triaged during `/wrap-up`

## Naming

- Daily notes: `YYYY-MM-DD.md`
- Projects: `kebab-case/` folder, with `index.md` as the project root
- Meetings: `YYYY-MM-DD-topic-kebab.md`
- Wiki pages: `kebab-case.md`
- People entities: `firstname-lastname.md`

## Required frontmatter

### Daily (`daily/*.md`)
```yaml
---
date: YYYY-MM-DD
mood:
energy:
tags: [daily]
links: []
---
```

### Project (`{work,personal}/projects/<slug>/index.md`)
```yaml
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active  # active | paused | done | archived
owner:
tags: [project]
links: []
---
```

### Meeting (`work/meetings/*.md`)
```yaml
---
date: YYYY-MM-DD
attendees: []
project:
tags: [meeting]
links: []
---
```

### Wiki page (`wiki/**/*.md`)
```yaml
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept  # concept | entity | summary | synthesis
sources: []    # paths under raw-sources/
related: []    # paths under wiki/
tags: []
---
```

Always set `updated` when you modify a file. Never backdate `created`.

## Linking rules

- Wrap every significant concept, entity, or project in `[[double brackets]]` so Obsidian's graph view lights up.
- On first mention of a concept that has no page yet, create a stub in `wiki/concepts/` and link to it. Stubs are fine — they get filled in later.
- Cross-link liberally. Wiki pages should reference each other, not exist in isolation.
- Use wiki-links (`[[page]]`) for vault-internal refs; reserve markdown links for external URLs.

## Operations

### Ingest
**Trigger:** a new file lands in `raw-sources/`, or user says "ingest &lt;file&gt;" / "ingest everything in raw-sources/".

**Reading non-markdown sources** — convert to text first, don't try to read binaries:
- `.docx` → `textutil -convert txt -output /tmp/<slug>.txt <file>.docx`, then `Read /tmp/<slug>.txt`
- `.pdf` → use `Read` tool's native PDF support (`pages: "1-20"` for large files)
- `.xlsx` → `python3 -c "import pandas as pd; print(pd.read_excel('<file>').to_string())"` or equivalent
- `.html` → prefer `pandoc -f html -t markdown` if installed
- If a format has no clean converter, log it and ask rather than guess

**Steps:**
1. **Read every source in full before writing anything.** For multi-file ingests, cross-source synthesis is often more valuable than per-source summaries alone — you need all the context first.
2. **Write a summary** in `wiki/summaries/<source-slug>.md` with `type: summary`, `sources: [<path>]`. Slug = kebab-case of filename minus extension and trailing dates.
3. **Create concept + entity pages** for anything meeting the **rule of three**: mentioned ≥ 3 times across the ingest batch + existing wiki, OR mentioned once but load-bearing (a critical bug, a core signal, a system entity). Otherwise inline the explanation in the summary. Avoid stub-of-stub pages.
4. **Cross-link liberally** with `[[double brackets]]`. Every summary links to every concept/entity it introduces. Every concept links back to at least one source summary.
5. **Check for cross-source synthesis.** When two or more sources in the same batch contradict, complement, or update each other, write `wiki/syntheses/<slug>.md` analyzing the relationship. This is where the wiki accumulates real value — not optional when the material is there.
6. **Check for auto-memory overlap.** If a source references something already in `~/.claude/projects/-Users-harrisonwillis/memory/`, note the cross-reference in the summary and include the memory file name in `related:`. Keeps the two stores in sync.
7. **Mark suspect data.** If a source's claims are questionable (known bug contamination, missing methodology, unverified numbers), add `suspect-results` tag and include a dedicated "Why this is tagged suspect" section with the specific reason. Never silently filter — always surface.
8. **Update `index.md`** — add new summaries, concepts, entities, syntheses under the right headers. Rewrite the whole file with `Write` (faster than `Read → Edit` for small files).
9. **Remove `.gitkeep`** from any folder that now has real content.
10. **Append to `log.md`** — **one entry per source, plus one entry per synthesis written**:
    ```
    ## [YYYY-MM-DD HH:MM] ingest | <source filename> → <summary path>; concepts <list>; entities <list>
    ## [YYYY-MM-DD HH:MM] synthesis | <slug> — <one-line thesis>
    ```
11. **Never modify the raw source itself.** Read-only. Annotations belong in `wiki/summaries/`.

### Query
**Trigger:** user asks a question.
1. Read `index.md` first.
2. Follow it into relevant wiki pages. Read them fully.
3. Answer with citations as wiki paths (e.g., `wiki/concepts/foo.md`).
4. If the answer is non-trivial and reusable, file it as `wiki/syntheses/<slug>.md` and add to `index.md`.
5. Append to `log.md`: `## [YYYY-MM-DD HH:MM] query | <question summary>`.

### /lint
**Trigger:** user says `/lint`.
1. Scan the wiki for:
   - Contradictions between pages
   - Orphan pages (no inbound `[[links]]`)
   - Concepts mentioned in prose but without their own page
   - Stale claims (pages with `updated` &gt; 90 days, or sources no longer in `raw-sources/`)
2. Write the report to `wiki/lint-report.md` (overwrite previous).
3. Append to `log.md`: `## [YYYY-MM-DD HH:MM] lint | N issues found`.

### /daily
**Trigger:** user says `/daily`, or session starts on a new day without a note.
1. Create `daily/YYYY-MM-DD.md` from `templates/daily.md`.
2. Prefill **Open actions** by scanning `work/projects/` and `personal/projects/` for unchecked action items (`- [ ]`) and carrying them forward with a source link.
3. Append to `log.md`: `## [YYYY-MM-DD HH:MM] daily | created`.

### /wrap-up
**Trigger:** user says `/wrap-up`.
1. Summarize the session: what was read, written, decided.
2. Extract decisions → `work/decisions/YYYY-MM-DD-<topic>.md` or the relevant project file.
3. Extract action items → the right project's **Open actions** section, with owner and deadline if given.
4. Triage `inbox/` — file items or flag them.
5. Append to `log.md`: `## [YYYY-MM-DD HH:MM] wrap-up | <session summary>`.
6. Run `git add . && git commit -m "wrap-up: <session summary>" && git push`. If no remote is configured, skip the push and say so.

## Hard rules

- **NEVER modify files in `raw-sources/`.** Read-only. Annotations go in `wiki/summaries/`.
- **`log.md` is append-only.** Never rewrite history. Format: `## [YYYY-MM-DD HH:MM] <operation> | <description>` so it greps with `grep "^## \[" log.md | tail -20`.
- **Always update `updated:` frontmatter** when modifying a wiki or project page.
- **No emoji in generated markdown** unless the user adds them first.
- **LF line endings, consistent heading levels, no trailing whitespace.**
- **If something is ambiguous, update this file** to disambiguate it next time — don't silently work around it.

## Tags vocabulary

Reserved tags that carry specific meaning across the vault:

- `suspect-results` — source or summary contains claims contaminated by a known bug, missing methodology, or unverified numbers. MUST be accompanied by a "Why this is tagged suspect" section.
- `superseded` — superseded by a newer source or finding; link forward to the replacement with `related:`.
- `open-question` — contains an unresolved question flagged for future research.
- `stub` — page exists only as a cross-link target, content to be filled later.

Free-form tags are fine for domain (e.g. `trading`, `nq`, `ninjatrader`). Reserved tags must be used exactly as above.

## Tool shortcuts

- `Write` is faster than `Read → Edit` for files small enough to rewrite in full. Use it for `index.md`, `log.md` updates, and any single-page overwrite.
- `Edit` is correct when (a) the file is large and only a small region changes, or (b) you need `replace_all` semantics. Edit requires a prior `Read`.
- For non-markdown ingests, convert to text in `/tmp/` and read the converted file — don't try to read `.docx`, `.xlsx`, or compressed formats directly.
