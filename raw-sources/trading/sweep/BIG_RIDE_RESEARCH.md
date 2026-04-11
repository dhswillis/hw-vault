# Big Ride Research — Catching the HOD/LOD Early and Riding All Day

**Date**: March 3, 2026
**Data**: 299 trading days of NQ tick data (Feb 2025 – Feb 2026, 258 with valid RTH)
**Goal**: Find entry techniques with tight stops that catch the high/low of day early and ride with a wide trailing stop to EOD.

---

## Phase 0: When Is the HOD/LOD Put In?

Analyzed 297 full 24H sessions (17:00 CST open) to find when the daily extreme is established.

### Key Findings
- **U-shaped / bathtub curve** — H/L overwhelmingly made at the session open or the close
- **30% of both highs and lows** are put in during the first 30 minutes of globex (17:00–17:30 CST)
- **Three clusters**: Globex open (17:00–18:00, ~20%), Pre-RTH/RTH open (07:30–09:00, ~15%), RTH close (13:30–15:30, ~20%)
- **Dead zone**: 19:00–07:00 CST, each 30-min bucket only 0–2%

### 50% Milestones (full session)
| Measure | High of Day | Low of Day |
|---------|-------------|------------|
| 50% settled by | 08:00 CST | 07:30 CST |
| 80% settled by | 13:30 CST | 13:00 CST |
| 90% settled by | 14:00 CST | 14:00 CST |

**Chart**: `/tmp/nq_hl_bell_curve.png`, also on Desktop

---

## Phase 1: Initial Entry Screening (Wide Stops)

Tested 5 entry techniques with 80pt stop and 30/40/50/75pt trail across 299 days. One position per signal type per day allowed (multi-position).

### Entry Techniques Tested
1. **open_crossback** — Price sweeps past RTH open, crosses back → fade
2. **overnight_fade** — Price sweeps overnight H/L after RTH open → fade
3. **ib_fade** — First 30-min IB range, fade sweep of IB H/L
4. **vwap_cross** — After initial VWAP departure (15pt+), fade the cross back
5. **first5m_wick** — First 5M candle big wick → limit at wick midpoint

### Results (50pt trail, 80pt stop)

| Signal | N | WR | AvgPts | TotPts | Pts/day |
|--------|---|-----|--------|--------|---------|
| open_cross_short | 224 | 48.7% | +7.9 | +1,777 | +5.9 |
| open_cross_long | 227 | 45.8% | +5.9 | +1,338 | +4.5 |
| overnight_short | 109 | 31.2% | +5.1 | +552 | +1.8 |
| overnight_long | 97 | 25.8% | +4.3 | +414 | +1.4 |
| vwap_cross_short | 121 | 64.5% | +8.6 | +1,045 | +3.5 |
| vwap_cross_long | 130 | 56.9% | -0.3 | -35 | -0.1 |
| ib_fade | — | — | — | DEAD | — |

**Wider trail = more total pts**: 75pt trail → +17.1 pts/day total vs 30pt at +10.2.

### Conclusions
- **Open crossback is the best level-based signal** — especially shorts
- **IB fade is dead** — negative or flat everywhere
- **VWAP cross long is dead** — short side only
- Wide stops work but question was: can we get tighter stops?

---

## Phase 2: Tight Stop Exploration (2–10pt stops)

Tested the level-based entries with 2, 3, 5, 7, 10pt stops. All with 50pt trail.

### Results Summary
- With 2–7pt stops, level-based entries (open_cross, overnight, prev_close) get stopped 85–97% of the time
- **Gap fade short on gap-up days** with 2pt stop was the best lottery ticket: Cal 1.9
- **Open crossback is dead with tight stops** — 96–97% stop rate kills it

**Conclusion**: Level-based entries don't provide enough precision for tight stops. Need structural entries.

---

## Phase 3: Structural Entry Signals (THE BREAKTHROUGH)

Added structural/price-action entries to the scanner alongside the level entries. All tested across stop sizes 5, 10, 15, 20, 25, 30, 40, 50pt with 50pt trail.

### New Entry Techniques
1. **fvg_5m** — 5M Fair Value Gap: 3 consecutive 5M bars where bar[i].low > bar[i-2].high (bullish) or bar[i].high < bar[i-2].low (bearish). Enter at FVG edge. All 3 bars fully closed before detection — NO look-ahead.
2. **sweep_wick_15m** — 15M candle sweeps prior bar's high/low with a wick (≥3pt), closes back inside prior range. Enter at body edge of the sweep candle.
3. **bpr_5m** — 3+ consecutive directional 5M bars followed by reversal bar. Fade the run.
4. **engulf_1m** — 1M engulfing candle near VWAP or open (within 10pts).

### Heat Map: Total Pts / Calmar by Signal × Stop Size

| Signal | s5 | s10 | s20 | s30 | s50 |
|--------|-----|-----|-----|-----|-----|
| **fvg_5m** | **+17,679 / Cal 321** | **+17,396 / Cal 193** | **+17,560 / Cal 125** | **+17,576 / Cal 84** | **+17,638 / Cal 111** |
| **sweep_wick_15m** | **+7,691 / Cal 77** | **+7,748 / Cal 33** | **+7,190 / Cal 20** | **+7,676 / Cal 25** | **+8,724 / Cal 26** |
| engulf_1m | +299 / Cal 1.2 | +209 / Cal 0.5 | +1,062 / Cal 1.5 | +1,124 / Cal 1.3 | +835 / Cal 0.9 |
| open_cross | -206 | -533 | -340 | +138 | +338 |
| overnight_fade | -211 | -588 | -458 | -342 | -335 |
| bpr_5m | -508 | -638 | -718 | -1,185 | -1,415 |

**FVG 5M is the monster.** 433 trades, 66% WR at 5pt stop, +40.8 avg pts, +59 pts/day.

**Sweep Wick 15M is the second winner.** 473 trades, 33% WR at 5pt stop, +16.3 avg pts, +26 pts/day.

**BPR is dead.** Negative everywhere.

---

## Phase 4: Full Audit of FVG 5M + Sweep Wick 15M

Comprehensive audit with walk-forward, slippage, delay, monthly, and MFE analysis.

### 4.1 Walk-Forward Validation

**FVG 5M (s10, trail50, no slip):**

| Period | N | WR | AvgPts | TotPts | Pts/day | Calmar |
|--------|---|-----|--------|--------|---------|--------|
| Q1 (IS) | 111 | 71.2% | +42.4 | +4,710 | +63.6 | 78.5 |
| **Q2 (OOS)** | 100 | 56.0% | +29.6 | +2,958 | +39.4 | **42.3** |
| **Q3 (OOS)** | 109 | 62.4% | +34.4 | +3,746 | +50.0 | **41.6** |
| **Q4 (OOS)** | 113 | 83.2% | +52.9 | +5,982 | +79.8 | **199.4** |
| H1 (IS) | 211 | 64.0% | +36.3 | +7,668 | +51.5 | 109.5 |
| **H2 (OOS)** | 222 | 73.0% | +43.8 | +9,729 | +64.9 | **108.1** |

**ALL 4 quarters OOS positive. Signal gets STRONGER in the second half.**

**Sweep Wick 15M (s10, trail50, no slip):**

| Period | N | WR | AvgPts | TotPts | Pts/day | Calmar |
|--------|---|-----|--------|--------|---------|--------|
| Q1 (IS) | 116 | 45.7% | +21.1 | +2,448 | +33.1 | 25.5 |
| **Q2 (OOS)** | 113 | 25.7% | +6.2 | +695 | +9.3 | **3.0** |
| **Q3 (OOS)** | 125 | 38.4% | +17.8 | +2,220 | +29.6 | **26.9** |
| **Q4 (OOS)** | 119 | 42.9% | +20.0 | +2,384 | +31.8 | **29.8** |

All 4 quarters OOS positive. Q2 weak (Cal 3.0) but still positive.

### 4.2 Slippage Stress Test (FVG 5M, s10)

| Slippage | TotPts | Calmar | Impact |
|----------|--------|--------|--------|
| 0pt | +17,396 | 193.3 | baseline |
| 1pt | +16,965 | 188.5 | -2.5% |
| 2pt | +16,609 | 151.0 | -4.5% |
| 3pt | +16,172 | 147.0 | -7.0% |
| **5pt** | **+15,391** | **139.9** | **-11.5%** |

**Survives 5pt slippage with Calmar 140.** Barely dents it.

### 4.3 Delay Test (FVG 5M, s10)

| Delay | TotPts | Calmar | N |
|-------|--------|--------|---|
| 0 bars | +17,396 | 193.3 | 433 |
| **1 bar (5 min)** | **+17,998** | **200.0** | 426 |
| **2 bars (10 min)** | **+17,996** | **200.0** | 420 |

**Delaying 5–10 minutes does NOT hurt.** Actually slightly improves results. This confirms the edge is a big all-day ride, not a scalp. No speed advantage needed.

### 4.4 Trail Size Comparison (FVG 5M, s10)

| Trail | WR | TotPts | Calmar |
|-------|-----|--------|--------|
| 30pt | 84.8% | +17,170 | **446** |
| 50pt | 68.6% | +17,396 | 193 |
| 75pt | 54.3% | +17,688 | 182 |
| 100pt | 46.0% | +16,379 | 182 |

30pt trail has highest Calmar (446) due to tiny drawdown. Total pts are similar across all trail sizes because the core edge is the entry, not the exit management.

### 4.5 Monthly Breakdown (FVG 5M, s10)

**Every single month positive across 13 months:**

| Month | N | WR | Total | Avg |
|-------|---|-----|-------|-----|
| 2025-02 | 22 | 59.1% | +1,087 | +49.4 |
| 2025-03 | 41 | 65.9% | +1,222 | +29.8 |
| 2025-04 | 36 | 86.1% | +2,004 | +55.7 |
| 2025-05 | 31 | 67.7% | +1,215 | +39.2 |
| 2025-06 | 35 | 54.3% | +1,061 | +30.3 |
| 2025-07 | 40 | 47.5% | +528 | +13.2 |
| 2025-08 | 33 | 66.7% | +1,227 | +37.2 |
| 2025-09 | 35 | 42.9% | +760 | +21.7 |
| 2025-10 | 39 | 71.8% | +1,740 | +44.6 |
| 2025-11 | 33 | 84.8% | +1,964 | +59.5 |
| 2025-12 | 39 | 84.6% | +1,758 | +45.1 |
| 2026-01 | 36 | 80.6% | +1,825 | +50.7 |
| 2026-02 | 13 | 92.3% | +1,006 | +77.4 |

Worst month: Jul 2025 (+528 pts, 47.5% WR). Still +13.2 avg.

### 4.6 Direction Split (FVG 5M, s10)

| Direction | N | WR | AvgPts | TotPts | Calmar |
|-----------|---|-----|--------|--------|--------|
| Long | 225 | 67.1% | +36.7 | +8,252 | 84.2 |
| Short | 208 | 70.2% | +44.0 | +9,144 | 152.4 |

Both directions profitable. Shorts slightly stronger.

### 4.7 MFE Distribution (FVG 5M, s10)

| MFE Range | Count | % |
|-----------|-------|---|
| 0–10 pts | 2 | 0.5% |
| 10–30 pts | 64 | 14.8% |
| 30–50 pts | 72 | 16.6% |
| **50–100 pts** | **152** | **35.1%** |
| **100–150 pts** | **88** | **20.3%** |
| **150–200 pts** | **35** | **8.1%** |
| 200+ pts | 20 | 4.6% |

**55% of trades reach 50+ pts MFE. 33% reach 100+ pts.** These are genuine all-day rides.

Average MAE only **3.5 pts** at 10pt stop — the FVG edge holds tight.

### 4.8 Sample Trades (FVG 5M, s10)

```
20250212 long  entry=21595.00 exit=21651.00 pts=+56.00  mfe=106.0 mae=0.0  fvg=10.0 trail
20250212 short entry=21669.25 exit=21630.50 pts=+38.75  mfe=88.8  mae=0.0  fvg=35.2 trail
20250213 long  entry=21894.75 exit=21884.75 pts=-10.00  mfe=44.2  mae=10.0 fvg=18.8 stop
20250213 short entry=22017.50 exit=21992.00 pts=+25.50  mfe=75.5  mae=0.0  fvg=12.5 trail
```

Note: even stopped trades had significant MFE (30–44 pts) — they went in the right direction first but came back to the stop. The 10pt stop is tight enough that some winners get shaken out, which is the tradeoff for the insane Calmar ratio.

---

## Phase 5: LOOK-AHEAD BUG FOUND — FVG 5M and Sweep Wick 15M INVALIDATED

### The Bug (same class as V10o alignment bug)

**FVG 5M**: The V3/audit entered at the FVG edge price (e.g. `fvg_top` for bearish short). But at the time we detect the FVG (bar[i] close), price has ALREADY GAPPED PAST the FVG edge. You cannot sell at `fvg_top` when the market is at `bar[i].close` which is below `fvg_top`. Every trade got free embedded profit equal to the FVG size (~10-15 pts average).

**Sweep Wick 15M**: Entered at `body_top` (candle open for bearish) but we only know it's a sweep candle at bar CLOSE. By then the open price is in the past. Same issue — entering at a price no longer available.

### Corrected Results — FVG 5M (Phase 5a: Sniper Test)

| Entry Method | N | Fill% | WR | Tot Pts | Calmar | Verdict |
|-------------|---|-------|-----|---------|--------|---------|
| Market at bar close, 10pt stop | 433 | 100% | 12.9% | **-934** | -1.0 | **DEAD** |
| Market at bar close, 5pt stop | 433 | 100% | 6.7% | **-578** | -0.9 | **DEAD** |
| Limit at FVG edge (wait 15 min) | 167 | 39% | 3.0% | **-166** | -0.9 | **DEAD** |
| Limit at FVG midpoint (wait 15 min) | 218 | 50% | 17.0% | **+500** | 0.9 | Marginal |
| Limit at FVG far edge (wait 15 min) | 269 | 62% | 25.7% | **+253** | 0.3 | Marginal |
| 15S rejection candle in FVG zone | 162 | — | 8.0% | **-50** | -0.2 | **DEAD** |
| 1M engulfing at FVG edge | 27 | — | 11.1% | +73 | 0.9 | Too few trades |

**The +17,000 pts was an artifact. Correct FVG 5M edge is approximately break-even.**

### Corrected Results — Sweep Wick 15M (Phase 5b)

| Entry Method | N | WR | Tot Pts | Calmar | Verdict |
|-------------|---|-----|---------|--------|---------|
| Original (body_top, 5pt stop, 30pt trail) | 473 | 44.6% | +8,090 | 116.4 | LOOK-AHEAD |
| **Corrected (bar close, 5pt stop, 30pt trail)** | 473 | 11.8% | **-221** | -0.5 | **DEAD** |
| **Corrected (bar close, 10pt stop, 30pt trail)** | 473 | 21.1% | **-581** | -0.7 | **DEAD** |
| **Corrected (bar close, 20pt stop, 30pt trail)** | 473 | 35.1% | **-880** | -0.8 | **DEAD** |
| **Corrected (bar close, 30pt stop, 50pt trail)** | 473 | 34.9% | **-1,133** | -0.7 | **DEAD** |
| Next bar entry, 5pt stop, 50pt trail | 473 | 8.9% | -14 | -0.0 | **DEAD** |
| Next bar entry, wick stop, 30pt trail | 473 | 59.0% | +1,004 | 1.5 | Marginal |

**ALL corrected bar-close entries are negative.** The "edge" was entirely the distance between the candle's body edge (where we entered) and the close (where we should have entered). Not tradeable.

---

## Summary of Dead Ends (UPDATED)

| Signal | Result | Notes |
|--------|--------|-------|
| **FVG 5M** | **DEAD (LOOK-AHEAD)** | Entry at FVG edge not achievable at detection time. Market entry = negative. |
| **Sweep Wick 15M** | **DEAD (LOOK-AHEAD)** | Entry at body edge not achievable at bar close. Corrected = negative. |
| IB fade (30min range) | DEAD | Negative everywhere |
| BPR 5M (price run fade) | DEAD | -500 to -1,400 pts at every stop |
| VWAP cross long | DEAD | Short side only has edge |
| Open crossback (tight stop) | DEAD | 96%+ stop rate at 2–10pt stops |
| Overnight fade (tight stop) | DEAD | Same problem, no precision |
| 15S rejection at FVG | DEAD | -50 pts |
| 1M engulfing at FVG | DEAD | Too few trades (27) |

---

## What Still Works (from Phase 1 — wide stops only)

The Phase 1 results used MARKET entries at crossback detection time and are clean:

| Signal | Stop | Trail | N | WR | Pts/day | Calmar |
|--------|------|-------|---|-----|---------|--------|
| open_cross_short | 80pt | 50pt | 224 | 48.7% | +5.9 | 3.6 |
| open_cross_long | 80pt | 50pt | 227 | 45.8% | +4.5 | 3.6 |
| vwap_cross_short | 80pt | 50pt | 121 | 64.5% | +3.5 | 2.2 |
| overnight_short | 80pt | 50pt | 109 | 31.2% | +1.8 | 1.3 |

These are real but require wide stops (80pt) making them more of a positioning/swing concept than a tight-stop sniper entry. Combined ~+15 pts/day with 50pt trail.

---

## Key Lesson

**Any entry that uses a price from the PAST of a candle (open, body edge, wick) is suspect if you only know the signal at the candle CLOSE.** This is the same class of bug as V10o multi-TF alignment. The test is: at the exact moment you detect the signal, is the entry price achievable at market? If not, it's look-ahead.

---

## Scripts

| Script | Purpose |
|--------|---------|
| `/tmp/hl_time_distribution.py` | RTH-only H/L time analysis |
| `/tmp/hl_time_full.py` | Full 24H session H/L timing |
| `/tmp/hl_bell_curve.py` | Smoothed distribution chart |
| `/tmp/bigride_miner.py` | Phase 1: initial entry screening |
| `/tmp/bigride_v2.py` | Phase 2: tight stop exploration |
| `/tmp/bigride_v3.py` | Phase 3: structural entries (FVG, sweep wick, BPR, engulfing) |
| `/tmp/bigride_audit.py` | Phase 4: full audit (walk-forward, slippage, delay, monthly) |

## Data Files

| File | Contents |
|------|----------|
| `/tmp/hl_full_session_data.json` | H/L time distribution raw data |
| `/tmp/bigride_v3_results.json` | Phase 3 ranked results |
| `/tmp/bigride_audit.json` | Phase 4 audit trade-level data |
| `/tmp/nq_hl_bell_curve.png` | H/L time distribution chart |
