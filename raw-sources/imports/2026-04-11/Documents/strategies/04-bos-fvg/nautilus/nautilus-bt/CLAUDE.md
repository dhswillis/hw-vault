# NautilusTrader NQ Futures Backtesting Project

---

## ⚠️ GROUND TRUTH — READ THIS FIRST, UPDATE IT WHEN YOU LEARN SOMETHING NEW

**This section is the single source of truth for what is validated, what is broken, and what mistakes have been made. If you discover a new bug, confirm a finding, or overturn a prior conclusion — UPDATE THIS SECTION BEFORE DOING ANYTHING ELSE. Do not let findings accumulate only in conversation. Write them here.**

**Last updated: 2026-02-20 (Parameter sweep 87 configs complete)**

### VALIDATED FINDINGS (proven with evidence)

1. **BE+Trail exit is dead.** V10aa tested it cleanly: 21.8% WR on 1M bars, 30.7% on 15S bars. Mining's original 63.5% WR used look-ahead bias (see mistake #1 below). Do not use BE+Trail (breakeven at 1R, trail from 2R). V11 re-confirmed: every fixed_XR_BE1 config underperforms its no-BE equivalent.

2. **Pure trailing exit works on 1M bars.** Scanner BOS_FVG with trail 0.5/0.1 on 1M bars: 2932 signals, 51.7% WR, +0.687 avgR, +6.46 R/day, Calmar 189.2. V11 full dataset, mining framework.

3. **Signal detection is correct.** Scanner finds 19,753 signals across 8 types over 312 days. BOS_FVG: 2932, engulfing_key_level: 5710, sweep_reversal: 3192, failed_breakout: 3111.

4. **BOS_FVG is the dominant signal.** Only signal type with consistently strong positive avgR across all exit methods. Other signals (engulfing_key_level, fib_retracement, etc.) have high WR with trail_0.25_0.1 but very low avgR (+0.06 to +0.17R).

5. **Delta alignment is dead (after look-ahead fix).** V11 Phase 5: delta_aligned 50.8% WR, delta_opposed 52.4% WR — NO edge. Previous "75% WR Calmar 526" result was 100% look-ahead bias (using entry bar's delta before it closed). FIXED: use previous bar's delta.

6. **Candlestick patterns at entry are dead (after look-ahead fix).** V11 Phase 8: engulfing_aligned 48.8% WR (was 90.4%), strong_body_aligned 49.5% WR (was 100%). ALL candlestick "edge" was look-ahead from using the unclosed entry bar's OHLC. FIXED: use previous completed bar.

7. **Clean alignment (3m/5m/15m/1h completed bars) provides NO filtering edge.** align>=0: 51.7% WR, align>=4: 52.2% WR — flat across all thresholds. Consistent with V10p findings.

8. **Day of week: Friday slightly weaker but within noise.** Mon 52.6%, Tue 54.5%, Wed 52.7%, Thu 50.6%, Fri 48.7%. No actionable day-of-week filter.

9. **Confirmation candle entries increase trade count without hurting edge.** Rejection on 1M: 6403 trades, 68.5% WR, +0.310 avgR (trail 0.25/0.1). Midpoint_away: 4662 trades, 72.3% WR, +0.321 avgR. Both promising but need no-overlap validation.

10. **Trail trigger 0.25R > 0.5R > 1.0R.** Tighter triggers capture more winners but lower avgR per trade. Best Calmar: trig=0.25R buf=0.05R (Cal 206.4, +6.72 R/day). Trail sensitivity is REAL.

11. **NautilusTrader full 312-file run VALIDATES positive expectancy across all entry modes.** Batched by contract month (5 batches to avoid OOM). Results file: `results/backtest_20260220_083937.json`. Key findings:
    - **44,751 total signals** across 315 trading days (24h, all sessions)
    - **All 8 entry modes profitable** (both 24h and US-session filtered)
    - **rejection_bar US session**: 9,188 trades, 55.6% WR, +0.286 avgR, +10.20 R/day, 86.4% daily WR
    - **full_fill_bar US session**: 6,915 trades, 60.6% WR, +0.247 avgR, +6.61 R/day
    - **rejection_mom_bar US session**: 5,902 trades, 64.3% WR, +0.200 avgR, +4.60 R/day
    - **Edge consistent across all sessions**: Asia/London/US all show positive avgR (+0.25-0.36)
    - **NOT apples-to-apples with mining baseline**: Nautilus uses bar.close market entry while mining uses FVG-edge limit entry. Mining gets better fills → higher avgR (+0.687 vs +0.286) but fewer trades (2932 vs 9188).
    - **_mid stop variants**: lower WR but higher avgR (wider stop = more room = bigger runners)
    - **MaxDD 16-34R trade-by-trade** — much higher than mining's per-day DD. Calmar 0.2-1.1 (low).
    - **The 3x trade count difference** is because Nautilus fires on every bar.close confirmation, while mining waits for price to touch the FVG edge (limit order fill). Mining is more selective.

12. **OOM fix: batch by contract month.** Original runner loaded all 312 files into one engine — crashed at file 127/312 (38M+ ticks in 24GB RAM). Fixed by processing each contract's files independently (5 batches, ~60-85 files each), then aggregating results. File: `run_backtest.py` v8_pure_trail_batched.

### SUSPECT / UNVALIDATED

- **Cross-TF BOS+FVG combos** (BOS on higher TF, FVG on lower TF) — e.g. BOS_5m_FVG_1m showed 56.4% WR but with massive trade overlap. Needs no-overlap validation and timing correction. V11c audit running.
- **Tight stop placements** — tight_2pt showed +20.14 R/day but 33.6% WR. High R because risk is tiny. Need slippage testing.
- **15S bar trailing results** — 82.5% WR is bar-smoothing artifact. A 2-tick trail buffer is unrealistic in live execution. Do NOT trust 15S trail results for live trading.
- **Confirmation candle entries with no overlap filtering** — the high WR may partially come from trading the same signal multiple times.

### MISTAKES MADE — DO NOT REPEAT

1. **Look-ahead bias in mining alignment (V10i).** Forming-bar alignment inflated WR from ~38% to 91%. Fixed by using only COMPLETED bars for MTF trend. All mining results from before this fix are invalid.

2. **Same-bar stop bug.** Comparing bar.low/high against stop price on the SAME bar the entry occurs. Every trade gets instantly stopped. Fix: stops activate on the NEXT bar after entry. If you see 100% loss rate on anything, it's a bug, not a finding.

3. **Confidently narrating results before verifying.** Multiple times, results were declared as findings ("confirmation candles are dead", "bar.close entry kills R:R") when they were actually bugs. Rule: if a result is extreme (100% loss, 0% win, >90% WR), question it before building a narrative.

4. **Confusing mining framework results with NautilusTrader results.** Mining = bar-by-bar Python scanner. NautilusTrader = event-driven backtest engine. Always state which engine produced a result.

5. **Claude Code re-adding dead parameters.** Claude Code added BE at 1R and max hold 120 bars back into v7, which contradicts the pure trailing finding. Always check that the strategy params match what was actually validated.

6. **Delta on entry bar look-ahead (V11).** Using `c1m['delta'].iloc[entry_idx]` — the bar hasn't closed at entry time. Fix: use `entry_idx - 1` (previous completed bar). Result collapsed from 75% WR Calmar 526 → 50.8% WR (no edge).

7. **Candlestick pattern on entry bar look-ahead (V11).** Classifying patterns using `candles.iloc[idx]` at entry — bar hasn't closed. Fix: use `candles.iloc[idx - 1]`. Result collapsed from engulfing 90.4% → 48.8% (no edge).

8. **Same-TF BOS+FVG look-ahead (V11 Phase 6).** When BOS and FVG use the same timeframe (3M+3M, 5M+5M), the displacement candle that causes the BOS IS the FVG. The "fill" happens inside the bar that caused the BOS — you can't actually trade that. This is why same-TF combos show 73-82% WR while cross-TF combos show 54-58% WR. The 18-25pp WR difference is pure look-ahead.

9. **15S bar-smoothing inflation (V11).** Trailing at 0.1R buffer on 15S bars = 2 NQ ticks trail stop. Bar-level simulation smooths over tick-level whipsaw. The 82.5% WR on 15S is an artifact, NOT a real edge. Do not build strategies based on 15S trailing.

10. **check_fvg_fills argument swap (V11).** Called `engine.check_fvg_fills(candles, fvgs)` instead of `engine.check_fvg_fills(fvgs, candles)`. Produced 0 signals. Fix: correct argument order.

11. **YYYYMMDD date format parsing (V11).** Python 3.9 `date.fromisoformat()` can't parse YYYYMMDD. Use `datetime.strptime(date_str, '%Y%m%d').date()` instead. Was silently producing 0 results for day-of-week and weekly stats.

12. **Calmar ratios are inflated by daily aggregation.** Computing max DD on daily totals (not trade-by-trade) produces smaller DDs when daily WR is high. Always report trade-by-trade DD alongside daily DD for realistic comparison.

13. **87-config parameter sweep validates BOS_FVG structural edge.** clean_backtest.py across 8 phases (trail, risk, displacement, FVG params, sessions, swing lookback, entry mode, composites). Every single config tested is profitable. avgR locked at +0.87 to +0.93 regardless of params. The edge is structural, not parameter-dependent. Full results: `results/FINAL_SUMMARY.md` and `results/phase_*_summary.json`.

14. **Swing lookback=2 is the biggest improvement.** Cal 329.1, +19.04 R/day (vs lb=3 baseline Cal 237.0). +39% Calmar from this single change. More swing points = more signals = more R/day at same avgR.

15. **Composite Best config: Cal 335.3, +21.81 R/day, 94.6% DWR, 100% WWR.** 6091 trades over 312 days. +42% R/day and +41% Calmar over baseline. 2nd-best params still +24% over baseline → low overfitting risk.

16. **Max R/day config (Cal 542.9, +33.15 R/day) is UNRELIABLE.** Contains 1pt-risk trades with 0.05R trail buffer = 0.05pts = less than 1 NQ tick. Top winner 209R on 1.2pt risk is bar-smoothing artifact. Do NOT use for production.

### CURRENT BEST STRATEGY PARAMS (validated via 87-config sweep)

```
SWING_LOOKBACK = 2          # was 3 — biggest single improvement (+39% Cal)
BOS_MIN_DISPLACEMENT = 1.0  # was 2.0 — more trades, same quality
FVG_MAX_FILL_BARS = 30      # was 50 — fresher FVGs slightly better
BOS_FVG_MAX_BARS = 20       # was 15 — wider window catches more signals
RISK_MIN_PTS = 1.5          # unchanged
RISK_MAX_PTS = 20.0         # was 50.0 — removes sloppy wide-stop trades
TRAIL_TRIGGER_R = 0.75      # was 0.5 — marginal improvement
TRAIL_BUFFER_R = 0.1        # unchanged — universal sweet spot
Entry: FVG edge
Session: 13:00-20:00 UTC (8 AM - 3 PM ET)
Last entry: 19:45 UTC (2:45 PM ET)
No breakeven, no max hold, no max target
```

Result: 6091 trades, 47.6% WR, +0.924 avgR, +21.81 R/day, Cal 335.3, 94.6% DWR, 100% WWR

### WHAT NEEDS TO HAPPEN NEXT

1. ~~Full 312-file validation of v7 4-entry-mode results in mining framework~~ DONE (finding #11)
2. ~~Port pure trailing (no BE, no max hold) into NautilusTrader and validate~~ DONE (finding #11)
3. ~~Parameter sweep (87 configs, 8 phases)~~ DONE (findings #13-16)
4. Run walk-forward validation (4-quarter split) on Composite Best to confirm OOS holds
5. Add 1pt and 2pt slippage tests on Composite Best
6. Port Composite Best params to NautilusTrader for tick-level validation
7. Run plot_trades.py to visually verify trades look correct
8. Test multi-signal portfolio with optimized BOS_FVG params as foundation
6. Run mining clean_backtest.py parameter sweep (Phase 1-8) — ongoing
7. Re-run portfolio construction with pure trailing across all signals

---

## Project Overview

This is a NautilusTrader backtesting project for NQ E-mini Nasdaq 100 futures. It ports a proven signal detection system from a custom Python mining framework (`full_mine_v2.py`) into NautilusTrader's event-driven architecture for proper backtesting with backtest-to-live parity.

## What's Been Done (in Cowork)

1. **Comprehensive signal mining** across 34,323 trades, 258 days (Feb 2025 - Feb 2026)
2. **Found and fixed a critical look-ahead bias** in V10i alignment (91% WR → 38% when fixed)
3. **Confirmed our alignment is clean** (3-bar HH/HL trend on completed bars only)
4. **Portfolio construction**: 7-strategy portfolio generating +20,662R total (but with BE+Trail — needs re-validation with pure trailing)
5. **Correlation analysis**: avg pairwise correlation 0.134 — genuine diversification
6. **Scaffolded this NautilusTrader project** with the BOS_FVG strategy ported
7. **V10aa proved BE+Trail dead**, pure trailing is the edge
8. **V7 strategy built** with 4 entry confirmation modes × 2 stop types = 8 parallel tracks
9. **Same-bar stop bug found and fixed** by user catching 100% loss rate as obviously wrong

## Key Files

- `strategies/bos_fvg_strategy.py` — Main BOS_FVG strategy, v7 with 4 entry modes × 2 stop types
- `run_backtest.py` — Backtest runner with Databento data loading
- `mine_fvg_confirm.py` — Standalone confirmation candle mining script
- `plot_trades.py` — Trade visualization (candlestick charts with BOS/FVG/entry/exit/trail)
- `MINING_SIGNAL_SPEC.md` — Detailed mining logic spec for matching NautilusTrader to mining
- `../databento/` or `~/trading_operator/data2/` — .trades.dbn.zst tick data files
- `../context-engine/full_mine_v2.py` — Original mining script (reference implementation)

## Phase Plan

### Phase 1: Validate Pure Trailing in NautilusTrader ← CURRENT
1. Ensure v7 strategy uses pure trailing (no BE, no max hold)
2. Run full backtest across all data files
3. Compare to mining's 75.8% WR — expect similar but not identical

### Phase 2: Full Signal Sweep with Pure Trailing
Port remaining signals with pure trailing exit:
- `sweep_wick_sweep`, `sweep_close_through`, `swing_failure`
- `fvg_exit_fvg_stop` / `fvg_exit_body_stop`, `sweep_fvg_combo`

### Phase 3: Portfolio Re-Construction
- Re-run correlation analysis with pure trailing results
- Build new portfolio with validated exits

### Phase 4: Live Execution Bridge
Architecture: NautilusTrader → CrossTrade REST API → NinjaTrader → Replikanto → Prop Accounts

## Critical Implementation Details

### BOS_FVG Signal Logic
1. Detect swing highs/lows using 3-bar lookback (HH/HL pattern on COMPLETED bars only)
2. Detect BOS: close above confirmed swing high (bullish) or below swing low (bearish)
3. Detect FVGs: 3-bar gap pattern (bar[i].low > bar[i-2].high for bullish)
4. Signal fires when: FVG fills within 15 bars after a same-direction BOS
5. Entry varies by mode — see strategy file for current 4 confirmation modes

### MTF Alignment (CLEAN — No Look-Ahead)
- 7 timeframes: 1M, 3M, 5M, 10M, 15M, 1H, 4H
- Trend = last 3 COMPLETED bars: close[-1] - close[0] vs threshold
- Thresholds: 1M=2pt, 3M=3pt, 5M=5pt, 10M=8pt, 15M=10pt, 1H=15pt, 4H=25pt
- Count aligned (trend matches direction) and contra (trend opposes)

### Session Filter
- Only trade 8 AM - 3 PM ET (13:00-20:00 UTC)
- Still feed bars to engine outside session for structure detection

## Databento Data Format
Files are `.trades.dbn.zst` — Databento Binary Encoding, zstd compressed.
Each file contains one day of NQ tick data (MBP-1 or trades schema).
Use `DatabentoDataLoader().from_dbn_file(path=fpath, as_legacy_cython=True)` to load.
