# Tempo NQ Research Journal
## The Full Story — V7 through V11

**Author**: Harrison Willis + Claude
**Period**: January — February 2026
**Instrument**: NQ / MNQ (Nasdaq-100 E-mini / Micro E-mini Futures)
**Data**: 312 trading days of Databento tick data, rebuilt into 1M/5M/15S bars

---

## The One-Paragraph Summary

We ran 115+ backtesting experiments across 4 major research versions (V7–V11), testing every order flow signal, filter, alignment method, and trade management technique we could think of against a full year of NQ tick data. Along the way we discovered a critical look-ahead bias that invalidated 14 versions of results, killing every multi-timeframe alignment strategy we'd built. After the audit, we rebuilt from scratch with paranoid look-ahead prevention and found that **one signal dominates everything: BOS_FVG** (Break of Structure into Fair Value Gap). Combined with 15-second trailing stops — our biggest breakthrough — it produces +1.14R per trade, Calmar 263.6, and has never had a losing month across 312 days. Two minor signals (vol_spike_FVG, ORB_breakout) add marginal diversification. Everything else is dead.

---

## Phase 1: The Mining Era (V7–V8)

### What We Built

We started with `intraday_scanner.py` (95KB), a massive engine that ingests raw Databento tick data and reconstructs:
- 1-minute and 5-minute candles with full OHLCV
- Swing points with configurable lookback confirmation
- Break of Structure (BOS) events
- Fair Value Gaps (FVGs) with fill tracking
- Order flow features: delta, cumulative delta, stacked imbalances, absorption, exhaustion prints

The V7 series (~50 scripts) was the parameter mining phase — brute-force testing of every filter combination across session windows, displacement thresholds, FVG sizes, swing lookbacks, and structural overlays.

### V8: Strategy Composition

V8 shifted from parameter mining to signal composition. Key experiments:

- **V8a–V8f**: Built "Strategy A" — a composite of rejection bars, delta alignment, and session filters
- **V8g**: Multi-timeframe candle alignment — tested whether aligning trades with 15M/1H/4H/Daily candle direction improves WR. **It appeared to work spectacularly** (70%+ WR with alignment >= 5). This became the foundation of everything for months.
- **V8h–V8k**: Deep dives on rejection entries, delta overlays, volume filters
- **V8l–V8m**: Equity curve analysis and master synthesis
- **V8n**: 15-second delta overlay testing
- **V8o–V8ab**: Walk-forward validation, higher-TF BOS, enhanced portfolios, entry optimization

**V8 conclusions at the time**: Strategy A with multi-TF alignment looked like a 70%+ WR system. We had massive confidence. We were wrong.

---

## Phase 2: V9 — Production Spec

V9 was short (6 scripts) — the goal was to nail down realistic execution parameters:

- **V9b**: Realistic execution modeling (slippage, commissions, partial fills)
- **V9c**: Stacked filter optimization
- **V9d**: Drawdown recovery analysis
- **V9e**: Target/stop optimization
- **V9f**: Final production specification

V9 produced a production spec, but it was built on V8's alignment results. All of it would need to be thrown out.

---

## Phase 3: The Reckoning — V10o Look-Ahead Audit

### The Bug

V10a–V10n continued building on multi-TF alignment. Then **V10o changed everything**.

The alignment code used this mask:
```python
mask = tf_c.index <= candles_1m.index[idx]
```

This selects all higher-timeframe bars whose **start time** is before entry. But a 1H bar that starts at 10:00 doesn't close until 11:00. If entry is at 10:03, the code was using a close price that's **57 minutes in the future**.

For 15M bars: up to 14 minutes of look-ahead. For 4H bars: up to 3 hours and 59 minutes. For Daily bars: the entire day.

### The Fix
```python
mask = tf_c.index + TF_DURATION[tf_label] <= entry_time
```
Only use bars that have **fully closed** before entry.

### The Fallout

With the fix applied: every alignment strategy collapsed to ~50% WR. The entire "edge" was look-ahead. We tested 5 alternative clean alignment methods in V10p:

1. `completed_dir` — direction of last completed bar
2. `open_vs_price` — price vs forming bar open
3. `prev_close_vs_price` — price vs previous bar close
4. `ema_trend` — EMA direction on completed bars
5. `multi_bar` — majority direction of last N bars

**All 5 methods: DEAD.** Every combination had negative avgR, Calmar 1.0 (pure drawdown). There is no multi-timeframe candle alignment edge in NQ with clean data.

### The Lesson

Look-ahead bias is the #1 killer in backtesting. It doesn't just inflate results — it creates phantom edges that don't exist. We lost ~2 weeks of research. The silver lining: we found it before going live.

---

## Phase 4: The Rebuild — V10r through V10t

After the audit, we rebuilt everything with paranoid look-ahead prevention:

- Swing points require `lb` bars of confirmation **after** formation
- BOS only fires on confirmed swings (`confirmed_index < current bar`)
- FVG fill checks start the bar **after** the FVG forms
- Entry is at the start of the **next** bar after fill (not the fill bar)
- Stop checks start the bar **after** entry
- Trailing only uses completed bar data

### V10r: Native Swing Targets (Clean)

Tested all signals with swing-level targets (the "natural" exit). Only one signal survived:

**BOS_FVG**: 361 trades, 62.6% WR, +0.460 avgR, Calmar 26.0
Walk-forward: IS +0.372 → OOS +0.350 (rock solid)

Everything else was negative with native targets.

### V10s–V10t: Fixed R Targets (Recovery)

Fixed R targets (2R, 3R, 5R) rescued some signals that couldn't reach swing targets:

| Signal | Trades | Best Config | OOS avgR | Quarters+ |
|--------|--------|------------|----------|-----------|
| engulfing_key_level | 1096 | T2.0R no BE | +0.218 | 4/4 |
| BOS_FVG | 685 | T5.0R BE=1.0 | +0.304 | 3/4 |
| VA_fade | 319 | T2.0R BE=1.0 | +0.150 | 4/4 |
| pre_gap_fill | 121 | T5.0R BE=1.0 | +0.682 | 2/4 |
| ORB_breakout | 114 | T3.0R no BE | +0.415 | 2/4 |
| vol_spike_FVG | 88 | T5.0R BE=1.0 | +0.653 | 2/4 |

6 signals, 2423 trades, +1.44 R/day combined. But we hadn't stress-tested trailing yet.

---

## Phase 5: The Breakthrough — V10y 15-Second Trailing

### The Thesis

1-minute bars are too coarse for trailing stops. Price can run +3R and reverse to +1R within a single 1M candle — the trailing stop never sees the high. 15-second bars give **4x more granular updates**, catching fat-tail peaks before reversals.

### The Result

BOS_FVG head-to-head, same 685 trades:

| Trailing | 1M avgR | 15S avgR | 1M Calmar | 15S Calmar |
|----------|---------|----------|-----------|------------|
| trig=0.5R trail=0.25R | +0.748 | **+1.054** | 52.2 | **231.6** |
| trig=0.5R trail=0.1R | +0.820 | **+1.141** | 60.1 | **263.6** |
| trig=1.0R trail=0.5R | +0.690 | **+0.895** | 49.8 | **95.3** |

**15S wins on every single configuration.** Tighter trails benefit the most because they need the granularity to avoid getting shaken out by intra-bar noise.

Best config: **trig=0.5R trail=0.1R on 15S** — 685 trades, +1.141 avgR, Calmar 263.6, every month positive.

This was the single biggest improvement in the entire research program.

---

## Phase 6: The Final Audit — V10z Conservative Simulation

### The Problem

Standard backtesting assumes favorable intra-bar ordering (high before low in longs, low before high in shorts). This inflates trailing stop results because the trail ratchets up before the stop gets hit. In reality, the stop might get hit first.

### V10z Fix

Conservative simulation assumes **worst-case** OHLC ordering:
- For longs: check if low hits stop **before** checking if high advances the trail
- Re-check the stop **after** updating the trail high-water mark

### The Casualties

3 signals that looked profitable on 15S trailing were killed by conservative sim:

| Signal | Optimistic avgR | Conservative avgR | Verdict |
|--------|----------------|-------------------|---------|
| engulfing_key_level | +0.117 | **-0.105** | DEAD |
| VA_fade | +0.183 | **+0.037** | DEAD |
| pre_gap_fill | +0.229 | **-0.059** | DEAD |

Their "edge" was the intra-bar ordering bug. On 15S bars, the bug is minimal (15 seconds of ordering ambiguity), which is why BOS_FVG was barely affected.

### The Survivors

| Signal | Trades | avgR | R/Day | Calmar | WF |
|--------|--------|------|-------|--------|-----|
| **BOS_FVG** | **685** | **+1.141** | **+2.51** | **263.6** | **4/4** |
| vol_spike_FVG | 88 | +0.665 | +0.19 | 18.2 | 4/4 |
| ORB_breakout | 114 | +0.471 | +0.17 | 12.3 | 4/4 |

**3-signal portfolio**: 887 trades, +1.008 avgR, +2.87 R/day, Calmar 94.4, 13/13 months positive.

BOS_FVG alone is 87% of portfolio R. The other two add marginal diversification.

---

## Phase 7: Parameter Sweep — clean_backtest.py

With the signal question settled, we optimized BOS_FVG's structural parameters. 87 configurations across 8 phases, 11 hours of compute:

### Key Findings

| Parameter | Change | Impact |
|-----------|--------|--------|
| Swing lookback | 3 → 2 | **+39% Calmar** (biggest single improvement) |
| Risk max cap | 50 → 20 pts | +14% Calmar (blocks sloppy wide-stop trades) |
| BOS displacement | 2.0 → 1.0 | +14% Calmar (more trades, same quality) |
| FVG fill bars | 50 → 30 | Marginal (fresher FVGs slightly better) |
| BOS-FVG max bars | 15 → 20 | Marginal (wider window catches more setups) |

**Composite best**: 6091 trades, +0.924 avgR, +21.81 R/day, Calmar 335.3 (+42% over baseline).

**Overfitting risk: LOW.** The 2nd-best parameter set still beats baseline by +24%. All 87 configurations are profitable. The edge is structural, not parameter-dependent.

---

## Phase 8: NautilusTrader Validation

We ran the full 312-day dataset through NautilusTrader (event-driven backtester) as an independent validation:

- All 8 entry modes profitable
- Best US-session mode: rejection_bar — 9188 trades, 55.6% WR, +0.286 avgR, +10.20 R/day
- **Not apples-to-apples**: Nautilus uses bar.close market entries vs our FVG-edge limit entries (3x more trades but ~0.4R lower avgR)
- Confirms the structural BOS_FVG edge exists across different execution models

---

## The Complete Dead-End List

Everything we tested that doesn't work, so we never waste time on it again:

### Trade Management
- **Break-even stops**: Kill Model 2 edge (confirmed 11 configurations)
- **Inverse trades** (fading losses): Losses are noise-driven stopouts, not directional failures
- **Re-entry after correct signal**: Net negative (-0.220 avgR)
- **Partial profits**: Don't improve without alignment
- **Time stops** (15/30/45 bars): Zero impact on WR or avgR
- **V-reversals**: Negative at every target level
- **Runner management** (B/E + trailing): All 16 configs deeply negative without alignment

### Signals
- **Sweep reversal**: Negative at all R-cap levels
- **VP_POC_retest**: Always negative
- **Fib retracement**: Always negative
- **Failed breakout**: Always negative
- **Double BOS momentum**: Always negative
- **VWAP mean reversion**: Always negative
- **London breakout**: Dead with fixed targets, rescued by trailing but dies at 0.75pt entry slip

### Filters
- **Multi-TF candle alignment** (old method): Look-ahead bias, not a real edge
- **All 5 clean alignment methods**: Zero edge at any level
- **Structural filters** (cluster, body ratio, volume, displacement, VWAP, FVG presence): All dead
- **Stacked imbalances** (standalone): 18% WR, too rare
- **Exhaustion prints** (standalone): 36.6% WR, too noisy
- **Absorption**: Too rare for signal (28 trades at threshold)

---

## Slippage & Execution Stress Tests

BOS_FVG 15S (trig=0.5R trail=0.1R):

| Test | Calmar | Verdict |
|------|--------|---------|
| Baseline (0.25pt entry, 0.5pt exit) | 263.6 | -- |
| 0.5pt entry slip | 236.8 | Survives |
| 1.0pt entry slip | 147.4 | Survives |
| 1.0pt exit slip | 214.7 | Survives |
| **2.0pt exit slip** | **145.6** | **Survives** |
| 1 minute delay | ~26 | **Kills 90% — DO NOT DELAY** |

The edge is front-loaded. Execution speed matters. A 1-minute delay destroys most of the Calmar ratio.

---

## Production Recommendations

### Signal
**BOS_FVG only.** The other two (vol_spike_FVG, ORB_breakout) add <15% of total R. Keep it simple.

### Parameters
```
SWING_LOOKBACK = 2
BOS_MIN_DISPLACEMENT = 1.0
FVG_MAX_FILL_BARS = 30
BOS_FVG_MAX_BARS = 20
RISK_MIN_PTS = 1.5
RISK_MAX_PTS = 20.0
TRAIL_TRIGGER_R = 0.5
TRAIL_BUFFER_R = 0.25   # 0.1R = 2 ticks (too tight for live). 0.25R = 6 ticks (safe).
ENTRY_MODE = fvg_edge
SESSION = 8:00 AM - 3:00 PM ET
LAST_ENTRY = 2:45 PM ET
```

### Expected Performance (Conservative)
- **~6 trades/day**, ~2.5 R/day
- **0.25R trail**: Calmar 225.9 (vs 263.6 with 0.1R)
- With MNQ at $2/pt: ~$30/contract/day after commissions
- With NQ at $20/pt: ~$280/contract/day after commissions
- Max drawdown: ~3R (~$150 MNQ, ~$1500 NQ)
- **Every month positive across 312 days**

### Execution Requirements
- 15-second bar construction from tick data (required for trailing)
- Sub-second order execution (1 min delay kills 90% of edge)
- Limit entry at FVG edge (not market orders)

---

## Project Infrastructure

```
trading-system/
├── clean_backtest.py          # Primary backtester (26.5KB, bar-by-bar, look-ahead verified)
├── intraday_scanner.py        # Tick → candle/swing/BOS/FVG engine (95KB)
├── context-engine/
│   ├── run_v7*.py             # ~50 scripts — parameter mining
│   ├── run_v8*.py             # ~25 scripts — signal composition
│   ├── run_v9*.py             # 6 scripts — production spec
│   ├── run_v10*.py            # ~30 scripts — rebuild + breakthrough
│   ├── run_v11*.py            # 3 scripts — latest research
│   ├── models/                # 149 JSON result files
│   └── NQ_PLAYBOOK.md         # Technical reference (see separately)
├── nautilus-bt/               # NautilusTrader event-driven backtester
├── results/                   # Parameter sweep results
│   └── FINAL_SUMMARY.md       # 87-config sweep findings
├── databento/                 # Raw tick data (318 contract-months)
└── trading_operator/          # Live trading infrastructure
    └── data2/                 # Databento tick data (312 days)
```

---

## Key Technical Bugs Encountered

| Bug | Impact | Fix |
|-----|--------|-----|
| Multi-TF alignment look-ahead (V10o) | Invalidated 14 versions | Use completed bars only |
| Intra-bar OHLC ordering (V10z) | Inflated 3 signals | Conservative worst-case ordering |
| numpy bool as string in JSON | Silent filter failures | `t[key] = t[key] == 'True'` |
| Trade list truncation (V8u) | Lost 1490 of 1690 trades | Save all trades, not `[:200]` |
| Databento tz-aware timestamps | Comparison crashes | Strip tz for naive comparisons |
| operator/ module shadowing | Import failures | Run from /tmp or rename directory |
| extended_5m no end-time cap | Future candles in targets | Cap by entry time |

---

## What's Next

- **V11**: Overnight session mining (12-phase deep dive completed), TF cross validation
- **Live deployment**: Go-live checklist in `go_live.sh`, requires 15S bar infrastructure
- **NautilusTrader integration**: Event-driven execution with real tick data
- **Risk scaling**: Start at 1 MNQ contract, scale based on realized Sharpe

---

*This document covers the complete research arc. For technical signal specifications, see `context-engine/NQ_PLAYBOOK.md`. For parameter sweep details, see `results/FINAL_SUMMARY.md`.*
