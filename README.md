# HW Vault

Personal Obsidian vault and second brain. Follows the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): raw sources are immutable, the wiki is LLM-maintained markdown, and `CLAUDE.md` tells Claude how to operate.

## Day-to-day use

Open this folder in Claude Code (`cd ~/HW && claude`). Claude reads `CLAUDE.md` automatically. Then use any of:

- **Ingest** — drop a file into `raw-sources/` and say `ingest that`. Claude reads it, writes a summary, updates the wiki and `index.md`, logs it.
- **Query** — ask a question. Claude consults `index.md` first, drills into relevant wiki pages, answers with citations, and files reusable answers to `wiki/syntheses/`.
- **/lint** — health-check the wiki (contradictions, orphans, stale claims). Report lands in `wiki/lint-report.md`.
- **/daily** — create today's daily note from `templates/daily.md`, prefilled with carried-forward open actions.
- **/wrap-up** — end-of-session: extract decisions and action items, triage `inbox/`, append to `log.md`, commit and push.

## Where to clip web articles

Point the [Obsidian Web Clipper](https://obsidian.md/clipper) at this vault and set the default folder to `raw-sources/`. Images get stored in `raw-sources/assets/`. Claude will pick them up on the next `ingest`.

## Sync

Sync is a private git repo.

```bash
# first machine (after reviewing the scaffold)
git remote add origin <your-private-repo-url>
git push -u origin main

# second machine
git clone <url> ~/HW
# then point Obsidian at ~/HW
```

`.obsidian/` is mostly tracked so settings move between machines. Per-device workspace state and plugin caches are gitignored.

## The assets warning

`raw-sources/assets/` is gitignored because clipped images pile up fast and will bloat the repo. If you need assets synced, consider git-lfs or a separate object store — don't just remove the ignore rule.

## The operating manual

Read `CLAUDE.md`. That's the schema Claude follows. When Claude does something you don't like, fix `CLAUDE.md` so it doesn't happen again.
