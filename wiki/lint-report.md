---
created: 2026-04-14
updated: 2026-04-14
type: report
related: []
tags: [maintenance, lint, audit]
---

# Vault Audit Report — 2026-04-14

Full audit of strategy statuses, structural health, and data integrity.

## Strategy status audit

All six strategies checked against raw source evidence. Two corrections made this session (Lumi and Wick Fade). Current statuses verified correct:

| Strategy | Status | Verified Against | Result |
|---|---|---|---|
| IFVG | SIM ONLY | tempo-ifvg-research, unified-brain-architecture | CORRECT |
| Lumi | LIVE (V15 backtest) | tempo-portfolio-v15 (+19.8 pts/day, Calmar 23.1) | CORRECT (was EXCLUDED, fixed this session) |
| Wick Fade | UNPROVEN | Research Log March 4 (+0.29 R/day marginal), 15S flip dead (0/216) | CORRECT (was VALIDATED, fixed this session) |
| BOS FVG | DEAD | bos-fvg-failure-consolidated (bar +0.359, tick +0.001) | CORRECT |
| SF Portfolio | TICK-CONFIRMED | sf-portfolio-cluster (+3.34 R/day, 260d) | CORRECT |
| Sweep Fade | DEAD/MARGINAL | sweep-cluster, Research Log March 4 | CORRECT |

Pages updated: strategies-moc, tempo-moc, root-moc, wick-fade concept, wickfade-complete summary, lumi-strategy-spec, tempo-portfolio-v15, project index.

## Structural health

| Check | Result |
|---|---|
| Orphan pages | 0 (all wiki pages referenced) |
| Frontmatter completeness | 115/115 wiki files have required fields |
| Date validation | No inverted dates |
| Stale pages (>30d) | 0 |
| Source arrays on summaries | 75/75 populated |
| Index.md coverage | Complete |
| Daily notes | 4 files, all with proper frontmatter and wiki-links |
| Templates | 11 files, all present with content |
| Log.md integrity | Properly formatted, append-only |

## Issues found and fixed

1. **Lumi status** — was EXCLUDED everywhere based on V14 data, ignoring V15 rewrite. Fixed to LIVE (V15) across 6 pages.
2. **Wick Fade status** — was VALIDATED/CANONICAL/TICK-VALIDATED based on uncritical acceptance of WickFade Complete doc claims. Downgraded to UNPROVEN across 6 pages. Tags changed from `canonical, tick-validated` to `unproven`.
3. **Project index** — said "Live portfolio" for IFVG+Lumi, clarified to "Validated portfolio (backtest, not yet live)".
4. **dedup-report.md** — missing `related:` field in frontmatter. Added.
5. **wickfade-complete.md** — opening blockquote still claimed "strongest defensible edge" and "tick-level validated". Rewritten to reflect UNPROVEN status with evidence.

## Remaining cosmetic issues (not fixed)

135 wikilinks across 45 files use shorthand format (`[[ifvg]]` instead of `[[wiki/concepts/ifvg]]`). These resolve correctly in Obsidian via fuzzy basename matching but are technically non-standard. Not fixing because: (a) Obsidian handles them fine, (b) the shorthand is actually more readable, (c) mass-editing 135 links risks introducing new broken links.

## Vault stats

- Total .md files: 300
- Wiki pages: 115 (75 summaries, 19 concepts, 6 entities, 4 syntheses, 10 maps, 1 dedup-report)
- Raw sources: 224
- Connected graph: 1 component (285 content files), 1,170+ wiki-link edges
- Scheduled tasks: 5
- Slash commands: 5
- Agent skills: 5
