---
description: End-of-day trading review — logs trades, links to strategy pages, updates project state
---

# /trade-review

Run an end-of-day trading review for the current session.

## Steps

1. **Ask** the user what trades were taken today (or if none, what setups were watched). Capture:
   - Instrument (default: NQ)
   - Strategy used (IFVG, Wick Fade, SF Portfolio, etc.)
   - Entry time, entry price, stop, target
   - Outcome (win/loss/breakeven, R-multiple if known)
   - Notes on execution quality

2. **Create** a trade log entry at `work/projects/tempo-trading/trade-log/YYYY-MM-DD.md` with frontmatter:
   ```yaml
   ---
   date: YYYY-MM-DD
   type: trade-log
   strategies: []
   trades: N
   net_r: X.XX
   tags: [trade-log]
   ---
   ```

3. **Link** each trade to its strategy's wiki page using `[[wiki-links]]`:
   - IFVG trades link to `[[ifvg]]` and `[[tempo-methodology]]`
   - Wick Fade trades link to `[[wick-fade]]`
   - SF Portfolio trades link to `[[sf-portfolio-cluster]]`

4. **Update** the relevant strategy's project file if the trade reveals anything notable (new edge case, execution issue, parameter that needs adjustment).

5. **Check** if today's trades contradict any claims in `wiki/concepts/invalidation-rules.md`. If a trade used a pattern tagged as dead or suspect, flag it immediately.

6. **Append** to today's daily note (`daily/YYYY-MM-DD.md`) under a `## Trades` section with a summary and link to the full trade log.

7. **Append** to `log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] trade-review | N trades, net X.XX R, strategies: [list]
   ```

## Output format

A clean markdown trade log with per-trade entries, R-multiple calculations, and cross-links to every concept, entity, and strategy page mentioned.
