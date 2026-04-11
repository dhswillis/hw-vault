# Weekly Review Prompt

Paste this into Claude Code on Friday (or Monday morning for the prior week):

---

Run `/weekly` for the week ending {{date}}. Specifically:

1. Read every `daily/YYYY-MM-DD.md` from Monday to Friday of the review week. Pull:
   - Wins
   - Blockers
   - Unchecked action items (`- [ ]`)
   - Any decisions logged
   - Meetings attended

2. Scan `work/projects/*/index.md` and `personal/projects/*/index.md` for any project with `updated:` inside the week — summarize the change.

3. Cross-reference `work/decisions/` for new decision files dated in the review week.

4. Surface stale items:
   - Anything in `inbox/` > 7 days old
   - Action items that have been carried forward more than twice from earlier daily notes
   - Wiki pages with `updated` > 90 days and `status: active`

5. Create `weekly/YYYY-Www.md` using `templates/weekly.md`. Fill in:
   - **Summary** — 2-3 sentences on the week's trajectory
   - **Completed** — checklist of wins
   - **Open actions carried forward** — grouped by project
   - **Stale items flagged** — with suggested next steps
   - **Next week — top 3** — leave blank for me to fill in

6. Append to `log.md` with `## [YYYY-MM-DD HH:MM] weekly | Www review | N open actions, M stale`.

7. Report back with just the one-paragraph summary + the top 5 open actions + the stale-item list. I'll decide what to prioritize from there.

Don't ask clarifying questions — just do it and show me the result.
