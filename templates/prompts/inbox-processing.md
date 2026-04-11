# Inbox Processing Prompt

Paste this into Claude Code as your morning ritual:

---

Run `/inbox`. Process every file in `inbox/` by:

1. Read the item in full.

2. Classify into ONE of:
   - **note** → create `wiki/concepts/<slug>.md` or `wiki/entities/<slug>.md` with full frontmatter
   - **action** → append to the relevant project's **Open actions** section as `- [ ] <task>` with owner/deadline if obvious
   - **calendar** → add to the current or target `daily/YYYY-MM-DD.md` under **Today's focus**
   - **reference** → move the file to `raw-sources/<domain>/` and run `/ingest` on it
   - **delete** → it's junk, remove it

3. Before filing, add `[[double brackets]]` around every significant concept, entity, or project mentioned in the item. Create stub pages in `wiki/concepts/` for any referenced concept that doesn't yet have one.

4. Set proper frontmatter. `created:` = today, `updated:` = today, `type:` from the enumeration, `tags:` lowercase kebab-case.

5. Remove the file from `inbox/` once filed.

6. Report back with a 3-bullet summary:
   - Items processed: N (split by classification)
   - New wiki pages created: <list>
   - Anything you couldn't confidently classify — flag for human review

7. Append one `## [YYYY-MM-DD HH:MM] inbox | N items processed` line to `log.md`.

**Rules:**
- Never leave a file in `inbox/` if you can classify it with > 70% confidence — file it.
- If you're < 70% confident, tag it `inbox-unclassified` and leave it — `/weekly` will catch it.
- No apologies or commentary. Just do the work and report the outcomes.
- If the same classification mistake keeps happening, update CLAUDE.md's `/inbox` operation rules — don't just correct each item individually.
