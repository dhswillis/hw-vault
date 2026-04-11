# SF Portfolio — Backtest Issues & Audit Log
**Last Updated**: 2026-03-19

---

## Issue #1: Engulfing Look-Ahead Bias (CRITICAL)
**Date found**: 2026-03-18
**Affects**: ALL Python backtests using `sf_scan_window()` from `leppyrd_v4.py`

### The Bug
`sf_scan_window()` (lines 97-98) uses `cc['bd']` and `cc['body']` — the current candle's FINAL close direction and body size — to gate entries that happen DURING that candle. Both `eng_req=True` and `eng_req=False` are affected.

### Impact
| Metric | Buggy (260d) | Fixed (260d) |
|--------|-------------|-------------|
| Trades | 1066 | 623 |
| Win Rate | 77% | 40% |
| Net P&L | +18,328 pts | -1,092 pts |
| Losing Weeks | 0/54 | 27/54 |

### Fix: DIR scan — only check completed prior bar direction. No current bar data.

---

## Issue #2: Premature "No Edge" Assessment (CORRECTED)
**Date**: 2026-03-18
**Error**: After finding the engulfing bug, I declared "the sweep-fail concept has no edge" and "the strategy is net negative." This was WRONG.

### What Actually Works (260d, DIR scan, zero look-ahead)
The V21 portfolio is NOT trash. Multiple legs are genuinely profitable:

| Leg | T/S | Trades | WR | PF | R/Day | Status |
|-----|-----|-------:|---:|---:|------:|--------|
| NY1_15M S10 5R | 50/10 | 201 | 21% | 1.24 | +0.152 | REAL |
| NY1_5M S10 BE5 | run/10 | 258 | 10% | 1.15 | +0.130 | REAL |
| NY1_15M S10 3R | 30/10 | 201 | 30% | 1.22 | +0.127 | REAL |
| PRE_PDH S30 BE5 | run/30 | 207 | 22% | 1.19 | +0.119 | REAL |
| ALATE_SF S10 2R | 20/10 | 257 | 39% | 1.18 | +0.116 | REAL |
| PRE_PDH S30 1.5R | 45/30 | 207 | 45% | 1.20 | +0.089 | REAL |
| NY1_15M S30 BE5 | run/30 | 201 | 25% | 1.16 | +0.088 | REAL |
| NY1_15M S20 1.5R | 30/20 | 201 | 45% | 1.16 | +0.070 | REAL |
| NLATE_15M S30 2R | 60/30 | 251 | 35% | 1.03 | +0.017 | MARGINAL |
| LON_SF S10 BE5 | run/10 | 250 | 20% | 1.05 | +0.038 | MARGINAL |

Non-overlapping V21 portfolio: **+0.44 R/Day** on 260 days.

---

## Issue #3: H4 Bias and AMANIP Filter Look-Ahead
**Date found**: 2026-03-18

### The Bug
- `h4_bias` uses 8:00-12:00 UTC candle close → look-ahead for LON/ALATE windows
- `asia_manip` uses full 0:00-8:00 data → look-ahead for ALATE window

### Rule
- BIAS_H4/H4SW: only valid for windows starting >= 720 (PRE, NY1+)
- AMANIP: only valid for windows starting >= 480 (LON+)
- BIAS and PDH_SW: clean for ALL windows (prior-day data only)

---

## Issue #4: Sign Error in Dynamic Stop Script
**Date found**: 2026-03-18
**Script**: `/tmp/sf_dynamic_trail_260d.py`
**Bug**: Short stop-out formula `pnl = d * (ep - trail_stop)` returned POSITIVE instead of NEGATIVE. Short stops counted as wins.
**Impact**: +16.4 R/day result was fake. Stress test with correct formula showed all configs negative.

---

## Issue #5: Keylevel Sweep Fade Swing Confirmation Timing
**Date found**: 2026-03-18
**Bug**: `find_swing_points` used bar START time as confirmation, not bar END time. 5 minutes of look-ahead.
**Impact**: +10.43 R/day dropped to -0.16 R/day with fix.

---

## Issue #6: SMT Filter Look-Ahead (found by cowork session)
**Date found**: 2026-03-18
**Bug**: SMT computed from ALL bars before trading loop. Afternoon divergence used for morning trades.
**Fix**: SMT must be incremental — only use bars completed before current bar.

---

## FULL COMBINED PORTFOLIO — All Verified, 260 Days

### Mega Portfolio Legs (BE5_FLAT runners)
| Leg | Concept | Trades | WR | PF | R/Day |
|-----|---------|-------:|---:|---:|------:|
| 1H_LON_PDH | 1H SF London, PDH_SW | 132 | 11% | 3.03 | +0.88 |
| ASIA_SW | Asia range sweep reversal | 395 | 8% | 1.60 | +0.75 |
| 15M_PRE_PDH | 15M SF PRE, PDH_SW | 229 | 4% | 1.50 | +0.40 |
| 1H_ASIA_PDH | 1H SF Asia, PDH_SW | 218 | 6% | 1.47 | +0.37 |
| 1H_NL_BIAS | 1H SF NLATE, bias | 124 | 7% | 1.31 | +0.13 |
| GAP_FILL | Fade overnight gap | 179 | 35% | 1.19 | +0.08 |
| 15M_NY_NONE | 15M SF NY, no filter | 258 | 18% | 1.07 | +0.05 |
| 15M_LON_BIAS | 15M SF LON, bias | 235 | 37% | 1.08 | +0.04 |
| **Subtotal** | | | | | **+2.69** |

### V21 Fixed-Target Legs (non-overlapping additions)
| Leg | Trades | WR | PF | R/Day |
|-----|-------:|---:|---:|------:|
| NY1_15M S10 5R | 201 | 21% | 1.24 | +0.152 |
| ALATE_SF S10 2R | 257 | 39% | 1.18 | +0.116 |
| NLATE_15M S30 2R | 251 | 35% | 1.03 | +0.017 |
| **Subtotal** | | | | **+0.29** |

### Cowork-Verified Additional
| Leg | Trades | WR | PF | R/Day |
|-----|-------:|---:|---:|------:|
| BOS_NY1 (FVG limit fill) | 89 | ~20% | 1.64 | +0.18 |

### GRAND TOTAL (non-overlapping): ~3.16 R/Day

Note: Some legs trade the same windows (e.g., 15M_PRE_PDH and PRE_PDH are both PRE window). In live trading, pick the better one per window. The realistic non-overlapping total is approximately 3.0-3.2 R/Day.

---

## Issue #7: 1H MTF Confirmation Look-Ahead Bias (CRITICAL)
**Date found**: 2026-03-21
**Script**: `/tmp/mtf_confirm_260d.py`
**Bug**: `get_htf_dir()` at line 92 uses `b['sm'] < entry_minute` — checks if the 1H bar's START time is before entry. A 1H bar starting at 10:00 doesn't close until 11:00. If entry is at 10:15, the code uses the 10:00-11:00 bar's close price = **45 minutes of look-ahead**.

This is the EXACT same class of bug as V10a-V10n alignment (documented in memory as "CRITICAL BUG: Multi-TF Alignment Look-Ahead").

### Impact
| Metric | Buggy (look-ahead) | Expected (clean) |
|--------|--------------------|-----------------:|
| MTF portfolio WR | 60-83% per leg | ~45-50% (same as base) |
| Losing weeks | 7 | ~12+ (same as base) |
| R/Day | +1.03 | ~0.5-0.7 (degraded) |

**The "7 losing weeks" MTF result is FAKE.** The entire WR improvement came from using future 1H bar closes.

### Correct Fix
```python
# WRONG (look-ahead):
if b['sm'] < entry_minute:  # bar STARTED before entry but hasn't CLOSED

# CORRECT:
if b['sm'] + 60 <= entry_minute:  # bar fully CLOSED before entry (60 = 1H TF)
```

Only use 1H bars whose close time (`sm + 60`) is at or before the entry minute. This means for a 10:15 entry, the latest usable 1H bar is the one that closed at 10:00 (started at 9:00).

### Lesson
This is the 3rd time this exact bug class has been introduced (V10a alignment, doji break, now MTF confirmation). **Any time a higher-TF bar is used for filtering, the bar's END time (not start time) must be checked against entry time.**

---

## Issue #8: Late-Day VWAP Fade Uses End-of-Day VWAP (LOOK-AHEAD)
**Date found**: 2026-03-21
**Scripts**: `/tmp/calmar_compare_260d.py`, `/tmp/add_latefade_260d.py`, `/tmp/final_mega_260d.py`
**Signals affected**: NQ LF1170, NQ LF1080, ES LF1170

**Bug**: The late-day VWAP fade signals compute VWAP from ALL RTH ticks (870-1260 min) before checking the entry condition. For LF1170 (entry at 3:30 PM), the code uses end-of-day VWAP which includes 30 minutes of future data (3:30-4:00 PM). For LF1080, it includes 3 hours of future data.

```python
# WRONG: vw is end-of-day VWAP (computed by the VWPM loop which runs all ticks)
if vw is not None:
    for i in range(n):
        if mi[i]>=1170:
            p=pr[i]
            if p>vw+10:  # vw here includes ticks AFTER 1170!
```

**Impact**: The distance-from-VWAP check (`p>vw+10`) uses future information. If price rallies 3:30-4:00, end-of-day VWAP is higher than true 3:30 VWAP, making the signal less likely to trigger short. Conversely if price dumps, EOD VWAP is lower. This biases the signal toward correct entries.

**Fix**: Compute VWAP incrementally up to the entry time:
```python
# CORRECT: compute VWAP only up to entry minute
vp=0;vv=0;vw_at_entry=None
for i in range(n):
    if mi[i]<RTH_START or mi[i]>=RTH_END:continue
    if mi[i]>=1170:  # stop accumulating at entry time
        vw_at_entry=vp/vv if vv>0 else None
        break
    vp+=pr[i];vv+=1
```

**Severity**: Moderate. The VWAP at 3:30 PM and end-of-day VWAP typically differ by 1-5 pts (on a 20,000 pt instrument). The 10pt threshold provides some buffer. But the ES version (3pt threshold) is more affected since the future data has proportionally more impact.

**Re-test results (NQ+ES+Runner full portfolio):**
| Version | R/Day | Calmar | Sharpe | LW |
|---------|------:|-------:|-------:|---:|
| BUGGY (EOD VWAP) | +3.59 | 61.7 | 7.04 | 5 |
| **FIXED (VWAP at entry)** | **+3.34** | **53.1** | **6.57** | **7** |

Impact: -0.25 R/Day, -8.6 Calmar, +2 losing weeks. The edge is real but inflated by ~7%. The fixed version at **Calmar 53.1, Sharpe 6.57, +3.34 R/Day** is still the best portfolio found.

---

## Key Lessons
1. **B/E at 1R kills the edge** — confirmed across all legs. BE5 works because only rare runners get the protection.
2. **PDH_SW is the strongest clean filter** — adds 4-8% WR consistently.
3. **Leppyrd bias works** — multiple legs profit with bias filter on 260 days.
4. **2:1 R:R asymmetry is critical** — symmetric T/S legs are break-even.
5. **1H timeframe produces highest quality signals** — 1H_LON_PDH at PF 3.03 is the single best leg.
6. **V21 legs are NOT dead** — ALATE, NY1, PRE_PDH, NLATE all have real edge with DIR scan.

## Scripts
| Script | Purpose |
|--------|---------|
| `/tmp/mega_portfolio_30d.py` | Mega portfolio builder |
| `/tmp/v21_exact_260d.py` | V21 exact configs |
| `/tmp/v21_rtargets_260d.py` | V21 with various R targets |
| `/tmp/wick_study_and_confluence.py` | NQ wick analysis |
| `/tmp/highwr_260d.py` | High WR + B/E study |
| `python/v25_combined.py` | Cowork combined portfolio |
| `python/v25_bos_fvg.py` | BOS_FVG backtester |
