---
Tempo Trading System — Claude Instructions FIRST: Read START_HERE.md in this directory. It contains the full project context, current state, infrastructure map, credentials, and links to all reference docs. THEN: Read CLAUDE_BOOTSTRAP.md for technical setup details, credential locations, and integration configs. BEFORE ENDING ANY SESSION: Update START_HERE.md "Current State" section with what changed, then commit and push to git. Key Rules Harrison is a trader, not a developer. Be practical, not academic. Build things, don't describe things. He wants code and results. Credentials are in .local/config.json (gitignored, never commit secrets) VPS: root@172.93.181.209 GitHub: dhswillis/trading-system (PAT in .local/config.json, expires May 15 2026) QC Project: 28083727 Context engine spec: ~/Documents/Tempo_Context_Engine_Spec.docx If it's not documented in START_HERE.md, it doesn't exist next session. Critical Mental Model: Portfolio of Non-Correlated Models Tempo uses MULTIPLE signal models with DIFFERENT entry logic — this is intentional Model 1 (QC): Sweep + IFVG, market orders, multi-timeframe Model 2 (Backtester): BOS/sweep/VA/PGF, limit orders, 5M+1M These are NOT supposed to match — they generate different trade populations for diversification Goal: 10R+ per day COMBINED across all models Don't try to align them or make them match — non-correlation IS the strategy Position sizing normalizes risk to 1R per trade — gap/stop size doesn't change risk B/E kills Model 2 edge (validated full year). Don't add B/E to backtester signals. Spreadsheet Preferences (NQ Test.xlsx)
The spreadsheet is for an executive/investor team, NOT developers.
Win rate is the primary ranking criteria — the team wants highest probability of success.
Every strategy needs: R/Trade, Trades/Year, Trades/Day, R/Year, R/Week, Trade WR%, Daily WR%, Weekly WR%.
R/Day = R per calendar trading day (annualized to 258 days), NOT R per day the strategy trades.
Don't blend metrics from different studies into one row. Each row = one deployable strategy.
Layman-accessible: every abbreviation defined, Summary & Definitions is first sheet.
Model 2 has MORE data than QC (footprints, delta, 6-TF). Don't frame Model 2 as lacking features.
Don't over-emphasize WHERE data was run (QC vs Claude Code vs Cowork). Frame by STRATEGY and FILTER, not platform.
Flag unverified data periods in yellow. Don't claim full year without confirmation.
Totals/averages at bottom of every table using Excel formulas.
Full session preferences: SESSION_NOTES_FEB18.md

Document Hierarchy
CLAUDE.md (this file) — Claude Code reads this automatically
START_HERE.md — Full project state, infrastructure, what's next
SESSION_NOTES_FEB18.md — Spreadsheet preferences, corrections, what was built Feb 18
CLAUDE_BOOTSTRAP.md — Technical details, credentials, integrations
CONTEXT.md — Strategy mechanics, signals, execution parameters, batch results
Tempo_Context_Engine_Spec.docx — Engineering spec for context engine build
