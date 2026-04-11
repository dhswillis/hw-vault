# Session Notes — Feb 18, 2026 (Cowork Session)

## Harrison's Spreadsheet Preferences

**The spreadsheet is for an executive team, not developers.**

1. **Win rate matters most.** The team is trying to generate the most R with the highest probability of success. WR is the primary sort/ranking criteria.
2. **Every strategy needs complete metrics:** R/Trade, Total Trades/Year (annualized), Trades/Day, R/Year, R/Week, Trade WR%, Daily WR%, Weekly WR%. Don't leave columns out — compute what you can, estimate and label what you can't.
3. **R/Day means R per calendar trading day, not R per day traded.** If a strategy fires 0.5 trades/day, R/day accounts for the zero-trade days too. Annualize everything to 258 trading days.
4. **No misleading summaries.** Don't present a single row that blends 80% WR and 11R/day if those come from different studies. Each row = one deployable strategy with internally consistent metrics.
5. **Layman-accessible.** Every abbreviation needs a definition. Added "Summary & Definitions" as the first sheet. N = sample size, R = risk multiple, WR = win rate, etc.
6. **Show the math.** If something is computed, make the formula visible or at least the logic obvious. Don't just present a number — show where it came from.
7. **Don't frame Model 2 as lacking features QC has.** Model 2 has MORE data (footprints, delta, imbalance, 6-TF alignment). QC is the one missing these. The "Gap" sheet should show what QC needs to replicate, not what Model 2 doesn't have.
8. **Data period accuracy matters.** If a finding is from V8b footprints and the data period is unverified, SAY SO. Yellow highlight = unverified. Green = confirmed full year. Don't claim full year if it hasn't been independently confirmed.
9. **Totals and averages at the bottom of every table.** Use Excel formulas, not hardcoded values.
10. **The sniper ruleset is important.** Harrison specifically asked for the high-WR combinations to be called out with their own section. These are the highest-conviction trades.
11. **Don't over-emphasize WHERE data was run.** Harrison said: "I don't think it's correct to say we ran two models, we ran data on QC, Claude Code, and Claude Cowork. We ran a ton of different models with different strategies and variables. You are putting too much emphasis on where we ran the data." Frame by STRATEGY and FILTER, not by platform. QC, Claude Code, Cowork are just execution environments.
12. **"The Filtered Edge"** = the 50%+ WR middle-ground portfolio. Harrison asked for "all trades >50% win rate as another middle ground strategy, but call it something cool." 3 signal legs: BOS >=3TF (51.2%), sweep confirming (58.4%), VA overlap+confirming (50.6%). Combined: 53.7% WR, 1,642 trades/year, +2,604R/year. NOTE: Harrison said don't use the name "Tempo" for strategies.

## Key Corrections Harrison Made This Session

- **"Why do we just show QC?"** — The original "QC vs Model 2 Gap" sheet was QC-centric, framing Model 2 findings as future plans. Fixed: Model 2 HAS all these features tested. QC is the one lacking them.
- **"Footprints weren't run on full 13 months"** — V8b footprint findings claim "Full Year (312 days)" in run log but this is unverified for tick-level computation. Added "Data Period" column with yellow "Unverified" flags.
- **"How does BOS_R only have one trade?"** — The row "BOS R/trade" showed values 1.037 and 1.37, which Harrison read as n=1 trade. Fixed: renamed to "BOS avg R per trade" with note showing n=1,804 BOS trades.
- **"R per day is not R per day traded"** — Harrison caught that R/day needs to account for the full 258-day year, not just days the strategy fires. Fixed the strategy table to show Trades/Year, Trades/Day, R/Year, R/Week all annualized properly.
- **"Add sniper ruleset and combinations"** — The 80%+ WR combos are the most exciting finding. Harrison wanted them called out as a deployable "sniper portfolio" with combined metrics.

## What Was Built/Changed Today

### NQ Test.xlsx Changes
1. **Summary & Definitions sheet** (NEW, position 1) — Executive summary, 13 metric definitions, 4 signal types, 12 filter definitions, V7/V8 study guide, sheet-by-sheet reading guide, 5 caveats
2. **Model 2 Results (V8) sheet** — Major rebuild:
   - Fixed BOS R/trade label
   - "Strategy Options" table with complete per-strategy metrics (R/Trade, Trades/Year, Trades/Day, R/Year, R/Week, Trade WR%, Daily WR%, Weekly WR%)
   - "THE LOGIC" section explaining why portfolio WR is lower but generates more total R
   - Monthly WR section
   - **Sniper Ruleset section** (NEW): 9 proven 75%+ WR combos, combined sniper portfolio (A+B = ~226 trades, ~80% WR, ~893R/year), 7 untested priority stacks, deployment recommendation
3. **QC vs Model 2 Gap sheet** — Rewritten to correctly show Model 2 has footprints/MTF tested, QC doesn't
4. **QC Model 1 Rankings sheet** — Completely rebuilt:
   - All 48 variants (was only top 20)
   - Starting capital $50,000 stated
   - New columns: Signals, Session, BE Mode, Trades/Day, Return %, Annualized R, Stage A/B Sharpe, Sharpe Drift, Stable flag
   - Color coding: green = profitable + stable, red = losing
   - Full pipeline documented: 400 → 88 → 50 → 48
   - Notes section with signal/session/BE mode keys
5. **High Win Rate Combos** — Added summary row with averages
6. **What We Ran** — Already had all 14 analyses documented

### Other Files
- **NEW_SIGNAL_CLASSES_RESEARCH.md** (CREATED) — 594 lines, 7 new signal classes with implementation specs
- **START_HERE.md** (UPDATED) — Added Willis Holdings Brain Layer and New Signal Classes Research sections

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| Baseline R/day | +11.31 | V7f, 258 days, 4,991 trades |
| Baseline trades/day | 19.3 | V7f |
| Best trade WR | 85.3% (sweep conf>=5) | V8a, n=54 |
| Best WR with volume | 80.3% (BOS >=3TF+0contra) | V8g, n=117 |
| Combined sniper portfolio | ~80% WR, ~893R/year, ~226 trades | V8g (computed) |
| Strategy A OOS | +4.31R/day, 65.3% daily WR, 94.4% weekly | V8d, 94 OOS days |
| Strategy B OOS | +11.05R/day, 68.4% daily WR | V8d, 94 OOS days |
| Optimized portfolio | +7.25R/day, Calmar 137.8, DD 16.4R | V7y, full year |
| Starting capital (QC) | $50,000 | main_v4.py |
| Risk per trade (QC) | $300 (0.6%) | main_v4.py |
| Monthly WR (baseline) | 13/13 = 100% | V7g |

## Pending / Not Yet Done

- **Run cross_mining_engine.py on real VPS data** — the 6,502 combo mining hasn't been executed yet. Tools built, need VPS execution.
- **Validate untested sniper stacks** (BOS >=3TF + imbalance, BOS >=3TF + delta, etc.) — P0 priority
- **Daily/weekly/monthly WR for QC variants** — not available from batch output. Need individual equity curve exports.
- **Re-run QC batch with correct slippage (0.5pt) + Model 2 filters** — expected to dramatically improve QC results
- **Kimi K2.5 integration** — Harrison said "do that tomorrow" in prior session
- **Wire new signal classes into intraday_scanner.py** — implement 6 new signals in production
- **Previous session/day bullish/bearish analysis** — Harrison asked about this but said to table it for now

## Harrison's Communication Style

- Trader, not developer. Practical, not academic.
- Types fast, makes typos — parse intent, don't correct spelling.
- Wants results and files, not explanations of what you're going to do.
- Will challenge numbers that don't add up. Double-check before presenting.
- Cares about accuracy over speed. Better to flag something as "unverified" than claim it's confirmed.
- "A layman needs to understand this document" — the audience is an executive/investor team, not quants.
