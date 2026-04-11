# Autonomous Mining Run — clean_backtest.py Parameter Sweep

You are running an autonomous overnight mining session. DO NOT ASK QUESTIONS. DO NOT STOP. If something fails, log the error, skip it, and move on. Use `--dangerously-skip-permissions`.

## CRITICAL: Read Before Anything Else
1. Read `CLAUDE.md` — it has the ground truth, validated findings, and documented mistakes
2. Read `CONTEXT.md` — it has the full strategy context and prior results
3. Read `clean_backtest.py` top to bottom — this is your ONLY backtesting engine. Do NOT create new frameworks.

## YOUR SINGLE BACKTESTING ENGINE
`clean_backtest.py` in this directory is a proven, debugged, single-file backtester. It:
- Loads Databento .dbn.zst files from `./databento/` (312 files, Feb 2025 - Feb 2026)
- Builds 1M candles, finds swings (confirmed), finds BOS, finds FVGs, matches BOS→FVG→fill
- Simulates trades with pure trailing (trigger + buffer, no BE, no targets)
- Saves every trade to CSV
- Has explicit look-ahead prevention at every step

**YOU MUST USE THIS FILE.** Modify its config block at the top to test different parameters. Run it, save the CSV, analyze results. That's it. Do NOT write a new backtester. Do NOT create a "mining framework." Do NOT abstract anything. Just change the config, run it, save results.

## Current Baseline (1.5pt min stops, session kill at 19:45 UTC)
- 4,429 trades, 50.7% WR, +0.90 avg R, +15.36 R/day
- Max DD: -16.73R (trade-by-trade), Calmar 237
- Trail: trigger=0.5R, buffer=0.1R
- Session: 13:00-20:00 UTC, last entry 19:45 UTC
- Every month profitable, 100% weekly WR

## PHASE 1: Trail Parameter Grid (HIGHEST PRIORITY)
The trailing stop is the entire exit mechanism. Sweep these combos:

| Trigger R | Buffer R |
|-----------|----------|
| 0.25 | 0.05, 0.10, 0.15 |
| 0.50 | 0.05, 0.10, 0.15, 0.20 |
| 0.75 | 0.10, 0.15, 0.20 |
| 1.00 | 0.10, 0.15, 0.20, 0.25 |
| 1.50 | 0.15, 0.20, 0.25, 0.30 |
| 2.00 | 0.20, 0.25, 0.30 |

That's ~20 combos. For each:
1. Edit `TRAIL_TRIGGER_R` and `TRAIL_BUFFER_R` in clean_backtest.py
2. Run: `python clean_backtest.py --data-dir ./databento`
3. Rename output: `mv clean_backtest_results.csv results/trail_T{trigger}_B{buffer}.csv`
4. Record: trades, WR, avg_R, total_R, R/day, max_DD, calmar

Create `results/` directory first: `mkdir -p results`

Save a summary JSON after ALL combos complete:
```json
{
  "phase": "trail_grid",
  "baseline": {"trigger": 0.5, "buffer": 0.1, "trades": 4429, "wr": 50.7, "avg_r": 0.90, "r_day": 15.36, "max_dd": -16.73, "calmar": 237},
  "results": [
    {"trigger": 0.25, "buffer": 0.05, "trades": ..., "wr": ..., ...},
    ...
  ]
}
```

## PHASE 2: Risk Range Sweep
Test minimum stop distances:

| RISK_MIN_PTS | RISK_MAX_PTS |
|-------------|-------------|
| 1.0 | 50.0 |
| 1.5 | 50.0 | ← current baseline
| 2.0 | 50.0 |
| 2.5 | 50.0 |
| 3.0 | 50.0 |
| 1.5 | 30.0 |
| 1.5 | 20.0 |
| 1.5 | 15.0 |
| 1.5 | 10.0 |

Use best trail params from Phase 1. Save CSVs as `results/risk_min{min}_max{max}.csv`.

## PHASE 3: BOS Displacement Sweep
Test `BOS_MIN_DISPLACEMENT`:
- 0.5, 1.0, 1.5, 2.0 (current), 2.5, 3.0, 4.0, 5.0

Use best trail + risk params from Phases 1-2.

## PHASE 4: FVG Parameters
Test combinations:
- `FVG_MAX_FILL_BARS`: 10, 20, 30, 50 (current), 75, 100
- `BOS_FVG_MAX_BARS`: 5, 10, 15 (current), 20, 30

That's 30 combos. Save as `results/fvg_fill{fill}_bos{bos}.csv`.

## PHASE 5: Session Windows
Test different session times (all UTC):
- Full US: 13:00 - 20:00 (current)
- NY AM only: 13:30 - 16:00
- NY PM only: 16:00 - 20:00
- Extended AM: 12:00 - 16:00
- London + NY overlap: 08:00 - 16:00
- London only: 08:00 - 13:00

Also test LAST_ENTRY cutoffs: 15min (current), 30min, 45min, 60min before SESSION_END.

## PHASE 6: Swing Lookback
Test `SWING_LOOKBACK`: 2, 3 (current), 4, 5, 7

## PHASE 7: Entry Mode
Test `ENTRY_MODE`:
- "fvg_edge" (current) — enter at the FVG boundary closest to entry direction
- "fvg_midpoint" — enter at FVG midpoint (you may need to implement this if not already done)

## PHASE 8: Composite Best Config
Take the best parameter from EACH phase and combine them into one run. Compare to baseline.

Then test 2nd-best and 3rd-best alternatives for each parameter to check for overfitting. If the "best" combo is dramatically better than alternatives, it's likely overfit.

## OUTPUT FORMAT
After each phase, save a summary to `results/phase_{N}_summary.json` with:
```json
{
  "phase": N,
  "phase_name": "...",
  "best_config": {...},
  "all_results": [...],
  "baseline_comparison": "...",
  "notes": "..."
}
```

After ALL phases, write `results/FINAL_SUMMARY.md` with:
- Best overall config and its metrics
- Comparison to baseline
- Overfitting risk assessment
- Recommended parameters for live trading

## ERROR PREVENTION RULES

### DO NOT:
- Create new Python files. Only modify clean_backtest.py config block.
- Create abstraction layers, frameworks, or wrapper scripts.
- Use daily aggregation for drawdown (trade-by-trade ONLY).
- Trust any result with Calmar > 500 — investigate before recording.
- Trust any result with WR > 70% — likely a bug.
- Trust any single trade > 50R — check if tiny stop caused it.
- Skip phases. Do them in order.
- Ask me questions. Just run.

### DO:
- Log every run's parameters and results.
- If a run crashes, log the error and skip to next parameter combo.
- After each phase, reset clean_backtest.py to baseline config before starting next phase.
- Spot-check: for each phase's best result, look at the top 5 biggest winning trades in the CSV. Are the risk_pts reasonable? Are the bars_held reasonable? Flag anything suspicious.
- Update CLAUDE.md ground truth with any validated new findings.

### SANITY CHECKS FOR EVERY RUN:
- If WR > 65%: flag and investigate
- If avg loss > -1.2R: something is wrong with stop logic
- If max single trade > 50R: tiny stop issue, check risk_pts distribution
- If Calmar > 400: likely inflated, check DD calculation
- If 0 session_end exits across 4000+ trades: confirm session kill is working

## TIME ESTIMATE
- Each full run takes ~3-4 minutes on 312 files
- Phase 1 (20 combos): ~60-80 min
- Phase 2 (9 combos): ~30 min
- Phase 3 (8 combos): ~25 min
- Phase 4 (30 combos): ~100 min
- Phase 5 (10 combos): ~35 min
- Phase 6 (5 combos): ~15 min
- Phase 7 (2 combos): ~8 min
- Phase 8 (3-5 combos): ~15 min
- Total: ~5-6 hours

START NOW. Phase 1 first. No questions.
