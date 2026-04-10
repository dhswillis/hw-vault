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
**Trigger:** a new file lands in `raw-sources/`, or user says "ingest &lt;file&gt;".
1. Read the source in full.
2. Write `wiki/summaries/<source-slug>.md` with `type: summary` and `sources: [<path>]`.
3. Update or create relevant pages in `wiki/concepts/` and `wiki/entities/`. Cross-link.
4. Update `index.md` — add the new summary, any new concepts/entities, under the right headers.
5. Append to `log.md`: `## [YYYY-MM-DD HH:MM] ingest | <source title>`.
6. Never modify the raw source.

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
