---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources: []
related:
  - wiki/concepts/maps-of-content.md
tags: [vault, methodology, workflow]
---

# Inbox Processing

A **morning ritual** for converting raw captured items into structured wiki notes. The pattern comes from the Obsidian + Claude Code community's standard workflow (2026) and is implemented in this vault as the `/inbox` operation.

## The problem it solves

Without a capture-and-process split, every new idea or link either:
- Interrupts whatever you're doing (you stop to file it properly) — breaks flow
- Gets lost on a sticky note / scrap of text / bookmark graveyard — never recovered
- Piles up in your head until you forget which ones mattered

The fix: **capture anywhere instantly**, then **process on a schedule**. The inbox is the buffer between the two.

## Capture (continuous, zero-classification)

Use `/capture <thought>` (see CLAUDE.md) or drop a file into `inbox/` directly. Rules for capture:

- **Fast**: zero friction. If it takes more than 10 seconds, you won't use the system.
- **No classification**: don't decide yet where it belongs.
- **Any format**: text, links, half-thoughts, screenshots, voice memo transcripts.
- **Time-stamped**: `inbox/YYYY-MM-DD-HHMM-<slug>.md` so chronology is preserved.

## Process (scheduled, Claude-assisted)

Run `/inbox` every morning (or whatever cadence works). Claude will:

1. **Read each file in `inbox/`** in order
2. **Classify** each into one of:
   - **Note** — belongs in `wiki/` (concept, entity, or synthesis stub)
   - **Action** — belongs in a project's **Open actions** section
   - **Calendar** — scheduled item, add to today's or a future daily note
   - **Reference** — web clip, article, PDF: move to `raw-sources/` and run `/ingest`
   - **Delete** — junk, duplicate, obsolete
3. **File** the item in the right place with **full frontmatter** and cross-links
4. **Link** to any existing wiki pages it touches
5. **Remove** the inbox item once filed

## The key discipline

**Never leave items in `inbox/` longer than a week.** The `/weekly` review flags orphans — if something has been sitting for 7+ days without being processed, it either:
- Doesn't matter (delete it)
- Is blocked on something else (note the blocker explicitly and link it)
- Is actually a project that needs to be created (create it)

The inbox is a **buffer, not a storage**. It must drain.

## Why Claude is suited for this

Processing inbox items requires:
- Reading each one carefully
- Classifying it based on content
- Matching it against existing vault structure
- Creating cross-links
- Writing proper frontmatter

All five of those are things Claude can do faster and more consistently than a human working manually on a Friday afternoon. The human's job is just to **glance at the classification and override it if wrong**.

## Common pitfalls

- **Over-processing**: writing a full summary when a one-line reference would do. The inbox item's value is preserving the thought; polish comes later via `/ingest` or `/query`.
- **Under-linking**: filing a note without adding `[[double brackets]]` for the concepts it mentions. The whole point of the vault is the link graph — an isolated note is barely better than a text file.
- **Classifier drift**: if the same type of item is getting misclassified repeatedly, update CLAUDE.md — don't just correct the output each time.
- **"I'll sort it later"**: the later never comes. Set a weekly floor on inbox age.

## The `/inbox` operation

See CLAUDE.md § Operations § /inbox for the full ritual and the classification schema.

## Relation to `/weekly`

The weekly review is the escalation path for anything `/inbox` couldn't cleanly file. If `/inbox` marks an item as "ambiguous" or "needs more context", `/weekly` will surface it for human review. Between the two, nothing should sit in the vault unclassified for more than 7 days.
