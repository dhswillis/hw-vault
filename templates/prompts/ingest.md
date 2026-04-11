# Ingest Prompt

Paste this into Claude Code when you drop something into `raw-sources/`:

---

Ingest `raw-sources/<path>`. Follow the CLAUDE.md `/ingest` operation exactly:

1. Read the source in full. If it's `.docx`, convert with `textutil -convert txt -output /tmp/<slug>.txt <path>` first. If `.pdf`, use the Read tool's native PDF support with `pages: "1-20"` for large files. If `.xlsx`, use pandas.

2. Write `wiki/summaries/<slug>.md` with full frontmatter (`type: summary`, `sources: [<path>]`, `created`, `updated`, `tags`, `related`).

3. Apply the **rule of three** for concept/entity pages — create a new page only for anything referenced ≥ 3 times across the ingest batch + existing wiki, OR mentioned once but load-bearing (a critical bug, a core signal, a system entity). Otherwise inline the explanation in the summary.

4. Cross-link liberally with `[[double brackets]]`. Every summary should link to every concept and entity it introduces.

5. If this source disagrees with an existing wiki page or adds material to an existing synthesis opportunity, write a synthesis in `wiki/syntheses/<slug>.md`.

6. Check for auto-memory overlap. If this source references something in `~/.claude/projects/-Users-harrisonwillis/memory/`, note the cross-reference in the summary's `related:` field.

7. Mark suspect data if the source's claims are questionable (known bug contamination, missing methodology, unverified numbers). Add `suspect-results` tag + a "Why this is tagged suspect" section with the specific reason.

8. Update `index.md` — add new summaries, concepts, entities, syntheses under the right headers. Rewrite the whole file with `Write` rather than `Edit`.

9. Remove `.gitkeep` from any folder that now has real content.

10. Append to `log.md` — one entry per source, plus one entry per synthesis:
    ```
    ## [YYYY-MM-DD HH:MM] ingest | <source filename> → <summary path>; concepts <list>; entities <list>
    ## [YYYY-MM-DD HH:MM] synthesis | <slug> — <one-line thesis>
    ```

11. **Never modify the raw source itself.** Read-only.

12. If anything is ambiguous, **update CLAUDE.md** to disambiguate next time — don't silently work around it.

Report back with the list of files written and any synthesis opportunities you noticed.
