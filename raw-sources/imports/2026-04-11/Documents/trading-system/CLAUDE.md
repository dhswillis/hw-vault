# Tempo Trading System — Claude Instructions

**FIRST: Read `START_HERE.md` in this directory.** It contains the full project context, current state, infrastructure map, credentials, and links to all reference docs.

**THEN: Read `CLAUDE_BOOTSTRAP.md`** for technical setup details, credential locations, and integration configs.

**BEFORE ENDING ANY SESSION:** Update `START_HERE.md` "Current State" section with what changed, then commit and push to git.

## CRITICAL CORRECTIONS (READ BEFORE ANYTHING ELSE)
- **BOS_FVG DOES NOT WORK.** The research journal shows backtesting results but Harrison has confirmed multiple times that BOS_FVG is not validated and does not work in practice. DO NOT cite BOS_FVG as a proven strategy. DO NOT claim it has edge. The backtesting numbers (Calmar 263.6, +1.141 avgR) are misleading.
- **NOTHING from the backtesting research is validated.** V7/V8 was killed by look-ahead. The V10t "clean" results are approximately breakeven after commissions. The research journal documents what was TESTED, not what WORKS.
- **v24 (IFVG MTF cascade) is the ONLY strategy currently on sim** via NinjaTrader. It has NOT yet proven itself — it is still collecting data.
- **Harrison is a SUBSCRIBER in Tempo Trades, NOT the educator.** He watches someone else's live stream. He is not the one teaching.
- **Do not claim any strategy is validated unless Harrison explicitly confirms it in the current session.**
- **RT (R target) must ALWAYS appear in every results table.** No exceptions.

## Key Rules
- Harrison is a trader, not a developer. Be practical, not academic.
- Build things, don't describe things. He wants code and results.
- Credentials are in `.local/config.json` (gitignored, never commit secrets)
- VPS: root@172.93.181.209
- GitHub: dhswillis/trading-system (PAT in .local/config.json, expires May 15 2026)
- QC Project: 28083727
- Context engine spec: `~/Documents/Tempo_Context_Engine_Spec.docx`
- If it's not documented in START_HERE.md, it doesn't exist next session.

## Critical Mental Model: Portfolio of Non-Correlated Models
- Tempo uses MULTIPLE signal models with DIFFERENT entry logic — this is intentional
- Model 1 (QC): Sweep + IFVG, market orders, multi-timeframe
- Model 2 (Backtester): BOS/sweep/VA/PGF, limit orders, 5M+1M
- These are NOT supposed to match — they generate different trade populations for diversification
- Goal: 10R+ per day COMBINED across all models
- Don't try to align them or make them match — non-correlation IS the strategy
- Position sizing normalizes risk to 1R per trade — gap/stop size doesn't change risk
- B/E kills Model 2 edge (validated full year). Don't add B/E to backtester signals.

## Spreadsheet Preferences (NQ Test.xlsx)
Harrison has 12 non-negotiable rules for the spreadsheet:
1. WR matters most — always sort and rank by win rate
2. Every strategy needs complete metrics: R/Trade, Trades/Year, Trades/Day, R/Year, R/Week, Trade WR%, Daily WR%, Weekly WR%
3. R/Day = R per calendar trading day (258 days), NOT per day traded
4. No misleading summaries — don't combine findings from different studies as if they're one result
5. Layman-accessible — executive audience, not quants
6. Show the math — break down how portfolio compositions produce their numbers
7. Don't frame research as lacking features QC has — frame by STRATEGY and FILTER
8. Data period accuracy — yellow highlight = unverified period
9. Totals/averages at bottom of every table
10. Sniper ruleset called out separately with all constituent combos
11. Don't over-emphasize WHERE data was run (QC vs Claude Code vs Cowork) — frame by what was tested
12. Don't use the name "Tempo" for strategies — use Willis Holdings or the strategy name

Formatting: Navy section headers (white text), light blue column headers, Arial throughout, all numbers centered, column A left-aligned, commentary merged across full table width, landscape print layout, fit-to-width.

Portfolio names: Sniper (~80% WR), Sharpshooter (~72% WR), The Filtered Edge (53.7% WR), Strategy A OOS, Baseline.

## Document Hierarchy
1. `CLAUDE.md` (this file) — Claude Code reads this automatically
2. `START_HERE.md` — Full project state, infrastructure, what's next
3. `CLAUDE_BOOTSTRAP.md` — Technical details, credentials, integrations
4. `CONTEXT.md` — Strategy mechanics, signals, execution parameters, batch results
5. `Tempo_Context_Engine_Spec.docx` — Engineering spec for context engine build
