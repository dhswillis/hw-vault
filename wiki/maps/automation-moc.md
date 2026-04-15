---
created: 2026-04-12
updated: 2026-04-12
type: moc
tags: [moc, automation, pipelines, operations]
related:
  - wiki/maps/root.md
  - BRAIN.md
---

# Automation — Map of Content

> How the brain maintains itself. Scheduled tasks, pipelines, and self-healing routines.
> Architecture defined in [[BRAIN|BRAIN.md]]. This page tracks the live implementation.

## Four core pipelines

These are the operations that move knowledge between layers (L0→L5 per [[BRAIN|BRAIN.md]]).

### 1. Ingest Pipeline (L0 → L1 → L2 → L3)

**What:** New raw source → summary → concepts/entities (rule of three) → synthesis if contradictions found → update MOCs → log entry.

**Trigger:** New file in `raw-sources/`, user command, or scheduled batch.

**Current backlog:** ~200 raw sources awaiting ingest (166 from the 2026-04-11 import batch + ~34 original sources without summaries). See [[wiki/maps/import-triage-moc|Import Triage]] for the full manifest.

**Who runs it:** Claude Code, per `CLAUDE.md` instructions.

### 2. Query Pipeline (L0–L4 → answer)

**What:** User asks a question → read root MOC → follow into wiki → answer with citations.

**Trigger:** User question in any session.

**Who runs it:** Claude Code or Cowork, depending on where the question is asked.

### 3. Daily Pipeline (L5)

**What:** Create `daily/YYYY-MM-DD.md` → prefill open actions from projects → log entry.

**Trigger:** Scheduled Mon–Fri at 7:00 AM local, or `/daily` command.

**Template:** `templates/daily.md`

**Scheduled task ID:** `daily-note`

### 4. Lint Pipeline (L0–L5 → diagnostic)

**What:** Scan for orphans, contradictions, stubs, stale claims (>90d), raw sources without summaries, broken links. Write report to `wiki/lint-report.md`.

**Trigger:** Scheduled Saturday at 8:00 PM local, or `/lint` command.

**Scheduled task ID:** `weekly-lint`

## Scheduled tasks

| Task ID | Schedule | Description | Status |
|---|---|---|---|
| `daily-note` | Mon–Fri 7:00 AM | Create daily note from template | Active |
| `weekly-rollup` | Sun 5:00 PM | Summarize week into `weekly/YYYY-Www.md` | Active |
| `weekly-lint` | Sat 8:00 PM | Run lint + dedup sweep, write report | Active |
| `inbox-triage` | Tue + Fri 6:00 PM | Scan inbox for items >3 days old | Active |
| `dedup-scan` | Wed 12:00 PM | Mid-week duplicate detection scan | Active |

## Slash commands (.claude/commands/)

Custom Claude Code commands that automate multi-step vault workflows:

- `/trade-review` — End-of-day trading log: captures trades, links to strategy wiki pages, checks invalidation rules, updates daily note
- `/person` — Create/update person files from mentions: extracts structured data, cross-links to projects and meetings
- `/meeting` — Full meeting capture: notes, decisions, actions, person file updates, project back-links
- `/dedup` — Duplicate detection scan: finds overlapping summaries, near-duplicate concepts, superseded pages
- `/ingest` — Ingest pipeline wrapper: summary, concept/entity creation (rule of three), synthesis, cross-linking

## Agent skills (.claude/skills/)

[[CLAUDE|kepano's official Obsidian Agent Skills]] installed from [github.com/kepano/obsidian-skills](https://github.com/kepano/obsidian-skills):

- `obsidian-markdown` — Obsidian Flavored Markdown syntax
- `obsidian-cli` — Obsidian CLI for vault operations and plugin dev
- `obsidian-bases` — Database-like views (.base files)
- `json-canvas` — Visual mind maps and flowcharts (.canvas files)
- `defuddle` — Clean web page extraction for ingests

## Git safety

A pre-commit hook validates YAML frontmatter on all staged .md files, catches tab indentation, unclosed quotes, and warns on log.md deletions (append-only rule).

## Concepts

- [[wiki/concepts/inbox-processing|Inbox Processing]] — How items flow from `inbox/` into the brain
- [[wiki/concepts/maps-of-content|Maps of Content]] — The L4 navigational layer

## Templates

These templates drive the automation pipelines. Each one is used by its corresponding scheduled task or `/command`:

- [[templates/daily|Daily note template]] — Used by `daily-note` task and `/daily` command
- [[templates/weekly|Weekly review template]] — Used by `weekly-rollup` task and `/weekly` command
- [[templates/meeting|Meeting note template]] — Used when logging meetings via `/meeting`
- [[templates/decision|Decision log template]] — Used when recording decisions
- [[templates/project|Project template]] — Used when creating new projects
- [[templates/person|Person template]] — Used when adding people to `work/people/`
- [[templates/moc|MOC template]] — Used by `/moc` command to create new Maps of Content

### Prompt templates

Reusable prompt snippets for common vault workflows:

- [[templates/prompts/ingest|Ingest prompt]] — Paste into Claude Code to run the full ingest pipeline
- [[templates/prompts/inbox-processing|Inbox processing prompt]] — Paste to run `/inbox` manually
- [[templates/prompts/moc|MOC creation prompt]] — Paste to run `/moc` manually
- [[templates/prompts/weekly-review|Weekly review prompt]] — Paste to run `/weekly` manually

## Related

- [[wiki/maps/import-triage-moc|Import Triage]] — The current big ingest job
- [[wiki/syntheses/research-arc-map|Research Arc Map]] — Example of a synthesis produced by the ingest pipeline
- [[wiki/summaries/unified-brain-architecture|Unified Brain Architecture]] — The summary that inspired this system

---

*Parent: [[wiki/maps/root|Root MOC]]*
