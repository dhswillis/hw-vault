# Willis Holdings NQ Playbook
## V10 Research Series — Clean Results (Post Look-Ahead Audit)

**Data**: 312 trading days of NQ futures tick data (Databento)
**Instrument**: Micro E-mini Nasdaq-100 (MNQ) / E-mini Nasdaq-100 (NQ)
**Risk Unit**: 1R = risk per trade (distance from entry to stop loss, in points, normalized)
**Slippage**: 0.5 pts per exit, $2.50 commission per side (conservative)

> **IMPORTANT**: V10o discovered that multi-TF alignment used forming-bar closes (look-ahead).
> ALL V10a-V10n alignment-filtered results were invalidated. V10r-V10t re-tested everything
> with clean data. The results below are **post-audit, walk-forward validated**.

---

## Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [15S Trailing — The Breakthrough (V10y)](#2-15s-trailing--the-breakthrough-v10y)
3. [Hybrid Portfolio (FLAGSHIP)](#3-hybrid-portfolio-flagship)
4. [BOS_FVG Deep Dive](#4-bos_fvg-deep-dive)
5. [Core Concepts](#5-core-concepts)
6. [All 6 Production Signals](#6-all-6-production-signals)
7. [1M Trailing Portfolio (V10v Baseline)](#7-1m-trailing-portfolio-v10v-baseline)
8. [Fixed R Portfolio (V10t Baseline)](#8-fixed-r-portfolio-v10t-baseline)
9. [Walk-Forward Results](#9-walk-forward-results)
10. [Dead Signals (6)](#10-dead-signals)
11. [Dead Ends — Comprehensive](#11-dead-ends)
12. [Look-Ahead Audit (V10o)](#12-look-ahead-audit)
13. [Risk & Realism](#13-risk--realism)
14. [Glossary](#14-glossary)

---

## 1. Executive Summary

> **V10z AUDIT**: Conservative intra-bar simulation (assumes worst-case OHLC ordering).
> engulfing_key_level, VA_fade, and pre_gap_fill are **all dead** when the intra-bar
> ordering bug is fixed. Their trailing/B/E edge was the bug. Only 3 signals survive.

**V10y proved 15-second trailing crushes 1-minute.** V10z confirmed it survives conservative
simulation, slippage stress testing, and worst-case exit fills. BOS_FVG is the engine.

| Metric | V10z Conservative 3-Signal | BOS_FVG 15S Only |
|--------|---------------------------|------------------|
| Signals | 3 | 1 |
| Trades (312 days) | 887 | 685 |
| Avg R/Trade | **+1.008** | **+1.141** |
| Total R | **+894.0** | **+781.8** |
| **R/Day** | **+2.87** | **+2.51** |
| Max Drawdown | -9.4R | **-3.0R** |
| **Calmar Ratio** | **94.4** | **263.6** |
| WF OOS avgR | **+0.996** | **+1.149** |
| Quarterly OOS+ | **4/4** | **4/4** |
| Every month positive | Yes (13/13) | Yes (13/13) |

### What "trig=0.5R trail=0.1R" means (median BOS_FVG risk = 5.9 pts)

| Parameter | In R | In NQ Points | In Ticks (0.25pt) |
|-----------|------|-------------|-------------------|
| Trigger | 0.5R | ~3 pts | ~12 ticks |
| Trail (0.1R) | 0.1R | ~0.6 pts | ~2 ticks |
| Trail (0.25R) | 0.25R | ~1.5 pts | ~6 ticks |
| Trail (0.5R) | 0.5R | ~3 pts | ~12 ticks |

**0.1R trail = 2 ticks.** Very tight. Recommended production: **0.25R trail (6 ticks)** for execution safety — still Calmar 225.9.

### V10z Slippage Stress Test (BOS_FVG 15S trig=0.5R trail=0.1R)

| Entry Slip | Exit Slip | avgR | Calmar |
|-----------|-----------|------|--------|
| 0.25pt | 0.50pt | +1.141 | 263.6 |
| 0.50pt | 0.50pt | +1.096 | 236.8 |
| 1.00pt | 0.50pt | +1.004 | **147.4** |
| 0.25pt | 1.00pt | +1.059 | 214.7 |
| 0.25pt | **2.00pt** | +0.893 | **145.6** |

**Survives 2 FULL POINTS of exit slippage** (Calmar 145.6). Tank.

---

## 2. 15S Trailing — The Breakthrough (V10y)

V10y tested managing trailing stops on 15-second bars instead of 1-minute bars. The thesis: **1M bars are too coarse — price can run +3R and reverse to +1R within a single 1M candle, and the trail never saw the high.** 15S gives 4x more granular updates, catching fat tail peaks before reversals.

### BOS_FVG: 1M vs 15S Head-to-Head (685 trades)

| Config | 1M avgR | 15S avgR | 1M Calmar | 15S Calmar | Improvement |
|--------|---------|---------|-----------|-----------|-------------|
| trig=0.5R trail=0.25R | +0.748 | **+1.054** | 52.2 | **231.6** | **+41%** |
| trig=0.5R trail=0.5R | +0.666 | **+0.921** | 45.9 | **187.5** | **+38%** |
| trig=1.0R trail=0.25R | +0.769 | **+1.017** | 57.6 | **112.7** | **+32%** |
| trig=1.0R trail=0.5R | +0.690 | **+0.895** | 49.8 | **95.3** | **+30%** |
| trig=1.0R trail=0.75R | +0.607 | **+0.773** | 41.3 | **79.3** | **+27%** |
| trig=1.0R trail=1.0R | +0.531 | **+0.657** | 30.7 | **64.9** | **+24%** |

**15S wins on every config.** Tighter trails benefit more because they need granularity to not get shaken out.

### Best BOS_FVG 15S Configs (Tight Trails)

| Config | WR | avgR | Total R | DD | Calmar | R/Day | WF OOS |
|--------|-----|------|---------|-----|--------|-------|--------|
| **trig=0.5R trail=0.1R** | **75.8%** | **+1.146** | **+785.1** | **-3.0** | **264.7** | **+2.52** | **+1.158** |
| trig=0.5R trail=0.15R | 75.8% | +1.111 | +761.1 | -3.0 | 252.3 | +2.44 | +1.125 |
| trig=0.5R trail=0.2R | 75.8% | +1.078 | +738.2 | -3.1 | 240.8 | +2.37 | +1.091 |
| trig=0.5R trail=0.25R | 75.8% | +1.054 | +721.7 | -3.1 | 231.6 | +2.31 | +1.066 |
| trig=0.75R trail=0.15R | 72.7% | +1.097 | +751.2 | -3.7 | 201.9 | +2.41 | +1.117 |
| trig=0.75R trail=0.25R | 72.7% | +1.045 | +715.8 | -3.8 | 187.3 | +2.29 | +1.060 |
| trig=1.0R trail=0.25R | 67.9% | +1.017 | +696.8 | -6.2 | 112.7 | +2.23 | +1.030 |

**All 14 configs pass walk-forward with 4/4 Q+. OOS ≥ IS on most.** This is not a fragile optimization — the edge is structural.

### Which Signals Benefit from 15S?

| Signal | 1M avgR | 15S avgR | Verdict |
|--------|---------|---------|---------|
| **BOS_FVG** | +0.690 | **+0.895** | MASSIVE improvement (use 15S) |
| **vol_spike_FVG** | +0.316 | **+0.686** | MASSIVE improvement (use 15S) |
| ORB_breakout | +0.432 | +0.471 | Slight improvement (use 15S) |
| engulfing_key_level | +0.117 | +0.095 | Slightly worse (keep 1M) |
| pre_gap_fill | +0.229 | +0.241 | About same (keep 1M) |
| **VA_fade** | +0.123 | **-0.206** | **DIES on 15S** (keep 1M) |

Why VA_fade dies: mean-reversion signals need room to breathe. 15S trailing gets shaken out by noise on the way back to POC. Trend signals (BOS, vol_spike) benefit because they have fat tails to capture.

### HWM Analysis (BOS_FVG, 15S trig=1R trail=0.5R)

| | Avg HWM | Avg Actual R | Avg Giveback |
|--|---------|-------------|-------------|
| Winners (n=465) | +2.43R | +1.85R | 0.58R |
| Losers (n=220) | +0.22R | -1.12R | — |

HWM distribution: P50 +1.46R, P90 +3.68R, P95 +4.56R, Max +13.86R.
Tighter trails (0.1R-0.15R) recover much of that 0.58R giveback from winners.

---

## 3. Production Portfolio (V10z Conservative, FLAGSHIP)

After V10z audit (conservative intra-bar sim), only 3 signals survive. All on 15S trailing.

### Portfolio Composition

| Signal | Config | N | avgR | Total R | R/Day | WF OOS | Q+ |
|--------|--------|---|------|---------|-------|--------|-----|
| **BOS_FVG** | 15S trig=0.5R trail=0.1R | 685 | **+1.141** | **+781.8** | +2.51 | +1.149 | 4/4 |
| vol_spike_FVG | 15S trig=1.0R trail=0.5R | 88 | +0.665 | +58.5 | +0.19 | +1.059 | 3/4 |
| ORB_breakout | 15S trig=1.0R trail=1.0R | 114 | +0.471 | +53.7 | +0.17 | +0.771 | 3/4 |
| **TOTAL** | | **887** | **+1.008** | **+894.0** | **+2.87** | +0.996 | 4/4 |

### Monthly P&L (all positive)

| Month | Trades | Total R | avgR |
|-------|--------|---------|------|
| Feb 2025 | 50 | +62.7 | +1.255 |
| Mar 2025 | 122 | +117.7 | +0.965 |
| Apr 2025 | 91 | +111.0 | +1.220 |
| May 2025 | 83 | +68.1 | +0.820 |
| Jun 2025 | 66 | +29.7 | +0.449 |
| Jul 2025 | 76 | +23.6 | +0.311 |
| Aug 2025 | 60 | +33.9 | +0.565 |
| Sep 2025 | 90 | +85.9 | +0.954 |
| Oct 2025 | 85 | +65.9 | +0.776 |
| Nov 2025 | 90 | +80.2 | +0.891 |
| Dec 2025 | 72 | +42.2 | +0.586 |
| Jan 2026 | 79 | +94.4 | +1.195 |
| Feb 2026 | 44 | +71.6 | +1.628 |

### Killed by V10z Audit

| Signal | Old avgR | Conservative avgR | Why |
|--------|---------|-------------------|-----|
| engulfing_key_level | +0.117 | **-0.105** | Trailing edge was intra-bar bug |
| VA_fade | +0.123 | **+0.037** → dies on 15S | Mean-reversion needs wide trail |
| pre_gap_fill | +0.229 | **-0.059** | B/E edge was intra-bar bug |

### Comparison

| Portfolio | Total R | R/Day | Calmar | Signals |
|-----------|---------|-------|--------|---------|
| V10t Fixed 1M | +449.0 | +1.44 | 17.4 | 6 |
| V10v Trail 1M (optimistic) | +744.9 | +2.39 | 59.5 | 6 |
| **V10z Conservative 15S** | **+894.0** | **+2.87** | **94.4** | **3** |
| BOS_FVG alone (conservative) | +781.8 | +2.51 | 263.6 | 1 |

**BOS_FVG drives 87% of total portfolio R.** The other 2 signals add diversification (+112R) but the system is effectively a BOS_FVG system.

---

## 4. BOS_FVG Deep Dive

**Recommended production portfolio: 6 signals** (drop london_breakout, which dies at 0.75pt entry slippage).

### Portfolio Composition

| Signal | Config | N | avgR | Calmar | OOS avgR | Q+ |
|--------|--------|---|------|--------|----------|-----|
| BOS_FVG | trig=1.0R trail=0.5R | 685 | +0.732 | 57.1 | +0.696 | 4/4 |
| engulfing_key_level | trig=1.0R trail=0.75R | 1,096 | +0.134 | 12.4 | +0.205 | 2/4 |
| ORB_breakout | trig=1.0R trail=1.0R | 114 | +0.440 | 4.9 | +0.683 | 2/4 |
| vol_spike_FVG | trig=1.0R trail=0.5R | 88 | +0.351 | 3.2 | +0.573 | 3/4 |
| london_breakout | trig=1.0R trail=0.5R | 713 | +0.077 | 3.0 | +0.216 | 4/4 |
| VA_fade | trig=1.0R trail=0.5R | 319 | +0.149 | 2.7 | +0.320 | 3/4 |
| pre_gap_fill | FIXED T5.0R BE=1.0 | 121 | +0.237 | 1.8 | +0.682 | 2/4 |

### Combined Performance (3,136 trades, 312 days)

| Metric | Value |
|--------|-------|
| Trades/Day | **~10.1** |
| Win Rate | 49.0% |
| Avg R/Trade | **+0.274** |
| Total R | **+860.1** |
| R/Day | **+2.76** |
| Max Drawdown | **-17.5R** |
| Calmar Ratio | **49.0** |
| Daily Win Rate | **67.8%** |
| Weekly Win Rate | **92.3%** |

### Walk-Forward Validation

| | IS (70%) | OOS (30%) |
|--|----------|-----------|
| n | 2,236 | 900 |
| avgR | +0.242 | **+0.354** |
| Q1 OOS | — | **+0.337** |
| Q2 OOS | — | **+0.059** |
| Q3 OOS | — | **+0.325** |
| Q4 OOS | — | **+0.294** |

**OOS is better than IS** (+0.354 vs +0.242) — no overfitting. All 4 quarters positive.

### With Daily Kill Switch (-3R)

| Metric | No Kill | Kill -3R |
|--------|---------|----------|
| Trades | 3,136 | 2,551 |
| avgR | +0.274 | +0.288 |
| Total R | +860.1 | +733.9 |
| R/Day | +2.76 | +2.35 |
| Calmar | 49.0 | 31.3 |

### Why Trailing Beats Fixed (Per-Signal Comparison)

| Signal | Fixed avgR | Trail avgR | Improvement |
|--------|-----------|-----------|-------------|
| BOS_FVG | +0.329 | +0.732 | **+122%** |
| VA_fade | +0.043 | +0.149 | **+248%** |
| vol_spike_FVG | +0.250 | +0.351 | **+40%** |
| ORB_breakout | +0.334 | +0.440 | **+32%** |
| engulfing_key_level | +0.110 | +0.134 | **+22%** |
| london_breakout | DEAD | +0.077 | **NEW** |
| pre_gap_fill | +0.237 | +0.237 | 0% (stays fixed) |

Key insight: **london_breakout was dead with fixed targets but alive with trailing** — trailing
unlocks edge that fixed targets miss because some signals need unlimited upside to be profitable.

---

BOS_FVG is the core signal, generating **72% of the hybrid portfolio's total R** from only 28% of trades.

### The Setup
After a **break of structure** (BOS) on the 1-minute chart, price creates a **fair value gap** (FVG).
When price retraces to fill the FVG, enter in the direction of the BOS. Minimum 3 confluence factors required.

### Trade Management: 15S Trailing Stop (PRODUCTION)

| Parameter | Value |
|-----------|-------|
| **Signal Detection** | 1-minute bars |
| **Trail Management** | **15-second bars** |
| **Trigger** | 0.5R (trailing activates after price moves 0.5R in your favor) |
| **Trail Distance** | 0.1R (stop follows 0.1R behind the high-water mark) |
| **Target Cap** | None (unlimited upside) |
| **Max Hold** | 120 minutes |
| **Exit Slippage** | 0.5 pts |

### Performance — 15S Production Config (685 trades, 312 days)

| Metric | 15S (trig=0.5R trail=0.1R) | 1M (trig=1.0R trail=0.5R) |
|--------|---------------------------|--------------------------|
| Win Rate | **75.8%** | 45.3% |
| Avg R/Trade | **+1.146** | +0.690 |
| Total R | **+785.1** | +472.6 |
| R/Day | **+2.52** | +1.51 |
| Max Drawdown | **-3.0R** | -9.5R |
| Calmar Ratio | **264.7** | 49.8 |
| WF IS avgR | **+1.140** | +0.896 |
| WF OOS avgR | **+1.158** | +0.893 |
| Quarterly OOS+ | **4/4** | 4/4 |

**15S turns BOS_FVG into a 75.8% WR system with +1.15 avgR.** The tight trail (0.1R) locks in gains quickly on the 15S timeframe — 4x per minute instead of once per minute. Winners still run because the trail follows the HWM up. Losers get stopped at -1R as before.

### Why 15S Works: Fat Tail Capture
On 1M bars, price can spike +3R and reverse to +1R within a single candle — the trailing stop never sees the peak. On 15S bars, the stop tracks through 4 price updates per minute, catching peaks before reversals. This:
- **Increases WR** from 45% to 76% (more trades hit trigger and get trailed to profit)
- **Increases avgR** from +0.69 to +1.15 (less giveback from HWM)
- **Decreases DD** from -9.5R to -3.0R (tighter exits limit damage)

### Winner/Loser Profile (15S, trig=1R trail=0.5R)
| | Count | Avg R | Avg HWM |
|--|-------|-------|---------|
| Winners | 465 | **+1.85R** | +2.43R |
| Losers | 220 | -1.12R | +0.22R |

HWM distribution: P50 +1.46R, P90 +3.68R, P95 +4.56R, Max +13.86R.
Average giveback from HWM: 0.58R on winners. The 0.1R trail reduces this further.

### All 14 Tight Trailing Configs Pass Walk-Forward
Every config from trig=0.5R to trig=2.0R with trails from 0.1R to 0.5R **passes with 4/4 Q+ OOS**.
Not a fragile optimization — the edge is structural across the full parameter space.

### Sensitivity to Parameters (15S, V10z Conservative)

| Trigger | Trail | ~Points | WR | avgR | Calmar | R/Day |
|---------|-------|---------|-----|------|--------|-------|
| **0.5R** | **0.1R** | **~0.6pt** | **75.8%** | **+1.141** | **263.6** | **+2.51** |
| 0.5R | 0.15R | ~0.9pt | 75.8% | +1.103 | 250.6 | +2.42 |
| 0.5R | 0.25R | ~1.5pt | 75.8% | +1.028 | 225.9 | +2.26 |
| 0.5R | 0.5R | ~3pt | 72.3% | +0.842 | 171.3 | +1.85 |
| 0.75R | 0.15R | ~0.9pt | 72.7% | +1.092 | 201.1 | +2.40 |
| 1.0R | 0.25R | ~1.5pt | 67.9% | +0.982 | 108.9 | +2.16 |
| 1.0R | 0.5R | ~3pt | 67.9% | +0.814 | 86.7 | +1.79 |

**Tighter trail = better** on 15S. Even with conservative sim, 0.1R trail dominates.

### Delay Trail Activation (V10z "Breathing Room" Test)

The edge is **front-loaded** — BOS_FVG produces a fast burst. Delaying the trail destroys Calmar:

| Delay | WR | avgR | Calmar | vs No Delay |
|-------|-----|------|--------|-------------|
| **0 sec** | **75.8%** | **+1.141** | **263.6** | baseline |
| 15 sec | 75.8% | +1.141 | 263.6 | identical |
| 30 sec | 67.6% | +1.008 | 106.3 | **-60%** |
| **60 sec** | 47.7% | +0.459 | **25.4** | **-90%** |
| 120 sec | 36.9% | +0.463 | 25.4 | -90% |
| 300 sec | 25.7% | +0.406 | 6.2 | -98% |

**Do NOT delay trail activation.** The initial burst IS the edge.

---

## 5. Core Concepts

### R-Multiple
Every trade's profit/loss is normalized to risk:
- Entry at 21050, Stop at 21040 → Risk = 10 pts = 1R
- If trade makes 20 pts → +2R. If stopped out → -1R.

### Fixed R Targets
V10s/V10t discovered that swing-based targets had look-ahead bias (using future 5M swing points).
The validated system uses **fixed R targets**:
- Target = Entry ± (Target_R × Risk)
- Each signal has its own optimal target R (ranging from 2R to 5R)

### B/E Management
Some signals use breakeven management:
- When price reaches B/E_R in your favor, move stop to entry + slippage
- Protects capital at the cost of some winning trades getting stopped at B/E

### Session Windows (UTC)
Trades are generated during high-quality sessions only:
- news_window: 13:30-14:00 (8:30-9:00 ET)
- ib_first: 14:45-15:30 (9:45-10:30 ET)
- ny_am: 16:00-17:00 (11:00-12:00 ET)
- silver_bullet: 18:30-19:30 (1:30-2:30 ET)
- power_hour: 20:00-21:00 (3:00-4:00 ET)

### Confluence System
All signals require minimum 3 confluence factors (e.g., BOS + FVG_fill + volume_confirmed).
This filters out low-quality setups.

---

## 6. All 3 Production Signals (V10z Conservative)

Only 3 signals survive the conservative intra-bar simulation audit. All on 15S.

### 6.1 BOS_FVG (see Section 4 for deep dive)
**Bar Freq**: 15S | **Config**: trig=0.5R trail=0.1R | **N=685** | Calmar 263.6 | OOS +1.149 | Q+: 4/4

---

### 6.2 Volume Spike + FVG Fill
**Bar Freq**: 15S | **Config**: trig=1.0R trail=0.5R | **N=88**

| Metric | Conservative 15S |
|--------|-----------------|
| WR | 62.5% |
| avgR | **+0.665** |
| Calmar | 11.6 |
| OOS avgR | +1.059 |
| Q+ | 3/4 |

**Entry**: Volume spike (3x+) creates displacement and FVG. Enter on fill.
Same fat-tail logic as BOS_FVG. Small sample — supplemental signal.

---

### 6.3 ORB Breakout
**Bar Freq**: 15S | **Config**: trig=1.0R trail=1.0R | **N=114**

| Metric | Conservative 15S |
|--------|-----------------|
| WR | 67.5% |
| avgR | +0.471 |
| Calmar | 5.2 |
| OOS avgR | +0.771 |
| Q+ | 3/4 |

**Entry**: Breakout above/below 15-min opening range with VWAP alignment.
Uses widest trail (1.0R) because ORB moves are volatile and need breathing room.

---

### Dead Signals (V10z Audit)

| Signal | Old avgR | Conservative avgR | Why Dead |
|--------|---------|-------------------|----------|
| engulfing_key_level | +0.117 | **-0.105** | Trailing edge was intra-bar ordering bug |
| VA_fade | +0.123 | **+0.037** (near zero) | Mean-reversion gets shaken out |
| pre_gap_fill | +0.229 | **-0.059** | B/E edge was intra-bar ordering bug |
| london_breakout | +0.077 | — | Dies at 0.75pt entry slippage (V10w) |

---

## 7. 1M Trailing Portfolio (V10v Baseline)

V10v applied trailing stops to ALL 13 signal types on 1M bars. 7 passed walk-forward. Superseded by V10y hybrid portfolio but kept for reference.

| Metric | Value |
|--------|-------|
| Signals | 6 (production) |
| Trades | 2,423 |
| avgR | +0.307 |
| Total R | +744.9 |
| R/Day | +2.39 |
| Calmar | 59.5 |

---

## 8. Fixed R Portfolio (V10t Baseline)

Superseded by V10v trailing portfolio but kept for reference.

### Previous Validated Signals — Fixed R Targets (6)

### 3.1 Engulfing at Key Level
**Config**: Target 2.0R, No B/E | **N=1096** (highest volume)

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 34.9% | 32.9% | 40.3% |
| avgR | +0.110 | +0.070 | +0.218 |
| Quarterly OOS+ | — | — | **4/4** |

**Entry**: Engulfing candle pattern at a key level (swing point or FVG zone). Requires:
- Engulfing candle with body > prior candle range
- Located at FVG fill zone or swing level
- Minimum 3 confluence factors

**Why it works**: Engulfing candles at structure show aggressive reversal at validated support/resistance. The 2R target is achievable because the reversal move typically extends 2x the entry candle range.

---

### 3.2 BOS + FVG Fill
**Config**: Target 5.0R, B/E at 1.0R | **N=685**

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 18.5% | 19.1% | 17.5% |
| avgR | +0.329 | +0.342 | +0.304 |
| Quarterly OOS+ | — | — | **3/4** |

**Entry**: After a break of structure on 1M, price fills the FVG (fair value gap) created during the displacement. Entry at the FVG fill price with stop below the gap.

**Why it works**: BOS shows institutional order flow. The FVG is an "unfilled imbalance" — when price returns to fill it, it's a high-probability continuation point. The 5R target works because BOS displacement moves are large enough to generate outsized winners, and B/E management protects against false breakouts.

---

### 3.3 VA Fade
**Config**: Target 2.0R, B/E at 1.0R | **N=319**

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 27.3% | 26.5% | 29.4% |
| avgR | +0.043 | +0.004 | +0.150 |
| Quarterly OOS+ | — | — | **4/4** |

**Entry**: Price reaches the Value Area High (VAH) or Value Area Low (VAL) from the prior session and reverses. Fade (trade against) the extreme of the value area back toward the POC.

**Why it works**: 70% of volume trades within the value area. When price reaches the extremes (VAH/VAL), mean-reversion pressure pushes it back toward POC. B/E management keeps losses small when breakouts occur.

---

### 3.4 Pre-Gap Fill (5M Displacement → 1M Entry)
**Config**: Target 5.0R, B/E at 1.0R | **N=121**

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 9.1% | 5.7% | 18.2% |
| avgR | +0.237 | +0.070 | +0.682 |
| Quarterly OOS+ | — | — | 2/4 |

**Entry**: Aggressive 5M displacement bar creates FVG. Switch to 1M and enter when price retraces into the gap zone with a confirming candle. Small N — use with caution.

**Caution**: OOS much better than IS, and only 121 trades total. This is the weakest validated signal.

---

### 3.5 Opening Range Breakout (ORB)
**Config**: Target 3.0R, No B/E | **N=114**

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 7.9% | 7.2% | 8.9% |
| avgR | +0.334 | +0.280 | +0.415 |
| Quarterly OOS+ | — | — | 2/4 |

**Entry**: After the 15-minute opening range is established (9:45 ET), trade the breakout above range high or breakdown below range low, with VWAP alignment. Minimum range of 10 pts.

**Caution**: Low WR (7.9%) but high avgR from rare large winners. Only 114 trades total.

---

### 3.6 Volume Spike + FVG Fill
**Config**: Target 5.0R, B/E at 1.0R | **N=88**

| Metric | Full | IS | OOS |
|--------|------|-----|-----|
| WR | 17.0% | 12.1% | 26.7% |
| avgR | +0.250 | +0.043 | +0.653 |
| Quarterly OOS+ | — | — | 2/4 |

**Entry**: Volume spike (3x+ average) creates displacement and FVG. Enter on the FVG fill with B/E protection. Smallest sample — use as supplemental only.

---

## 8b. Combined Portfolio (Fixed R — Superseded)

All 6 signals, each with its validated config:

| Metric | No Kill | Kill -3R |
|--------|---------|----------|
| Trades | 2,423 | 1,874 |
| Trade WR | 26.1% | 25.3% |
| Avg R/Trade | +0.185 | +0.179 |
| Total R | +449.0 | +335.2 |
| R/Day | +1.44 | +1.07 |
| Max DD | -25.8R | -29.2R |
| Calmar | 17.4 | 11.5 |
| Daily WR | 56.4% | 47.5% |
| Weekly WR | 73.1% | 67.3% |

---

## 9. Walk-Forward Results (Fixed R Portfolio — Superseded)

### Full IS/OOS Split (70/30)
| | n | WR | avgR | totR |
|---|---|-----|------|------|
| IS (70%) | ~1700 | 25.3% | +0.162 | ~275 |
| OOS (30%) | ~720 | 28.1% | +0.243 | ~175 |

**OOS is BETTER than IS** — no evidence of overfitting.

### Quarterly Breakdown
| Quarter | IS avgR | OOS avgR |
|---------|---------|----------|
| Q1 | +0.263 | +0.227 |
| Q2 | +0.037 | +0.090 |
| Q3 | +0.212 | +0.266 |
| Q4 | +0.253 | +0.135 |

All 4 quarters positive OOS.

---

## 9b. Signal Ranking (Fixed R — Superseded)

By robustness (N × Q+):

| Rank | Signal | Trades | OOS avgR | Q+ | Confidence |
|------|--------|--------|----------|-----|------------|
| 1 | engulfing_key_level | 1,096 | +0.218 | 4/4 | HIGH |
| 2 | BOS_FVG | 685 | +0.304 | 3/4 | HIGH |
| 3 | VA_fade | 319 | +0.150 | 4/4 | MEDIUM |
| 4 | pre_gap_fill | 121 | +0.682 | 2/4 | LOW |
| 5 | ORB_breakout | 114 | +0.415 | 2/4 | LOW |
| 6 | vol_spike_FVG | 88 | +0.653 | 2/4 | LOW |

---

## 10. Dead Signals (6 — No Edge Even With Trailing)

| Signal | N | Best Trailing avgR | Fixed avgR | Verdict |
|--------|---|-------------------|-----------|---------|
| VP_POC_retest | 657 | -0.061 | -0.107 | DEAD |
| fib_retracement | 568 | +0.020 (fails WF) | -0.010 | DEAD |
| failed_breakout | 556 | -0.095 | -0.137 | DEAD |
| sweep_reversal | 403 | +0.028 (fails WF) | -0.001 | DEAD |
| double_BOS_momentum | 363 | -0.059 | -0.116 | DEAD |
| VWAP_mean_reversion | 187 | -0.062 | -0.137 | DEAD |

Tested with both fixed R targets (1.5R-5R) and trailing stops (7 configs). None pass
walk-forward validation. Note: london_breakout was previously dead with fixed targets
but **was rescued by trailing stops** (see Section 5.5).

---

## 11. Dead Ends — Comprehensive

From V8 and V10 research combined:

| What | Result | Status |
|------|--------|--------|
| Multi-TF alignment (forming bars) | 48% agreement with clean alignment | LOOK-AHEAD |
| All 5 clean alignment methods | Zero edge at any alignment level | DEAD |
| BE at 1R + trail from 2R (V10aa) | 21.8% WR on 1M, 30.7% on 15S vs 75.8% pure trail | DEAD |
| BE+Trail + look-ahead alignment (V10aa) | Max 38.7% WR at align>=4 — still uses bias | DEAD |
| V10q 12 individual filters | ALL negative avgR | DEAD |
| V10q 19 multi-filter combos | ALL negative avgR | DEAD |
| B/E on Model 2 trades | Kills edge across 11 configs | DEAD |
| Inverse trades | Noise-driven stopouts | DEAD |
| Re-entry after correct signal | -0.220 avgR | DEAD |
| Stacked imbalances standalone | 18% WR, too rare | DEAD |
| Exhaustion prints standalone | 36.6% WR, too noisy | DEAD |
| Absorption as filter | Only 28 trades at threshold | DEAD |

---

## 12. Look-Ahead Audit (V10o)

### What Was Wrong
The alignment code used `tf_candle.index <= entry_time`, which selects the bar whose
**start time** is before entry. But the bar's **close** is at `start_time + bar_duration`.
For a 1-hour bar starting at 10:00, entry at 10:03 would use the 10:00-11:00 bar's close
— 57 minutes of future data.

### How We Fixed It
```python
# OLD (look-ahead): Uses forming bar close
mask = tf_candle.index <= entry_time

# FIXED: Only fully CLOSED bars
mask = tf_candle.index + TF_DURATION[tf_label] <= entry_time
```

### Impact
- **V10c (look-ahead)**: 8,140 trades, 31.3% WR, +0.714 avgR, +5,814.9 totR
- **V10r (fixed)**: 678 trades, 2.2% WR, +0.039 avgR, +26.5 totR

Additionally, `extended_5m` (used for target swing computation) had no end-time filter,
allowing future 5M candles into the target calculation. Fixed by capping at session end
and filtering target swings by `confirmed_time <= entry_time`.

### Recovery
V10s/V10t switched from swing-based targets (which had look-ahead in target selection)
to **fixed R targets**. This recovered **+1.44 R/day** with walk-forward validation.

---

## 13. Risk & Realism

### V10w Slippage Validation
Entry slippage tested from 0 to 1.0pts. **All signals survive 0.5pt entry slippage except london_breakout** (dies at 0.75pt). BOS_FVG is a tank: +0.558 avgR even with 1.0pt entry slippage.

### V10z Production Portfolio (3 signals, conservative sim, 0.25pt entry + 0.5pt exit)
- 887 trades over 312 days = 2.8 trades/day
- avgR = +1.008, +2.87 R/day, Total R = +894.0

With 1R = 5.9 pts median NQ, $5/pt (MNQ):
- Gross: 2.87 R/day × 5.9pts × $5 = **+$84.67/day/contract**
- Commission: 2.8 trades × $2.50 × 2 sides = $14/day
- **Net: +$70.67/day/contract**
- Annualized (258 days): **+$18,233/year per MNQ contract**

With NQ ($20/pt): **+$283/day, +$73,000/year per NQ contract**

### BOS_FVG 15S Only (Simplest, Highest Calmar)
2.2 trades/day, conservative sim:
- **Net: +$63/day per MNQ, +$252/day per NQ**
- Calmar 263.6, max DD only -3.0R. Simplest to execute.

### Recommended Production Config
- **Signal**: BOS_FVG on 15S (the only one that really matters)
- **Trail**: 0.25R (1.5 pts = 6 ticks) — safer than 0.1R (2 ticks) for execution
- **At 0.25R trail**: +1.028 avgR, Calmar 225.9, still passes WF 4/4
- vol_spike_FVG and ORB_breakout add +112R total — include if execution supports 3 signals

### Slippage Stress Test (V10z, BOS_FVG 15S trig=0.5R trail=0.1R)

| Test | avgR | Calmar | Status |
|------|------|--------|--------|
| 0.25pt entry, 0.5pt exit (baseline) | +1.141 | 263.6 | ALIVE |
| 0.5pt entry, 0.5pt exit | +1.096 | 236.8 | ALIVE |
| 1.0pt entry, 0.5pt exit | +1.004 | 147.4 | ALIVE |
| 0.25pt entry, 2.0pt exit (brutal) | +0.893 | 145.6 | **ALIVE** |

### V10aa: BE+Trail Replication (Harrison's Mining Spec)

Tested Harrison's original spec: BE at 1R, trail from 2R, with and without alignment score.

| Config | WR | avgR | Total R | Calmar |
|--------|-----|------|---------|--------|
| BE+Trail 1M, filtered (685) | 21.8% | +0.335 | +229R | 7.7 |
| BE+Trail 15S, filtered (685) | 30.7% | +0.563 | +386R | 33.4 |
| BE+Trail 1M, raw (1315) | 23.3% | +0.375 | +493R | 11.6 |
| BE+Trail 15S, raw (1315) | 31.8% | +0.596 | +784R | 110.8 |
| **Pure trail 15S** (685) | **75.8%** | **+1.141** | **+782R** | **263.6** |

**BE at 1R kills the edge.** Trades that dip below entry before running get stopped at breakeven. Pure trailing keeps them alive.

With look-ahead alignment (forming bar — HAS BIAS):
- align>=3: 30.9% WR, +0.703 avgR
- align>=4: 38.7% WR, +1.108 avgR
- Closest to 63.5% WR: 15S BE=1.0 trig=1.0 trail=0.5 align>=4 → 56.3% WR

**Conclusion**: Harrison's 63.5% WR / +15,368R required look-ahead alignment (proven biased in V10o). Our pure trailing at 75.8% WR without any alignment is genuinely superior and clean.

### FVG Edge (No Buffer)
Removing the 2pt stop buffer (Harrison spec: stop at FVG edge):
- With buffer: +0.317 avgR, Calmar 7.8
- No buffer: +0.772 avgR, Calmar 25.2
Wider risk = more room = fewer stops. But pure trailing still dominates both.

### Not Yet Accounted For
- Spread costs (beyond slippage)
- Partial fills on limit orders
- Regime changes / structural breaks
- Execution latency on 15S trail updates
- Real tick-by-tick simulation (NautilusTrader needed)

### Monte Carlo Validation (V10x, 10,000 sims)
Using 6-signal portfolio at 0.25pt entry + 0.5pt exit slippage:

| Metric | P5 (worst) | P50 (median) | P95 (best) |
|--------|-----------|-------------|-----------|
| Max Drawdown | -31.6R | -21.5R | -16.1R |
| Calmar | 23.6 | 34.6 | 46.2 |
| Max Losing Streak | 13 | 10 | 8 |

**Zero losing months** in actual data (13/13 positive). Best: Nov 2025 (+5.95 R/day).

### Direction & Timing Edge
- **SHORT dominates**: +0.418 avgR vs LONG +0.192 avgR (69% of profit from shorts)
- **Best session**: news_window (8:30-9:00 ET) +0.454 avgR
- **Best volume session**: ib_first (9:45-10:30 ET) +292.7R total (39% of portfolio)
- **Best day**: Thursday (+0.419 avgR), worst: Monday (+0.165 avgR)

### Not Yet Accounted For
- Spread costs (beyond slippage)
- Partial fills on limit orders
- Regime changes / structural breaks
- Execution latency

---

## 14. Glossary

| Term | Definition |
|------|-----------|
| R | Risk unit: 1R = distance from entry to stop, in points |
| WR | Win Rate: percentage of trades that hit target |
| avgR | Average R per trade (total R / number of trades) |
| DWR | Daily Win Rate: percentage of days with positive R |
| WWR | Weekly Win Rate: percentage of weeks with positive R |
| Calmar | Total R / Max Drawdown — higher is better |
| BOS | Break of Structure: price breaks a prior swing high/low |
| FVG | Fair Value Gap: 3-candle pattern leaving unfilled price gap |
| IS | In-Sample: training data (first 70% chronologically) |
| OOS | Out-of-Sample: test data (last 30% chronologically) |
| Q+ | Quarters with positive OOS performance |
| B/E | Breakeven: stop moved to entry after price reaches trigger |
| VAH/VAL | Value Area High/Low: boundaries of 70% volume zone |
| POC | Point of Control: price level with most volume |
| ORB | Opening Range Breakout: trade the break of first 15min range |
| Trailing Stop | Stop that follows price at a fixed distance behind the high-water mark |
| HWM | High Water Mark: the maximum favorable excursion (best price reached) |
| Trigger R | R-level at which the trailing stop activates (e.g., 1.0R) |
| Trail R | Distance behind HWM that the trailing stop follows (e.g., 0.5R) |
