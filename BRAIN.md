---
created: 2026-04-11
updated: 2026-04-11
type: architecture
tags: [architecture, operating-manual, root]
---

# BRAIN.md — Vault Operating Architecture

> The single spec for how this vault functions as a second brain.
> `CLAUDE.md` tells Claude Code *what to do*. `BRAIN.md` tells the whole system *how it fits together*.

## Entry points

Every path into the brain goes through one of three files. Nothing else is a starting point.

1. **[[wiki/maps/root|Root Map of Content]]** — the top-level map. Start here to browse.
2. **[[index|index.md]]** — the flat catalog. Start here when you know what you want by name.
3. **[[log|log.md]]** — the append-only operations journal. Start here when you want to know what changed and when.

`BRAIN.md` itself is the spec; it should rarely be read end-to-end — the root MOC is the daily entry point.

## The six layers

The brain is a six-layer knowledge pyramid. Each layer reads from the one below and writes upward.

| Layer | Folder | Mutable? | Purpose |
|---|---|---|---|
| **L0 — Raw sources** | `raw-sources/` | read-only | Original source material: articles, transcripts, PDFs, docx, exports. Nothing is rewritten here. |
| **L1 — Summaries** | `wiki/summaries/` | yes | One markdown digest per raw source. Captures the point of the source in Claude's own words, cross-linked to concepts + entities. |
| **L2 — Atomics** | `wiki/concepts/`, `wiki/entities/` | yes | Reusable atomic units. A concept is an idea or pattern; an entity is a named thing (person, product, system, org). These are the nodes that get reused across many summaries. |
| **L3 — Syntheses** | `wiki/syntheses/` | yes | Cross-source analyses: "how do sources A, B, C relate"; "why does claim X contradict claim Y"; "what does the arc of these five papers actually tell us?" This is where the vault accumulates insight the sources don't contain individually. |
| **L4 — Maps of Content** | `wiki/maps/` | yes | Navigational hubs. A MOC is a curated index page for a domain. Root MOC → topic MOCs → summaries/syntheses. |
| **L5 — Operations** | `daily/`, `weekly/`, `work/projects/`, `personal/projects/`, `inbox/` | yes | Time-bound work product: daily notes, weekly rollups, project pages, and the inbox queue. |

**Direction of flow:** raw → summary → atomics → synthesis → MOC → operations.
**Direction of linking:** every L(n+1) node links *down* to the L(n) nodes it derives from. L5 can link to anything.

## Four core pipelines

These are the operations that move knowledge between layers. All are defined in detail in `CLAUDE.md`.

### 1. Ingest (L0 → L1 → L2 → L3)

**Trigger:** new file in `raw-sources/`, or user command `"ingest <path>"`, or scheduled batch.

Read every source in full → write one summary per source → create concept/entity pages on the **rule of three** → write a synthesis when two or more sources in the batch contradict / complement each other → update the relevant MOC → append to `log.md`.

### 2. Query (L0–L4 → answer)

**Trigger:** user asks a question.

Read root MOC → follow into relevant wiki pages → answer with citations as wiki paths. If the answer is non-trivial and reusable, file it as a synthesis.

### 3. Daily (L5)

**Trigger:** new day, or `/daily` command, or scheduled at 7:00 local.

Create `daily/YYYY-MM-DD.md` from `templates/daily.md` → prefill **Open actions** by scanning projects for unchecked `- [ ]` items → append to `log.md`.

### 4. Lint (L0–L5 → diagnostic)

**Trigger:** `/lint` command, or scheduled weekly.

Scan the vault for: orphans (no inbound links), contradictions between pages, stub concepts with no body, stale claims (>90 days), raw sources without summaries, summaries whose sources no longer exist. Write report to `wiki/lint-report.md`.

## Automation

The following scheduled tasks run without user intervention.

| Task | Schedule | What it does |
|---|---|---|
| **Daily note** | Mon–Fri 7:00 local | Creates today's daily note from template, prefills open actions. |
| **Weekly rollup** | Sun 17:00 local | Summarizes the week into `weekly/YYYY-Www.md`; aggregates decisions, open actions, new summaries/concepts added. |
| **Lint sweep** | Sat 20:00 local | Runs `/lint` and writes `wiki/lint-report.md`. If there are orphans or suspect-results tags added since last run, flags them at the top of the report. |
| **Inbox triage** | Tue + Fri 18:00 local | Scans `inbox/` for anything older than 3 days; routes or escalates. |

Tasks are managed via the Scheduled Tasks system. See `wiki/maps/automation.md` for the current live tasks, their IDs, and how to edit them.

## Hard rules

1. **`raw-sources/` is immutable.** Read-only. Annotations go in `wiki/summaries/`.
2. **`log.md` is append-only.** Never rewrite history. Format: `## [YYYY-MM-DD HH:MM] <operation> | <description>`.
3. **No orphans.** Every file in the vault must be reachable from the root MOC within 3 hops. Lint enforces this.
4. **Every wiki page has frontmatter.** `created`, `updated`, `type`, `tags`, plus type-specific fields. Set `updated` whenever you modify.
5. **Link liberally with `[[double brackets]]`.** The graph is the brain's connectivity map; if nothing links to a page, it doesn't exist.
6. **On the rule of three:** a concept or entity earns its own page only when it's referenced three times across the wiki, OR when it's load-bearing for one critical source.
7. **Suspect data gets tagged, never silently filtered.** Use `suspect-results` + a "Why this is tagged suspect" section.
8. **Privacy boundaries.** `personal/legal/` and `raw-sources/claude-data/` are gitignored. Never commit them.

## Reading order for a new Claude session

When Claude Code (or any agent) opens this vault for the first time in a session, it should read these files in order:

1. `BRAIN.md` — this file. Architecture.
2. `CLAUDE.md` — operating manual. Naming, frontmatter, operations.
3. `wiki/maps/root.md` — current state of the brain.
4. `log.md` (tail) — what's happened recently.
5. `wiki/lint-report.md` if it exists — known issues.

After that, follow the root MOC into whatever domain the task is in.
