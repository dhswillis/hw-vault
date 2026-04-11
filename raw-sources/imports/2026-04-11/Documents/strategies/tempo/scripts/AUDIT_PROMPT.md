# Backtest Integrity Audit — IFVG MTF Cascade + Lumi Engine

You are auditing a NQ futures tick-data backtest for correctness. This is a **code audit** — read every line of the trade simulation logic and verify there are zero errors. Do NOT run the code. Just read and reason.

## Files to audit (read ALL of them in full):

1. `composite_strategy.py` (607 lines) — IFVG engine: 15M HTF sweep → LTF FVG entry
2. `lumi_v19_backtest.py` (476 lines) — Lumi engine: 1H sweep → 3M MSS/OB/FVG → limit entry
3. `market_vs_limit_compare.py` (919 lines) — Base utilities: tick loaders, trade simulators, constants
4. `optimized_composite.py` (445 lines) — Per-component post-hoc filters (DailyR, streak, WeeklyR)
5. `portfolio_optimizer_v3.py` (477 lines) — Portfolio-level optimizer with circuit breakers

## What to check (be exhaustive):

### 1. NO LOOKAHEAD BIAS
- **Signal detection**: When an IFVG or sweep is detected on bar `i`, the entry must NOT use any data from bar `i` or later in the detection logic. The entry should be at bar `i+1` open or later.
- **Tick simulation start**: After a signal bar closes, the tick-level simulation must begin at the first tick AFTER that bar's close timestamp, not during the bar.
- **HTF level building**: Levels built from 15M/1H bars must only use bars that have already closed. No peeking at the current bar's high/low/close.
- **Session/time filters**: Verify that time-based filters use the signal bar's time, not future bar times.
- **Prior week high/low**: Must be from the PREVIOUS completed week, not the current week.

### 2. ENTRY LOGIC
- **IFVG entries are MARKET orders** at next-bar open. They MUST have slippage of 0.5 points (1 NQ tick) applied AGAINST the trader:
  - Long: `entry_price += 0.5`
  - Short: `entry_price -= 0.5`
- **Lumi entries are LIMIT orders** at the FVG retrace level. Limit orders should have NO adverse slippage — they fill at the limit price or better. Verify there is no slippage penalty on limit fills.
- **Limit fill detection**: For a limit buy, price must reach ≤ limit price. For a limit sell, price must reach ≥ limit price. Verify the fill condition is correct.

### 3. TARGET (TP) FILL LOGIC
- **TP is a LIMIT order** and must require TICK-THROUGH — price must trade strictly PAST the target, not just touch it:
  - Long TP: requires `price > target` (strict greater than)
  - Short TP: requires `price < target` (strict less than)
- Check this in ALL three files that simulate trades: `sim_trade_hard_stop()`, `run_lumi_day()`, and `sim_trade_market()`.
- When TP fills, the PnL should be exactly `risk_pts * r_target` (the full target amount), not the actual fill price past the target.

### 4. STOP LOSS LOGIC
- **Hard stops** are hit when price touches or passes the stop level. This should use `<=` for long stops and `>=` for short stops (non-strict, since stops are market orders that fill at the stop price or worse).
- When the stop is hit, PnL should be `actual_fill_price - entry` (not the theoretical stop price), since stops can gap through.
- Verify stop is checked BEFORE target on each tick (stop has priority).

### 5. FLATTEN / SESSION CLOSE
- If neither target nor stop is hit by flatten time, the trade must be closed at the current tick price at flatten time.
- Flatten PnL should be `last_price - entry` for longs, `entry - last_price` for shorts.

### 6. RISK CALCULATION
- `risk_pts` = distance from entry to hard stop in points
- For IFVG: stop is at FVG edge + buffer. After slippage adjusts entry, risk_pts must be recalculated as `entry - stop` (long) or `stop - entry` (short).
- Minimum risk_pts floor of 0.5 points.
- `r_result` = `pnl_pts / risk_pts` — verify this ratio is computed correctly.

### 7. COMMISSION
- COMMISSION = 0.50 points per trade (applied to pnl_pts after the trade sim).
- Verify commission is subtracted from every trade's PnL, not just winners.

### 8. POST-HOC FILTERS (optimized_composite.py)
- **DailyR limit**: Once a component's cumulative R for the day hits the limit (e.g., -1.0), remaining trades that day are skipped. Verify trades are processed in chronological order and the filter is applied BEFORE adding the trade to results.
- **Streak limit**: After N consecutive losses, skip the next trade. A "loss" should be defined consistently (e.g., pnl_pts < 0 or r_result < 0). Verify the streak resets on a win.
- **WeeklyR limit**: Same as DailyR but aggregated by ISO week.
- These filters must NOT change the trade outcomes — they only decide whether to TAKE the trade.

### 9. PORTFOLIO CIRCUIT BREAKERS (portfolio_optimizer_v3.py)
- Portfolio daily R limit: once combined portfolio R for the day hits the limit, ALL remaining trades that day are skipped.
- Portfolio weekly R limit: same but by week.
- Verify trades from ALL components are merged and sorted chronologically before applying breakers.
- Verify the circuit breaker checks the cumulative R BEFORE including the current trade (not after).

### 10. DATA INTEGRITY
- Tick prices loaded from parquet files — verify no NaN/null handling issues.
- Bar OHLC constructed from ticks via resample — verify open/high/low/close are correct aggregations.
- `bar_end` tick indices must point to the LAST tick within each bar, not the first tick of the next bar.
- Day boundaries: verify each day's data is processed independently (no bleed between days).

## Output format:

For each check above, state:
- ✅ **PASS** — with a one-line explanation of why it's correct
- ❌ **FAIL** — with the exact file, line number, code snippet, what's wrong, and what it should be
- ⚠️ **WARN** — for things that are technically correct but could be improved

End with a summary table of all findings and an overall PASS/FAIL verdict.
