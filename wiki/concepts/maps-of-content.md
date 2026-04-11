---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources: []
related:
  - wiki/concepts/inbox-processing.md
tags: [vault, methodology, navigation]
---

# Maps of Content (MOC)

A **Map of Content** is a curated index note that pulls together all the wiki pages on a single topic. MOCs are how a second-brain vault stays navigable once the flat `index.md` gets too big to scan.

## MOC vs index.md vs tag

| Tool | Strength | When to use |
|---|---|---|
| **`index.md`** | Flat catalog, fast lookup, one line per page | Always — it's the master directory |
| **Tags** | Automatic, cross-cutting, filter-based | When you want every page mentioning X regardless of topic grouping |
| **MOC** | Curated editorial hierarchy, opinionated annotations, 2-3 sentence orientation | When a topic cluster has 10+ pages and a newcomer would need guidance to navigate them |

**Think of it this way:** `index.md` is the card catalog, tags are the search filter, MOCs are the librarian's curated "start here if you want to learn about X" reading list.

## When to create a MOC

- When you notice yourself typing the same 5 cross-links across multiple wiki pages
- When a topic cluster has 10+ pages
- When a newcomer would need orientation before they could navigate the cluster productively
- When two related topics are starting to conflate and you need editorial hierarchy to separate them

## When NOT to create a MOC

- Fewer than 5 pages on the topic — the flat index does fine
- The topic is already well-covered by a synthesis page or an entity page — don't duplicate
- You're tempted to make a MOC of MOCs (meta-indexing) — stop, that's what `index.md` is for

## MOC structure

See `templates/moc.md` for the canonical template. Key sections:

1. **Orientation** — 2-3 sentences: what this MOC covers, how to think about the topic, what the reader will find
2. **Core concepts** — the load-bearing idea pages, in logical reading order
3. **Entities** — the people, companies, products, systems that matter
4. **Canonical sources** — the one or two summaries that are "the" authoritative source
5. **Syntheses** — cross-source analyses that connect things
6. **Open questions** — things the MOC can't yet answer cleanly (signals where research is needed)
7. **Related MOCs** — adjacent topics with their own MOCs

## Back-links (critical)

Every page included in a MOC should carry a `[[maps/<topic>-moc]]` back-link in its `related:` frontmatter. That way:

- Obsidian's graph view shows the MOC as a hub node
- Dataview queries can filter "which MOCs does this page belong to"
- A reader dropping into a leaf page can navigate up to the MOC and from there to sibling pages

## The `/moc` operation

`CLAUDE.md` defines `/moc <topic>` as the creation/update ritual. See CLAUDE.md § Operations § /moc for the full steps.

## MOC quality signals

A good MOC:
- Has a curated hierarchy, not an alphabetical list
- Annotates each link with a one-line blurb (tells the reader *why* to click)
- Is short enough to read in < 2 minutes
- Is opinionated — the author's judgment of what matters most

A bad MOC:
- Is just a flat list of `[[page]]` links with no annotation
- Duplicates the flat `index.md` for a subset
- Never updates — stale MOCs are worse than no MOCs
- Tries to cover everything (it's a map, not the territory)

## Related

- Obsidian community background: see [[summaries/obsidian-best-practices-research]] (forthcoming)
- The Karpathy LLM Wiki pattern treats `index.md` as the primary navigation; MOCs are a layer added by the Obsidian community on top of that base
