# MOC Creation Prompt

Paste this into Claude Code when a topic cluster needs a Map of Content:

---

Create or update a Map of Content for `<topic>`. Follow CLAUDE.md `/moc`.

1. Scan `wiki/` for every page that tags, links to, or describes `<topic>`:
   ```
   grep -rln "<topic>" wiki/
   grep -rln "\[\[.*<topic-slug>.*\]\]" wiki/
   ```

2. Curate aggressively. A MOC is an editorial view, not a flat list. Pick the 10-20 most important pages, organized into logical sections:
   - **Orientation** — 2-3 sentence topic overview
   - **Core concepts** — load-bearing idea pages in reading order
   - **Entities** — people, companies, products
   - **Canonical sources** — the authoritative summaries
   - **Syntheses** — cross-source analyses
   - **Open questions** — things the MOC can't answer yet
   - **Related MOCs** — adjacent topics with their own MOCs

3. Annotate each link with a one-line blurb — tell the reader *why* to click.

4. Create `wiki/maps/<topic>-moc.md` from `templates/moc.md`.

5. Add a `[[maps/<topic>-moc]]` back-link to the `related:` frontmatter of every included page.

6. Update `index.md` — add the MOC under the `## Maps` section.

7. Append to `log.md`: `## [YYYY-MM-DD HH:MM] moc | <topic>`.

**Quality check before finishing:**
- Can the reader understand the topic in < 2 minutes from the MOC alone?
- Is the hierarchy opinionated (your editorial choice) or just alphabetical (lazy)?
- Does every link have a one-line blurb?
- Did you include only the 10-20 most important pages, not every page that mentions the topic?

If any of those answers is no, tighten before committing.
