# CLAUDE.md — Vault Operating Manual

Instructions for Claude Code when operating in this Obsidian vault. Read this first every session.

## Purpose

Personal second-brain vault following the Karpathy LLM Wiki pattern:

- `raw-sources/` is **immutable**. You read from it; you never write to it.
- `wiki/` is LLM-maintained markdown. You own it.
- `CLAUDE.md` (this file) is the schema. Update it when conventions change.

## Folder conventions

- `work/projects/` — active work projects, one folder per project (kebab-case). Primary context: **Haven Park Communities** (Harrison is Director of Asset Management). See `wiki/entities/harrison-haven-park.md` for the work context roster.
- `work/meetings/` — meeting notes, one file per meeting
- `work/decisions/` — decision log entries
- `work/people/` — colleagues, clients, stakeholders (one file each)
- `personal/projects/` — personal side projects (including trading: `personal/projects/tempo-trading/`)
- `personal/goals/` — goals, OKRs, reviews
- `personal/health/` — fitness, sleep, habits
- `personal/finance/` — personal finance notes (never store account numbers)
- `personal/legal/` — legal + estate matters (gitignored for privacy; sensitive docs)
- `daily/` — daily notes, `YYYY-MM-DD.md`
- `weekly/` — weekly reviews, `YYYY-Www.md`
- `wiki/concepts/` — concept pages (ideas, frameworks, patterns)
- `wiki/entities/` — people, companies, products, places
- `wiki/summaries/` — per-source summaries (one per raw source)
- `wiki/syntheses/` — cross-source analyses and answered queries worth keeping
- `wiki/maps/` — Maps of Content (MOCs) — curated index notes for a topic cluster, see [[concepts/maps-of-content]]
- `raw-sources/` — **read-only**. Articles, PDFs, transcripts, clipped pages. Subfolders by domain (`trading/`, `work/`, `personal/`, `claude-data/`).
- `raw-sources/assets/` — images (gitignored, may grow large)
- `templates/` — note templates (daily, weekly, project, meeting, moc, decision, person)
- `templates/prompts/` — reusable prompt snippets for common workflows
- `inbox/` — quick capture, triaged during `/inbox` or `/wrap-up`
- `.claude/skills/` — kepano's official Obsidian Agent Skills (obsidian-markdown, obsidian-cli, obsidian-bases, json-canvas, defuddle)
- `.claude/commands/` — custom slash commands (trade-review, person, meeting, dedup, ingest)

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

### /inbox
**Trigger:** user says `/inbox` or "process my inbox".

The morning ritual for converting raw capture into structured notes. Inspired by the standard Claude-Code-in-Obsidian pattern (see [[concepts/inbox-processing]]).

1. Read each file in `inbox/`.
2. For each item, classify: **note** (belongs in wiki), **action** (goes into a project's Open actions), **calendar** (scheduled item — add to today's or a future daily), **reference** (belongs in `raw-sources/` then run `/ingest`), or **delete** (junk).
3. Create the appropriate structured note or append to the appropriate place. Always set full frontmatter.
4. Identify cross-links to existing wiki pages. Add `[[double brackets]]` wherever the new note mentions a concept or entity that already has a page.
5. Delete or move the inbox item once filed.
6. Append to `log.md`: `## [YYYY-MM-DD HH:MM] inbox | N items processed`.

Rule: **never leave items in `inbox/` longer than a week**. The next `/weekly` review will flag orphans.

### /weekly
**Trigger:** user says `/weekly` on Friday, or on Monday for the prior week.

Weekly review — one of the most valuable habits in any knowledge system. Structured so Claude can do 80% of it automatically from the week's daily notes and project files.

1. Create `weekly/YYYY-Www.md` from `templates/weekly.md` where `Www` is ISO week number.
2. **Summarize completed work** from the week's daily notes (`daily/YYYY-MM-DD.md` × 5-7). Pull wins, blockers, notes.
3. **Aggregate open actions** from all projects with a `- [ ]` still unchecked. Group by project.
4. **Surface stale items** — anything in `inbox/` > 7 days, any `daily/*.md` with unchecked actions carried forward more than twice.
5. **Identify decisions** made this week — cross-reference `work/decisions/`.
6. **Flag pending queries** — anything in `wiki/syntheses/` with unresolved open questions.
7. **Next-week focus** — prompt the user for their top 3 priorities; fill in the template.
8. Append to `log.md`: `## [YYYY-MM-DD HH:MM] weekly | Www review`.

### /moc
**Trigger:** user says `/moc <topic>` or requests a Map of Content for a subject.

Create or update a Map of Content — a curated index note that pulls together all wiki pages on a topic. MOCs are how large clusters become navigable; they complement (but don't replace) `index.md`.

1. Scan `wiki/` for all pages that tag, link to, or describe the topic.
2. Create or update `wiki/maps/<topic>-moc.md` with:
   - A 2-3 sentence orientation for the topic
   - A curated hierarchy of the most important pages (not every page — the MOC is an editor's-cut view)
   - Cross-links to any related MOCs
3. Add a `[[maps/<topic>-moc]]` back-link to each wiki page included.
4. Update `index.md` — add the MOC under a new `## Maps` section if not already present.
5. Append to `log.md`: `## [YYYY-MM-DD HH:MM] moc | <topic>`.

See `templates/moc.md` for the MOC structure.

### /capture
**Trigger:** user says `/capture <thought>` or `/capture` followed by content.

Quick-capture without classification. Drops a timestamped file in `inbox/` for later processing via `/inbox`.

1. Create `inbox/YYYY-MM-DD-HHMM-<slug>.md` with the user's content verbatim and minimal frontmatter (`date`, `tags: [inbox]`).
2. Do NOT attempt to classify or cross-link yet — that's what `/inbox` is for.
3. Append to `log.md`: `## [YYYY-MM-DD HH:MM] capture | <slug>`.

### /trade-review
**Trigger:** user says `/trade-review` or "log today's trades".

End-of-day trading review. Full spec in `.claude/commands/trade-review.md`. Creates a trade log entry, links to strategy wiki pages, checks against invalidation rules, updates daily note.

### /person
**Trigger:** user says `/person <name>` or mentions creating/updating a person file.

Create or update a person file in `work/people/`. Full spec in `.claude/commands/person.md`. Extracts structured data, cross-links to projects and meetings.

### /meeting
**Trigger:** user says `/meeting <topic>` or "log a meeting".

Capture a meeting — create note, extract decisions and actions, update person files, link to projects. Full spec in `.claude/commands/meeting.md`.

### /dedup
**Trigger:** user says `/dedup` or "check for duplicates".

Scan wiki for duplicate or near-duplicate pages. Full spec in `.claude/commands/dedup.md`. Writes report to `wiki/dedup-report.md`. Does NOT auto-merge.

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

## Agent skills (.claude/)

**kepano's Official Obsidian Skills** are installed in `.claude/skills/`. These teach Claude Code how to use Obsidian-native features:

- `obsidian-markdown` — Obsidian Flavored Markdown (wikilinks, embeds, callouts, properties)
- `obsidian-cli` — Obsidian CLI for vault search, note management, task management, plugin dev
- `obsidian-bases` — Obsidian Bases (.base files) for database-like views
- `json-canvas` — JSON Canvas (.canvas) for visual mind maps and flowcharts
- `defuddle` — Clean web page extraction (use instead of WebFetch for articles)

**Custom slash commands** are in `.claude/commands/`:
- `/trade-review` — end-of-day trading log with strategy cross-links
- `/person` — create/update person files from mentions
- `/meeting` — full meeting capture pipeline (notes, decisions, actions, person files)
- `/dedup` — scan wiki for duplicate/overlapping pages
- `/ingest` — wrapper for the CLAUDE.md ingest pipeline with all rules

## Git safety

A **pre-commit hook** (`.git/hooks/pre-commit`) validates:
- YAML frontmatter syntax on all staged .md files
- No tab indentation in frontmatter
- No unclosed quotes
- Warns if `log.md` has deleted lines (append-only rule)

To bypass in emergencies: `git commit --no-verify`

## Obsidian best practices integrated

Schema is informed by what the Obsidian + Claude Code community has converged on in 2026. Not every plugin is required — the vault is fully usable with Obsidian core — but the conventions are friendly to these additions if/when they're installed.

### Recommended plugins (install if you want richer automation)

- **Templater** — variable-driven templates (`{{date}}`, `{{title}}`, JavaScript snippets). The existing `templates/*.md` use plain tokens that work with either raw Templater or manual substitution.
- **Dataview** — query notes as a database. Enables auto-updating tables in `index.md`, weekly reviews, and MOCs. Frontmatter is already shaped for Dataview queries (consistent `type`, `status`, `tags`, `date` fields).
- **Periodic Notes** — adds weekly/monthly/quarterly note support on top of daily notes. Matches the `weekly/` folder and `/weekly` operation.
- **Tasks** — tracks `- [ ]` checkboxes across the vault, aggregates them, supports due-date queries. Enables `/weekly` to pull open actions automatically.
- **Calendar** — month-view sidebar for navigating daily notes.
- **QuickAdd** — rapid capture shortcuts. Matches the `/capture` operation.

**Install order** (community guidance): start with Templater + Dataview + Calendar. Add the rest only when you feel the specific pain they solve. Do not install 50 plugins on day one.

### MOCs vs index.md

`index.md` is a **flat catalog** — one line per wiki page, grouped by type. Fast lookup.

A **Map of Content** (`wiki/maps/<topic>-moc.md`) is a **curated editorial view** of a topic — the pages that matter most, in a meaningful hierarchy, with opinionated annotations. Use MOCs when a topic cluster has 10+ pages that a reader wouldn't be able to navigate via flat index alone.

Rule of thumb: create a MOC when you notice yourself typing the same 5 cross-links in multiple wiki pages. Those links want to be one MOC.

### Frontmatter as a database

Every frontmatter field should support future Dataview queries. Keep the schema consistent:

- `type` — always one of the enumerated values per file type
- `tags` — lowercase kebab-case, no spaces, no `#` prefix in frontmatter
- `created`, `updated`, `date` — ISO format `YYYY-MM-DD`
- `status` — `active` / `paused` / `done` / `archived`
- `related` — array of vault paths for cross-linking
- `sources` — array of `raw-sources/` paths for provenance

Dataview queries you can run once those plugins are installed (example):

```dataview
TABLE status, updated FROM "work/projects" WHERE status = "active" SORT updated DESC
```

### Daily note philosophy

Daily notes are a **timeline**, not a planner. Don't over-structure them. The template has six sections (focus / open actions / notes / wins / blockers / tomorrow) but you should feel free to omit any. The `/weekly` operation aggregates across daily notes, so the value of any single daily note is low — the value is in the week of them together.

### Linking every mention

The single most load-bearing habit: **wrap every significant concept, entity, or project in `[[double brackets]]` the first time it appears in any note you write**. Obsidian's graph view rewards this exponentially — one extra link on each of 100 notes creates a navigable graph, while zero links creates a filing cabinet.

## Reserved status & workflow tags (extended)

In addition to `suspect-results`, `superseded`, `open-question`, `stub`:

- `dead-strategy` — signal/strategy tested and confirmed to have no edge. Must include a "Why it died" section and forward-link to the audit.
- `canonical` — the authoritative source for a concept/entity. Other pages should link *to* the canonical, not duplicate it.
- `draft` — work in progress, not ready to cite.
- `in-flight` — currently being edited this session; may change.
- `needs-reingest` — source has been updated since the summary was written; re-ingest to refresh.

## Authority hierarchy

When two sources disagree, trust them in this order (highest first):

1. **User's direct edits to vault files** made during the current session (highest — see `feedback_user_corrections_authoritative` memory)
2. **Project CLAUDE.md** files (e.g. `~/Documents/trading-system/CLAUDE.md`) — ground truth for the project's current position
3. **Most recent tick-level audit** that paired against ground-truth data
4. **Most recent clean-sim audit** (walk-forward, multi-comparison corrected)
5. **Mining reports / research docs** — historical, often contaminated, cite with caveats
6. **User's auto-memory** (`~/.claude/projects/-Users-harrisonwillis/memory/MEMORY.md`) — useful context but point-in-time
7. **General web knowledge** — lowest; only cite when nothing in the vault has the answer
