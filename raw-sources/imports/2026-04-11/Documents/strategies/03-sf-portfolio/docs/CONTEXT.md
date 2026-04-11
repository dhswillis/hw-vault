# Tempo Trading System — Session Context

## CRITICAL ISSUE LOG

### 2026-03-18: Look-Ahead Bias in SF Engulfing Backtester
**ALL** SF portfolio backtest results from `leppyrd_v4.py` are INVALID. The `sf_scan_window()` function uses the current candle's final close (body direction + size) to check engulfing patterns, but entries happen during that candle before it closes. This is look-ahead bias.

- Buggy (260d): 77% WR, +18,328 pts, 0 losing weeks → **FAKE**
- Fixed (260d, same V21 config): 40% WR, -1,092 pts, 27 losing weeks → V21's specific config is net negative

V21/V22/V23 NinjaTrader strategies are all based on parameters optimized against the biased results. DO NOT TRADE those configs.

### 2026-03-18: Premature "No Edge" Assessment (corrected same session)
After finding the look-ahead bias, the initial analysis incorrectly concluded that the sweep-fail concept itself has no edge. This was wrong — only the V21-specific configuration (with the buggy engulfing check) is net negative. A grid search of 5,870 bias-free configurations found 2,215 profitable configs.

The sweep-fail concept DOES work, particularly with:
- **DIR scan** (prior completed bar direction check only, no engulfing on developing bar)
- **PRE window** (12:00-14:30 UTC) — strongest and most consistent
- **BIAS and PDH_SW filters** — improve filtered legs significantly

### 2026-03-18: Filter Look-Ahead in Early Sessions (found during third audit)
AMANIP filter uses today's full Asia range (0-480 min) — look-ahead for any window within Asia (0-480). BIAS_H4 filter uses today's H4 candles (0-720) — look-ahead for any window before 720. Fixed by adding `FILTER_MIN_START` safety map that blocks contaminated filter+window combos.

### 2026-03-18: 13-Window Portfolio Was Overfit (260-day validation failed)
The 30-day 13-leg portfolio (+1,425 pts, 0 LW) was curve-fit. On 260 days: last 30d = +1,425, first 230d = **-565**. Do not use those configs.

### 2026-03-18: NinjaTrader Engulfing ≠ Python Engulfing
V21's NinjaTrader uses the CURRENT TICK as `currCandle.C` (not the final close). This is NOT look-ahead, but it's extremely strict and barely triggers on most eng_req=True legs. The NT-faithful engulfing produces very few trades compared to the buggy Python.

### Current Status: 260-Day Validated Results (HONEST)
Four scan variants tested on V21's exact 10 legs across full 260 days. No full-portfolio edge exists. Individual legs with genuine signal:

- **PRE_AMANIP** (RAW scan, 202 trades, 55% WR, PF 1.21, +557.5 pts) — **only convincing edge**
- NY1_15M (DIR scan, 210 trades, 37% WR, PF 1.11, +425.5 pts) — marginal, asymmetric R:R
- ALATE_SF (DIR scan, 257 trades, 38% WR, PF 1.13, +211.5 pts) — marginal

The AMANIP filter (Leppyrd bias + Asia manipulation confirmation) is the real signal from Leppyrd's work. Raw sweep-fail without directional filters is noise on 260 days.

See: `docs/BACKTEST_ISSUES_LOG.md` for complete audit trail.

---

## Owner
Harrison (@prop_profitable) — trader, not developer. Building autonomous NQ futures backtesting system.

## Core Strategy: Portfolio of Non-Correlated Signal Models

Tempo is NOT a single strategy. It's a **portfolio approach** — multiple independent signal models that generate non-correlated trade populations. Each model has its own entry logic, execution assumptions, and edge profile. The goal is **10R+ per day combined** by blending models that win in different conditions.

### Why Non-Correlation Matters
- Model A might crush trending days but lose in ranges
- Model B might fade ranges but get chopped in trends
- Combined, they smooth equity curves and compound edges
- If models used the same triggers, losses would cluster on the same days

### The Models

#### Model 1: QC Sweep + IFVG (QuantConnect)
- **Entry**: Liquidity sweep at key level (PDH, PDL, London H/L, Asia H/L, AM H/L) → IFVG confirmation (price closes through FVG boundary)
- **Fill**: Market order after IFVG confirms. 0.25pt slippage per side.
- **Timeframes**: Multi-TF scanning (15s, 30s, 1m, 2m, 3m, 5m, 15m)
- **Stops**: Dynamic — gap-based, then swing-based, then ATR fallback. Clamped 5-30pt.
- **Targets**: Liquidity-based (next key level). Multi-exit: 55% at TP1, 30% at TP2, runner on TP3.
- **Filters**: Quality rating (SMT, bias, gap size), news blackout, daily loss limit (2 max), session gating, inside/outside day
- **Commission**: $4.50 RT/contract
- **Code**: `batch-runner/main_v4.py` (QC), `batch-runner/main.py` (V3.34)
- **Status**: Running on VPS via QuantConnect. Batch 3 in progress.

#### Model 2: Context Engine Signals (Local Backtester)
- **Entry**: Multiple signal types with independent entry logic:
  - **BOS_FVG**: Break of structure + FVG fill at midpoint via limit order
  - **sweep_reversal**: Liquidity sweep + swing rejection at close
  - **VA_fade**: Value area boundary fade
  - **pre_gap_fill**: 5M displacement → 1M FVG fill entry
- **Fill**: Limit order at FVG midpoint (BOS_FVG, PGF) or candle close (sweep, VA). 0pt entry slippage on limits, 0.5pt exit slippage.
- **Timeframes**: 5M structure + 1M entry
- **Stops**: FVG boundary ± 2pt buffer
- **Targets**: Next 5M swing level or 2x displacement, capped at target_cap
- **Filters**: Confluence scoring (min 2), session timing, volume confirmation, bias alignment
- **Commission**: $5 RT ($2.50/side)
- **Code**: `context-engine/micro_structure.py`, `context-engine/intraday_scanner.py`
- **Status**: V7d validated. +3.98R/day full year (258 days). See results below.

#### Why These Models Are Different (Not a Bug)
| Dimension | QC Model | Backtester Model |
|---|---|---|
| Entry trigger | IFVG (close through FVG boundary) | FVG midpoint limit fill |
| Sweep requirement | Required (at specific key levels) | Not required (BOS is different pattern) |
| Fill assumption | Market order + slippage | Limit order, no entry slippage |
| Multi-timeframe | 7 timeframes simultaneously | 2 timeframes (5M + 1M) |
| Target model | Liquidity levels + multi-exit | Single swing target |
| These generate **different trade populations** — that's the point. |

## Execution Modeling

### QC Model (main_v4.py)
- Slippage: 0.25 pts (1 tick) per side — market orders
- Commission: $4.50 round-trip per contract
- Tick-through: Price must trade 1 tick past FVG entry for fill
- BE offset: 0.50 pts above entry to cover exit costs

### Backtester Model (micro_structure.py)
- Entry slippage: 0pt — limit orders at known levels (FVG midpoint)
- Exit slippage: 0.5pt (2 ticks NQ) — stops/targets/timeouts are market orders
- Commission: $5 RT ($2.50/side = 1.0pt NQ)
- Tick-through: Price must penetrate FVG midpoint by 0.25pt for fill confirmation
- Risk per trade: Fixed 1R regardless of stop distance (position sizing handles it)

### On Slippage and Gap Size
- Large FVG gaps do NOT mean more risk — position sizing normalizes everything to 1R
- A 30pt stop and a 5pt stop risk the same dollar amount with correct sizing
- Gap size filtering is a signal quality question, not a risk question

## Key Metric: Safe Rate
`safe_rate = (wins + BE_saves) / total_trades` — the "don't lose money" rate.

## Backtester Results (V7f — Full Year Validated, Optimized)

### Best Config: tc200 rr2.0 no-kill — +11.31R/day
- **Signals**: BOS_FVG, VA_fade, sweep_reversal, pre_gap_fill (9 losing signals disabled)
- **Parameters**: target_cap=200, min_rr=2.0, no daily kill switch, no B/E, max_hold=60
- 258 days, 4,991 trades, 26.5% WR, +2,918R total, **+11.31R/day**
- **72% profitable days**, 13/13 months positive, max drawdown 47R
- Median daily R: +8.7, avg 19.3 trades/day, 5.1 wins/day
- BOS_FVG: 7/day, sweep_reversal: 4.6/day, VA_fade: 7.6/day

### Conservative Config: conf≥3 kill-20R — +9.53R/day
- Same signals but confluence≥3 filter, -20R daily kill switch
- 256 days, 3,343 trades, 27.4% WR, +2,440R total
- **72% profitable days**, 13/13 months positive, max drawdown 29R (best risk-adjusted)
- 13.1 trades/day, 3.6 wins/day

### Key Optimization Findings (V7e/V7f)
- **Target cap is the biggest lever**: tc40→tc200 nearly doubles R/day (3.98→6.85)
- **Kill switch was choking off trades**: -5R kill prevented late-day recovery. Widening to -20R or removing it lets the system trade through drawdowns.
- **Daily kill at -5R was cutting BOS_FVG trades** by 40% (659 trades vs 1204 with kill off)
- **min_rr=2.0 filters marginal setups**, improving R/day without losing much volume
- **Days with 11+ trades = 74% profitable** at +10R/day avg
- **Days with 21+ trades = 78% profitable** at +19.7R/day avg

### Signal R Contribution (all at flat 1R)
| Signal | R/year | R/trade | WR | Trades/day |
|---|---|---|---|---|
| BOS_FVG | +714 | +0.593 | 33.9% | ~7 |
| sweep_reversal | +207 | +0.193 | 37.7% | ~4.6 |
| VA_fade | +89 | +0.062 | 24.4% | ~7.6 |
| pre_gap_fill | +17 | +0.242 | 22.2% | ~0.3 |

### Win Condition Analysis (V7d)
Best filters by R/trade:
- **Volume spike**: +1.021 R/trade, 41.8% WR, 13/13 months positive
- **BOS_FVG in silver bullet + london open**: +0.878 R/trade, 38.4% WR, 13/13 months
- **Engulfing**: +0.690 R/trade, 34.8% WR
- **Conf≥5 + volume**: +0.614 R/trade, lowest drawdown (11.1R)

### Management Tests (Full Year)
- **B/E at 1R**: -490R — DESTROYS the edge
- **Partials 1.5R runner 7R**: -43R — breakeven
- **Conclusion**: No trade management. Let winners run, take the losses.

### Walk-Forward Validation (V7g — PASSED)
Best config (tc200, rr2.0, no-kill) tested across all time splits. **ALL periods positive.**
- First half: +12.58R/day, 74% prof | Second half: +10.04R/day, 69% prof
- Q1 (Feb-Apr): +19.21R/day, 82% prof | Q2 (May-Jul): +7.66R/day, 67% prof
- Q3 (Aug-Oct): +7.01R/day, 62% prof | Q4 (Nov-Jan): +12.58R/day, 77% prof
- **0/13 months negative**. Worst month (Jun): +4.16R/day. Best month (Feb 25): +22.61R/day.
- Quarterly R/day: avg=+11.62, std=4.87, CoV=42%
- **Edge is real and robust across all out-of-sample periods.**

## Batch 2 Results (QC, pre-cost-modeling)
- 2000 variants generated, 842/2000 completed Stage A (Jul-Sep 2025)
- 290 winners found (positive net return)
- Top performer: v4_sig_0142 at 106.2% / $53K
- Results in `Tempo_Batch2_Winners.xlsx`

## Architecture
- **QuantConnect (QC)**: Cloud backtesting engine for Model 1
- **Local backtester**: `context-engine/` for Model 2
- **VPS**: QuantVPS Pro — runs batch jobs overnight
- **GitHub**: `dhswillis/trading-system` — source of truth
- **Data**: Databento tick data, 312 files (Feb 2025 - Feb 2026)

## Inverse Trade Theory (V7k/V7l/V7m — Directional Accuracy + Regime Hedging)

Harrison's insight: "if you have a 1:1RR trade with low probability, the opposite trade should have the inverse winning percentage."

### V7k Results: True 1:1 RR Win Rate
**KEY FINDING: Entries are 57.7% directionally accurate at 1R** — much better than the 26.5% headline WR at 2R+ target. Of 4991 trades, 2881 (57.7%) touch +1R before -1R. The remaining 2066 hit -1R first, and 44 time out.

- **BOS_FVG**: 62.6% touch +1R (inverse only 37.4%)
- **sweep_reversal**: 50.4% touch +1R (inverse 49.6% — coin flip)
- **VA_fade**: 57.0% touch +1R (inverse 43.0%)
- **pre_gap_fill**: 76.1% touch +1R (inverse 23.9%)

**The inverse theory doesn't work at the aggregate level** (42.3% < 50%), but conditional pockets exist.

### V7k-deep: Conditional Inverse Mining
Mining all dimensional combinations found:
- **24 combos above 62.5%** (profitable after costs at 1:1)
- **169 combos above 55%**
- **205 combos above 50%** (reverse martingale viable)

Best triple combos: sweep×sbam×frozen (87.5%), VA×overnight×consec3 (79.2%), sweep×sbam×prevloss (77.3%)

The `consec_inv_wins` dimension is a **regime detector**: after 3+ consecutive original-direction misses, inverse WR jumps to 52.8%. At streak ≥ 5, VA_fade inverse WR hits **71.9%**.

### V7l: Walk-Forward Validation
11 out of 17 conditions pass all 3 validation tests (60/40 split, rolling 3-window, monthly consistency):
- **ROBUST**: sweep×sbam×frozen, VA×overnight×consec3, sweep×Wed×consec2, VA×overnight×hot, sweep×lnyo×warm, sweep×cold×consec2, BOS×nyopen×Mon, sweep×lnyo, any×consec3, sweep×consec2, sweep×prevsess_red

### V7m: Regime Switch Strategy
Best config: **VA+sweep regime switch (VA=3, sweep=3, exit=2)**
- R/day: +10.09 (vs baseline +11.31) — lower R, but MaxDD: 35R (vs 47R)
- **Calmar: 0.607** (vs baseline 0.467) — **30% improvement**

**Best approach: HEDGE (additive overlay)**
Keep ALL original trades + add inverse trades when regime shifts:
- Config: VA inverse after 4 misses, sweep inverse after 3 misses
- **R/day: +11.42, MaxDD: 34R, Calmar: 0.514** — more R AND less drawdown
- Walk-forward confirmed: test period Calmar holds

**RULES:**
1. **NEVER invert BOS_FVG** — 32-36% inverse WR regardless of streak
2. VA_fade is the best inverse candidate (72% at streak ≥ 5)
3. sweep_reversal is moderate (60% at streak ≥ 5)
4. Inverse trades are HEDGES, not replacements
5. Track streaks per signal separately

---

## Model 3: NinjaTrader SF Portfolio (V14–V17) — CURRENT PRODUCTION TARGET

### Overview

The NinjaTrader line (V14–V17) is a completely separate strategy family from Models 1 and 2. It runs on NinjaTrader 8, uses a 1-minute chart with Calculate.OnEachTick, and targets live trading with managed orders. All backtesting is done in Python against Databento tick data (260 trading days, Feb 2025 – Feb 2026), then ported to NinjaScript C#.

### Platform Constraints (Hard Rules)
- NO SetStopLoss/SetProfitTarget — causes immediate trigger if stop price is past live price
- Must work ONLY on 1-minute chart (Calculate.OnEachTick for tick-level entry detection)
- Must work out of the box for live trading — no manual setup
- Cannot leave open orders without a stop
- All order management is manual via EnterLong/ExitLong with virtual fill tracking

### Core Entry Mechanic: Sweep & Fail (SF)

All V17 legs use the same tick-level entry engine:

1. Build candles at the leg's timeframe (5M or 15M) within the leg's time window
2. Price sweeps beyond prior candle's high or low (tick-level detection, no look-ahead)
3. Price fails back through the prior candle's boundary — this exact tick is the entry
4. Optional engulfing check: current candle body > prior candle body, closing in trade direction
5. Minimum sweep size filter (0 to 1.0 pts, leg-dependent)
6. Trade direction must agree with Leppyrd daily bias

Two SF variants:
- **SF Engulfing**: Requires engulfing body confirmation (higher WR, fewer trades)
- **Raw SF**: No engulfing requirement (more trades, slightly lower WR per trade but still profitable)

### Directional Filters

**Leppyrd Daily Bias** (all legs): Yesterday's close vs day-before-yesterday's RTH high/low.
- Close above RTH high → bullish bias
- Close below RTH low → bearish bias
- Close inside → no trade (flat)

**PDH Sweep Direction** (PRE_PDH leg): Yesterday's full-day range vs yesterday's RTH range.
- If yesterday's high exceeded RTH high but closed back inside → bearish signal
- If yesterday's low exceeded RTH low but closed back inside → bullish signal

**Asian Manipulation (AMANIP)** (PRE_AMANIP leg): Today's Asia session (0:00-8:00 UTC) moved AGAINST the Leppyrd bias direction, relative to yesterday's RTH midpoint. Computed after Asia close at 8:00 UTC.
- Bullish bias + Asia high above yesterday's RTH midpoint → manipulation confirmed
- Bearish bias + Asia low below yesterday's RTH midpoint → manipulation confirmed

**BIAS_H4 Stacked** (NY1_BH4 leg): Both Leppyrd bias AND overnight 4H swing direction must agree with trade direction. 4H swing = current 4H candle sweeps prior 4H candle's H/L and closes back inside (3 candle slots: 0-4h, 4-8h, 8-12h UTC).

### V17 Portfolio — 10 Legs

| # | Leg | TF | Window (UTC min) | Flat | Eng | MinSw | Filter | T/S |
|---|---|---|---|---|---|---|---|---|
| 0 | LON_SF | 15M | 480-600 | 630 | Yes | 0.5 | None | 15/8 |
| 1 | PRE_PDH | 15M | 720-870 | 900 | Yes | 1.0 | PDH_SW | 30/30 |
| 2 | NY1_15M | 15M | 870-930 | 990 | Yes | 1.0 | None | 60/30 |
| 3 | NY1_5M | 5M | 870-930 | 990 | Yes | 0.5 | None | 30/30 |
| 4 | NLATE_5M | 5M | 990-1110 | 1260 | Yes | 1.0 | None | 30/30 |
| 5 | AEARLY_SF | 15M | 0-240 | 270 | Yes | 0.0 | None | 30/15 |
| 6 | ALATE_SF | 15M | 240-480 | 510 | No | 0.0 | None | 20/10 |
| 7 | PRE_AMANIP | 15M | 720-870 | 900 | No | 0.5 | AMANIP | 30/30 |
| 8 | NLATE_15M | 15M | 990-1110 | 1260 | No | 0.5 | None | 30/30 |
| 9 | NY1_BH4 | 15M | 870-960 | 990 | Yes | 0.5 | BIAS+H4 | 60/30 |

Session windows in UTC: ASIA=0:00-8:00, LON=8:00-10:00, PRE=12:00-14:30, NY1=14:30-15:30, NLATE=16:30-18:30. RTH=14:30-21:00.

### V17 Backtest Results (260 days, tick-level, 0.25 slippage + 0.50 commission)

| Leg | Trades | WR | PF | Total Pts | Pts/Day | R/Day | LW |
|---|---|---|---|---|---|---|---|
| LON_SF | 68 | 84% | 8.84 | +733 | +2.82 | +0.35 | 2 |
| PRE_PDH | 48 | 92% | 10.64 | +1,176 | +4.52 | +0.15 | 0 |
| NY1_15M | 43 | 84% | 10.50 | +1,807 | +6.95 | +0.23 | 1 |
| NY1_5M | 139 | 88% | 7.61 | +3,110 | +11.96 | +0.40 | 4 |
| NLATE_5M | 75 | 72% | 2.79 | +996 | +3.83 | +0.13 | 18 |
| AEARLY_SF | 145 | 72% | 4.76 | +2,211 | +8.50 | +0.57 | 6 |
| ALATE_SF | 244 | 60% | 2.80 | +1,832 | +7.05 | +0.71 | 10 |
| PRE_AMANIP | 121 | 85% | 5.53 | +2,490 | +9.57 | +0.32 | 7 |
| NLATE_15M | 171 | 85% | 5.78 | +3,506 | +13.48 | +0.45 | 2 |
| NY1_BH4 | 12 | 92% | 16.34 | +468 | +1.80 | +0.06 | 0 |

**Combined Portfolio:**
- Total: 1,066 trades (4.1/day), 77.4% WR
- +18,328 pts total, **+70.5 pts/day, +3.36 R/day, +874 R/year**
- 100% winning weeks (54/54), 100% winning months (13/13)
- Worst week: +78 pts, worst month: +682 pts
- 90.8% winning days (236/260)

**Correlation:** Avg pairwise daily PnL correlation = 0.033 (essentially uncorrelated). Highest pair: NY1_15M/NY1_BH4 at 0.41 (expected — same window, BH4 is filtered subset). Session groups (ASIA, PRE, LON, NY1, NLATE) all under 0.15 correlation.

### Development History

**V14 (Feb 2026)**: First NinjaTrader port. 5-leg portfolio using Leppyrd bias + SF entries. Basic version, not audited.

**V15 (Feb 2026)**: Added Raw SF variant (no engulfing requirement). Found it nearly doubles trade count while keeping edge.

**V16 (Feb 2026)**: 9-leg portfolio. Two CRITICAL bugs found during audit:
1. **Leppyrd bias self-reference**: Compared same-day values instead of day-before-yesterday. The bias was using today's RTH H/L instead of the prior day's. Fixed by shifting reference levels at session start.
2. **Breaker entry look-ahead**: Entered at candle start using conditions only known at candle close. Fixed by implementing tick-level entry detection (sweep→fail→enter on the exact fail tick).

Both bugs were found via line-by-line audit against the Python backtester (leppyrd_v2.py). Fixed in both Python and C#.

**V17 (Mar 2026)**: 10-leg portfolio. Added NY1_BH4 leg. Four additional bugs found during audit against leppyrd_v4.py:
1. **PDH/PDL were full-day, not RTH-only**: C# used `dailyHigh` (24h session). Python filters to RTH hours (14:30-21:00 UTC). Fixed by adding `rthHigh`/`rthLow` tracking that only updates during RTH.
2. **Bias and PDH_SW used same reference levels but should use different days**: Python bias uses day-before-yesterday's RTH H/L, PDH_SW uses yesterday's RTH H/L. C# used same pdh/pdl for both. Fixed with separate variables and RTH shift logic in ResetDay.
3. **AMANIP used yesterday's Asia data at session start**: Should use TODAY's Asia H/L (accumulated from current day's ticks) vs yesterday's RTH midpoint. Fixed by deferring AMANIP computation to 8:00 UTC.
4. **H4 swing missing 8-12h candle**: Only tracked 2 of 3 H4 candles (0-4h, 4-8h). Fixed with generic h4Prev/h4Curr pattern covering all 3 slots.

### Key Files

- `V17SFFilteredPortfolio.cs` — 1,119 lines, fully audited NinjaTrader strategy (PRODUCTION)
- `leppyrd_v4.py` — Python backtester, SF + filter approach, 595 lines (REFERENCE)
- `leppyrd_v2.py` — Earlier Python backtester, SF + Leppyrd bias (REFERENCE)
- `v4_results.txt` — V4 parameter sweep: 2,036 configs, 56 zero-LW configs
- `v5_results.txt` — V5 parameter sweep: 221 configs, 8 zero-LW configs
- `v17_portfolio_stats.py` — Combined portfolio stats script
- `v17_corr.py` — Correlation analysis script

### Mining Methodology (V4/V5)

V4 tested 2,036 configs across: 7 session windows × 2 timeframes × 3 sweep thresholds × 2 engulfing modes × 6 filter types × various T/S combos. Selected legs that maintain 0 losing weeks when added to portfolio. V5 extended with finer parameter grid, found 8 additional zero-LW configs including NY1_5M.

### Potential Addition: BOS_FVG (Under Evaluation)

BOS_FVG (Break of Structure + Fair Value Gap) is a separate strategy using 1M candles:
- BOS fires when price closes through a confirmed 3-bar swing point
- FVG forms within 15 bars of BOS in same direction
- Entry when price fills back to FVG edge
- Pure trailing stop (trigger 0.5R, buffer 0.1R), session-end kill at 20:00 UTC
- Backtest: 5,090 trades over 258 days, 47.8% WR, +30.5 R/day, 0 losing weeks
- Code: `clean_backtest.py` (Python), `BOSFVGStrategy.java` (MotiveWave)
- Needs: NinjaTrader port, audit, correlation test against V17 legs

### Potential Addition: SMT Divergence (Under Evaluation)

Smart Money Technique — NQ vs ES swing structure divergence as directional filter.
- Bearish SMT: NQ makes new high, ES doesn't confirm
- Bullish SMT: NQ makes new low, ES holds higher
- Code exists: `context-engine/features/smt.py` (daily SPY proxy only)
- Needs: ES tick data from Databento (downloading, free "last year" tier, ~11 months overlap with NQ)
- Would test as: additional directional filter on existing SF legs, or standalone signal

---

## Known Issues / TODO
- ~~Inverse signal logic is broken~~ — **RESOLVED (V7k/V7l/V7m): inverse works as hedge, not as replacement strategy**
- ~~Walk-forward / out-of-sample validation not done yet~~ — **DONE (V7g): ALL periods positive**
- ~~V17 audit~~ — **DONE: 4 bugs found and fixed, brace balance verified (139/139)**
- QC has 64K file size limit — main_v4.py compressed to 47K
- Vol regime uses static VIX=18.5 in backtest — needs live feed
- Port BOS_FVG to NinjaTrader and audit against clean_backtest.py
- Download ES Databento tick data and test SMT divergence as filter/signal
- NLATE_5M is weakest leg (18 LW, 72% WR, PF 2.79) — consider dropping or replacing
- Run V17 live paper trading to validate tick-level fills match backtest
