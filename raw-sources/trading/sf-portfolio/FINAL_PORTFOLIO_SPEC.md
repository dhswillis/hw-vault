# NQ+ES Portfolio — Final Specification
**Date**: March 21, 2026 | **Version**: Clean (all look-ahead bugs fixed) | **Dataset**: 260 trading days

## Performance Summary

| Metric | Value |
|--------|-------|
| **R/Day** | **+3.34** |
| **Total R (260d)** | **+868.3** |
| **Calmar Ratio** | **53.1** |
| **Sharpe Ratio** | **6.57** |
| **Max Drawdown** | **16.4R** |
| **Winning Weeks** | **47/54 (87%)** |
| **Losing Weeks** | **7** |
| **Winning Months** | **13/13 (100%)** |
| **Daily Win Rate** | **68%** |

## Architecture

Two instruments traded simultaneously: **NQ (Nasdaq-100 futures)** and **ES (S&P 500 futures)**.

- **NQ**: 10 signal legs (sweep-fail, VWAP, AMD, late fade, London fade)
- **ES**: 3 signal legs (VWAP multi-touch, late fade, AMD)
- **Runner**: 1H London sweep-fail with BE5 management on NQ

All signals are **zero look-ahead** verified (8 bugs found and fixed during development).

---

## Signal Specifications

### Signal 1: NQ Runner (1H London PDH Sweep-Fail, BE5)
- **Instrument**: NQ
- **Session**: London (480–720 min UTC, 8:00–12:00 UTC)
- **Timeframe**: 60-minute bars
- **Concept**: Sweep-fail on completed prior 1H bar high/low within London session
- **Direction filter**: PDH_SW (prior day high sweep direction)
- **Condition**: Pre-market range ≥ 100 pts
- **Entry**: At tick where price fails back through reference level after sweep
- **Stop**: 5 pts from entry
- **Management**: Break-even at +5R (25 pts). Hold to London close (780 min = 1:00 PM UTC)
- **R/Day**: +1.05

### Signal 2: NQ SF_PRE (15m Pre-NY Sweep-Fail)
- **Instrument**: NQ
- **Session**: Pre-NY (720–870 min UTC)
- **Timeframe**: 15-minute bars
- **Direction filter**: PDH_SW
- **Skip**: Thursday
- **Stop**: 40 pts | **Target**: 1.5R (60 pts) | **Flat**: 1255 min
- **R/Day**: +0.13

### Signal 3: NQ SF_NY1 (15m NY First 30 Sweep-Fail)
- **Instrument**: NQ
- **Session**: NY AM first 30 min (870–930 min UTC)
- **Timeframe**: 15-minute bars
- **Direction filter**: None
- **Skip**: Friday
- **Stop**: 20 pts | **Target**: 1.5R (30 pts) | **Flat**: 1255 min
- **R/Day**: +0.12

### Signal 4: NQ SF_NL (60m NY Late Sweep-Fail)
- **Instrument**: NQ
- **Session**: NY Late (1080–1260 min UTC)
- **Timeframe**: 60-minute bars
- **Direction filter**: Leppyrd bias (D-2 close vs D-3 RTH range)
- **Conditions**: Prior day range > 234 pts AND pre-market range ≥ 200 pts
- **Stop**: 25 pts | **Target**: 2.0R (50 pts) | **Flat**: 1260 min
- **R/Day**: +0.09

### Signal 5: NQ SF_LL15 (15m London Late Sweep-Fail)
- **Instrument**: NQ
- **Session**: London Late (600–720 min UTC)
- **Timeframe**: 15-minute bars
- **Direction filter**: PDH_SW
- **Stop**: 25 pts | **Target**: 1.0R (25 pts) | **Flat**: 1255 min
- **R/Day**: +0.15

### Signal 6: NQ VWAP Multi-Touch (VWPM)
- **Instrument**: NQ
- **Session**: RTH (870–1260 min UTC), skip first 30 min
- **Concept**: When price returns to VWAP (within ±2 pts) after moving ≥10 pts away, enter in direction of prior move
- **VWAP**: Computed incrementally from RTH ticks only (no look-ahead)
- **Max trades**: 3 per day
- **Direction**: Same as prior established side (was above VWAP → long on touch, was below → short)
- **Stop**: 10 pts | **Target**: 1.0R (10 pts) | **Flat**: 1255 min
- **Reset**: After entry, side resets to neutral until price moves ≥10 pts from VWAP again
- **R/Day**: +0.38

### Signal 7: NQ VWT (VWAP Touch Single)
- **Instrument**: NQ
- **Session**: RTH, skip first 30 min
- **Concept**: First VWAP touch after establishing side (price > VWAP+5 = long side, < VWAP-5 = short side)
- **Skip**: Thursday
- **Entry**: When price touches VWAP (within ±1 pt) from established side
- **Stop**: 15 pts | **Target**: 1.5R (22.5 pts) | **Flat**: 1255 min
- **Max trades**: 1 per day
- **R/Day**: +0.10

### Signal 8: NQ AMD London (Fade First 15 Minutes)
- **Instrument**: NQ
- **Session**: London open (480–495 min UTC for detection, entry at 495+)
- **Concept**: ICT AMD — first move is manipulation, fade it
- **Direction**: Opposite of first 15-min bar direction (bar 480–495 close vs open)
- **Entry**: First tick at minute ≥ 495
- **Stop**: 25 pts | **Target**: 3.0R (75 pts) | **Flat**: 720 min (London close)
- **R/Day**: +0.19

### Signal 9: NQ LF1170 (3:30 PM VWAP Fade)
- **Instrument**: NQ
- **Concept**: At 3:30 PM ET (1170 min UTC), if price is > 10 pts from VWAP, fade toward VWAP
- **VWAP**: Computed from RTH ticks **up to minute 1170 only** (no look-ahead)
- **Direction**: Short if price > VWAP+10, Long if price < VWAP-10
- **Stop**: 15 pts | **Target**: 1.5R (22.5 pts) | **Flat**: 1255 min
- **R/Day**: ~+0.20 (clean)

### Signal 10: NQ LF1080 (2:00 PM VWAP Fade)
- **Instrument**: NQ
- **Concept**: Same as LF1170 but at 2:00 PM ET (1080 min UTC)
- **VWAP**: Computed from RTH ticks **up to minute 1080 only** (no look-ahead)
- **Stop**: 20 pts | **Target**: 1.5R (30 pts) | **Flat**: 1255 min
- **R/Day**: ~+0.10 (clean)

### Signal 11: NQ LEXT (London Extreme Fade)
- **Instrument**: NQ
- **Session**: Entry at London close (720 min UTC)
- **Concept**: If London closed in upper 35% of its range → short, lower 35% → long
- **Conditions**: London range ≥ 20 pts
- **Entry**: First tick at minute ≥ 720
- **Stop**: 25 pts | **Target**: 1.0R (25 pts) | **Flat**: 1255 min
- **R/Day**: +0.05

### Signal 12: ES VWAP Multi-Touch (ES_VWPM)
- **Instrument**: ES
- **Session**: RTH, skip first 30 min
- **Concept**: Same as NQ VWPM but on ES with tighter parameters
- **VWAP touch zone**: ±1 pt | **Side establishment**: ±2 pts | **Side reset**: ±4 pts
- **Max trades**: 3 per day
- **Stop**: 3 pts | **Target**: 1.0R (3 pts) | **Flat**: 1255 min
- **Commission**: $0.25/side (ES)
- **R/Day**: +0.66

### Signal 13: ES LF1170 (3:30 PM VWAP Fade)
- **Instrument**: ES
- **Concept**: Same as NQ LF1170 but on ES with tighter parameters
- **VWAP**: Computed from ES RTH ticks **up to minute 1170 only**
- **Threshold**: 3 pts from VWAP (vs 10 pts on NQ)
- **Stop**: 8 pts | **Target**: 1.5R (12 pts) | **Flat**: 1255 min
- **R/Day**: ~+0.20 (clean)

### Signal 14: ES AMD London (ES_AMD)
- **Instrument**: ES
- **Concept**: Same as NQ AMD but on ES
- **Stop**: 10 pts | **Target**: 2.0R (20 pts) | **Flat**: 720 min
- **R/Day**: +0.08

---

## Context Data (Prior-Day Filters)

All context is computed from **completed prior-day data only** (zero look-ahead).

### Leppyrd Bias
- Compare D-1 close to D-2 RTH high/low
- `bias = +1` if D-1 close > D-2 RTH high (bullish)
- `bias = -1` if D-1 close < D-2 RTH low (bearish)
- Sweep-through logic: if D-1 high > D-2 RTH high but close back inside → bearish bias

### PDH Sweep Direction (PDH_SW)
- `pdh_sw = +1` if D-1 close > D-1 RTH high (closed above prior day high)
- `pdh_sw = -1` if D-1 close < D-1 RTH low
- Sweep-through: D-1 high swept D-1 PDH but closed back → `pdh_sw = -1`

### Pre-Market Range
- Max price minus min price for all ticks with minute < RTH_START (870)
- Used as condition filter for Runner (≥100) and SF_NL (≥200)

### Prior Day Range
- D-1 RTH high minus D-1 RTH low
- Used as condition filter for SF_NL (>234)

---

## Sweep-Fail (SF_DIR) Entry Logic

The core entry mechanism. Zero look-ahead by construction.

```
For each completed bar in the session window:
  1. Check PRIOR bar direction (pc.close vs pc.open)
     - If prior bar bullish (close > open): look for SHORT sweep-fail
     - If prior bar bearish (close < open): look for LONG sweep-fail
  2. If direction filter active, skip if direction doesn't match filter
  3. Reference level = prior bar HIGH (for short) or LOW (for long)
  4. Scan CURRENT bar's ticks sequentially:
     a. Wait for price to SWEEP past reference (exceed it)
     b. Track the sweep extreme
     c. If price FAILS back through reference:
        - Check sweep distance > minimum (1.0 pt)
        - ENTRY at the fail-back tick
```

Key: direction is from the **completed prior bar**, not the current bar. Entry is at the real-time tick where the fail occurs.

---

## Trade Execution (trade_f)

```
Given: entry tick index, direction, entry price, stop distance, R-target, flat time
1. Stop = entry_price - direction * stop_distance
2. Scan ticks from entry+1 forward:
   a. If time >= flat: exit at market, compute P&L
   b. If price hits stop: return -1R (minus commission)
   c. If price reaches target: return +target_R (minus commission)
3. Commission: NQ = $0.50/side, ES = $0.25/side
```

---

## Bugs Found & Fixed (8 Total)

| # | Bug | Impact | Script |
|---|-----|--------|--------|
| 1 | Engulfing look-ahead (cc['bd'] used before candle closes) | 77%→40% WR | leppyrd_v4.py |
| 2 | H4 bias filter look-ahead (8-12 UTC candle for 8-10 UTC entries) | Inflated WR | leppyrd_v4.py |
| 3 | AMANIP filter look-ahead (full 0-8 Asia for 4-8 entries) | Inflated WR | leppyrd_v4.py |
| 4 | Sign error in trail stop (short stop-outs counted as wins) | +16.4 R/Day fake | dynamic_trail |
| 5 | Swing confirmation timing (bar START not END time) | +10.4→-0.2 R/Day | keylevel_sweep |
| 6 | FVG entry during forming bar (1 min early) | 65%→40% WR | offset_fvg |
| 7 | MTF 1H bar START time used (45 min look-ahead) | 7→17 LW | mtf_confirm |
| 8 | Late fade uses end-of-day VWAP (30 min look-ahead) | 5→7 LW, -0.25 R/Day | calmar_compare |

---

## Portfolio Comparison (All Clean)

| Config | R/Day | Calmar | Sharpe | Max DD | LW | Months |
|--------|------:|-------:|-------:|-------:|---:|-------:|
| **NQ+ES+Runner** | **+3.34** | **53.1** | **6.57** | **16.4R** | **7** | **13/13** |
| NQ+ES (no runner) | +2.28 | ~42 | ~7.5 | ~14R | ~7 | 13/13 |
| NQ only (no ES) | +1.60 | 27.8 | 6.78 | 14.9R | 5 | 13/13 |
| Runner only | +1.05 | 19.0 | 2.49 | 14.4R | 36 | 9/13 |
| ES only | +0.94 | 29.7 | 6.81 | 8.2R | 8 | 13/13 |

---

## Scripts

| Script | Purpose |
|--------|---------|
| `/tmp/audit_fix_260d.py` | **CANONICAL** — clean portfolio with all fixes |
| `/tmp/calmar_compare_260d.py` | Full portfolio comparison (has bug #8 — use audit_fix instead) |
| `/tmp/final_mega_260d.py` | NQ+ES combo (has bug #8) |
| `/tmp/add_latefade_260d.py` | Late fade discovery (has bug #8) |
| `/tmp/es_signals_260d.py` | ES signal standalone tests |
| `/tmp/es_vwpm_260d.py` | ES VWAP multi-touch tests |
| `/tmp/amd_session_260d.py` | AMD + London breakout + Turtle Soup |
| `/tmp/unicorn_fast_260d.py` | ICT Unicorn (Juno rules) |
| `/tmp/smt_divergence_260d.py` | SMT NQ/ES divergence |
| `/tmp/range_day_hedge_260d.py` | Mean-reversion hedge signals |
| `/tmp/losing_week_deep_260d.py` | Per-day per-leg losing week analysis |
| `/tmp/regime_skip_portfolio_260d.py` | Pre-market regime skip filter |
| `/tmp/runner_variants_v2_260d.py` | Runner stop/management variants |
| `/tmp/more_sf_legs_260d.py` | SF across all sessions and timeframes |

---

## Dead Ends (Tested, Don't Work)

- Weekly stop-loss (misses recovery days)
- MTF 1H confirmation with clean bars (hurts R/Day)
- Counter-trend SF (too weak)
- Stacked signal confluence (adds LW)
- Regime skip (identifies range days but doesn't reduce LW)
- Adaptive sizing (can't flip -10R weeks)
- ICT Unicorn (works but correlated, +0.11 R/Day)
- Kane TLM (too few trades with proper iFVG requirement)
- Turtle Soup 1H (negative)
- All V10a-V10n alignment methods (look-ahead bias)

---

## ES-Only Portfolio (6 Legs, March 22)

Standalone ES portfolio for trading without NQ exposure.

| Metric | Value |
|--------|-------|
| **R/Day** | **+1.01** |
| **Calmar Ratio** | **14.1** |
| **Legs** | **6** |

| # | Leg | Trades | WR | AvgR | R/Day | Notes |
|---|-----|-------:|---:|-----:|------:|-------|
| 1 | ES_VWPM | — | 66% | — | +0.66 | VWAP multi-touch, ±1pt zone, ±2pt side, ±4pt reset, 3pt stop |
| 2 | ES_LF1170 | — | — | — | +0.25 | 3:30 PM VWAP fade, 3pt threshold, 8pt stop |
| 3 | ES_AMD | — | — | — | +0.08 | London fade first 15min, 10pt stop |
| 4-6 | (remaining ES legs) | — | — | — | +0.02 | Minor contributors |

**ES Runner: DEAD** — runner concept does not transfer to ES (insufficient volatility for BE5 management).
**ES SF_PRE: DEAD** — pre-market sweep-fail on ES has negative expectancy.

---

## Cross-Correlation Matrix (March 22)

Signal correlations across the 15-leg NQ+ES portfolio. Low cross-correlation confirms genuine diversification.

| | Runner | SF_PRE | SF_NY1 | VWPM | ES_VWPM | LF1170 | AMD | LEXT |
|--------|:------:|:------:|:------:|:----:|:-------:|:------:|:---:|:----:|
| Runner | 1.00 | 0.02 | -0.04 | 0.08 | 0.03 | -0.06 | 0.01 | -0.14 |
| SF_PRE | | 1.00 | 0.11 | -0.03 | 0.01 | 0.04 | -0.02 | 0.03 |
| SF_NY1 | | | 1.00 | 0.06 | -0.01 | 0.02 | 0.05 | -0.03 |
| VWPM | | | | 1.00 | 0.12 | 0.09 | -0.05 | 0.01 |
| ES_VWPM | | | | | 1.00 | 0.07 | 0.02 | -0.08 |
| LF1170 | | | | | | 1.00 | -0.03 | 0.06 |
| AMD | | | | | | | 1.00 | -0.07 |
| LEXT | | | | | | | | 1.00 |

Key findings:
- **NQ-ES cross-instrument correlation is 0.03-0.12** — genuine decorrelation
- **LEXT has -0.14 correlation with Runner** — natural hedge during losing weeks
- **SF legs are 0.02-0.11 correlated with each other** — session separation provides independence
- **VWPM and ES_VWPM at 0.12** — highest same-concept correlation, still low

---

## Updated Portfolio: 15 Legs with PDH_PRE (March 22)

### Signal 15: NQ PDH_PRE (PDH Sweep Pre-Market)
- **Instrument**: NQ
- **Session**: Pre-NY (720-870 min UTC)
- **Timeframe**: 15-minute bars
- **Concept**: Sweep-fail on prior day high/low level during pre-market
- **Direction filter**: PDH_SW (prior day high sweep direction)
- **Stop**: 30 pts | **Target**: 1.5R (45 pts) | **Flat**: 1255 min
- **R/Day**: ~+0.10

### Updated Performance Summary (15 Legs)

| Metric | 14 Legs | 15 Legs (with PDH_PRE) |
|--------|:-------:|:----------------------:|
| **R/Day** | +3.34 | **+3.44** |
| **Calmar** | 53.1 | **~55** |
| **Legs** | 14 | 15 |
| **Losing Weeks** | 7 | 7 |
| **Winning Months** | 13/13 | 13/13 |

---

## Dead Legs (Tested, Confirmed Non-Viable)

- **ES Runner**: Insufficient ES volatility for BE5 runner management. Negative expectancy.
- **ES SF_PRE**: Pre-market sweep-fail on ES does not produce edge. Negative avgR across all parameter combos.

---

## Audit Notes (March 21)

**No remaining look-ahead bias.** All 8 bugs fixed and verified.

**Known limitations:**
1. **VWAP fill assumption**: VWPM/VWT/ES_VWPM enter at exact VWAP price when tick is within ±2 (NQ) or ±1 (ES). In reality, limit orders at VWAP may not fill if price doesn't reach exact level. ~5-10% of VWAP trades may not execute live.
2. **No slippage**: All entries/exits at exact price. 1-2 tick slippage on NQ (0.25-0.50 pts) and ES (0.25 pts) not modeled. ES_VWPM with 3pt stop is most affected (8% of risk per trade).
3. **No walk-forward on combination**: Individual signals validated on 260d, but the 14-signal portfolio combination was only tested in-sample. Calmar 53.1 would likely degrade to ~30-40 out-of-sample.
4. **Commission**: NQ 0.50 pts round-trip, ES 0.25 pts round-trip. Correct for typical retail futures rates.
5. **Multiple simultaneous positions**: Not tracked. Up to 5+ NQ and 3+ ES positions could be open at the same time. Capital/margin requirements not modeled.
