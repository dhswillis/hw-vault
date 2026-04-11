# BOS_FVG Code Audit — Findings & Script Guide

## How to Run

```bash
cd ~/Documents
/Users/harrisonwillis/Documents/trading-system/.venv/bin/python bos_fvg_full_audit.py
```

Output saved to: `~/Documents/trading-system/results/BOS_FVG_10PT_AUDIT.md`

Runtime: ~20-40 min depending on disk/CPU (loads all 312 days of tick data, runs tick-level sims).

---

## Code Bugs Found During Script Review

### Bug #1: P/D Look-Ahead (CRITICAL — already confirmed dead)

**Files affected**: `fvg_10pt_deep_mine.py` (lines 128-132), `fvg_10pt_filters.py` (lines 145-150)

```python
# BROKEN — uses ENTIRE day's data including future
day_high = candles_3m['high'].max()
day_low = candles_3m['low'].min()
day_mid = (day_high + day_low) / 2
```

The entire +0.267 avgR "edge" from P/D alignment was future data. Confirmed dead by `fvg_10pt_audit.py`. Rolling P/D reverts to +0.031 avgR (no separation). **`fvg_10pt_filters.py` was never patched** — all filter combos involving P/D from that script are invalid.

**Fix in audit script**: P/D removed entirely. Replaced with VWAP-based P/D using developing VWAP up to signal time (no look-ahead).

### Bug #2: Opposing FVG Doesn't Check Unfilled Status

**File**: `fvg_10pt_deep_mine.py` (lines 149-154)

```python
# BROKEN — counts FVGs that have already been traded through
for of in all_fvgs_3m:
    if of['dir'] == d or of['idx'] > fvg['idx']: continue
    if d == 'long' and of['gl'] > entry_level and of['gh'] < target_5r: opposing += 1
```

An opposing FVG that price has already completely traded through is no longer a valid supply/demand zone. The filter counts these "dead" zones as obstacles.

**Fix in audit script**: Added unfilled check — iterates candles between opposing FVG formation and signal time, skips if any candle traded through the FVG.

### Bug #3: DST Session Classification

**File**: `fvg_10pt_deep_mine.py` (lines 178-201)

UTC session boundaries are hardcoded for EST (UTC-5):
- NY Open: 14:30 UTC (correct in winter)
- During EDT (Mar-Nov): NY Open is actually 13:30 UTC
- ~7 months of data have NY AM trades classified as `ny_pre`
- London/Asia boundaries also shift by 1 hour

**Fix in audit script**: Full DST-aware `classify_session_dst()` function that computes EDT transitions per-year and converts to ET before classifying.

### Bug #4: `fvg_10pt_filters.py` Still Has Look-Ahead P/D

This file was written for 30-day filter analysis but still uses the broken P/D code at lines 145-150. Since it was used to declare "surviving filters," any combo involving P/D from its output is tainted.

### Bug #5: Target Path Check Uses 5R Even for T3R Trades

**File**: `fvg_10pt_deep_mine.py` (lines 153-154)

Opposing FVGs are checked against `target_5r` even when simulating T3R trades. This means the opposing FVG filter is too aggressive for T3R (checking a larger zone than necessary) and could be filtering out valid setups.

---

## What the Audit Script Tests

### Audit #1: Stacked FVG Filter — Volatility Proxy?

Tests three proximity thresholds (5pt, 10pt, 20pt) and correlates stacked count with 3-min ATR. If stacked FVGs only appear in high-volatility regimes and the "no stacked" filter just avoids vol, it's not a structural edge.

### Audit #2: DST Session Classification

Compares old (broken) vs new (DST-fixed) session labels. Reports how many trades shift categories and whether session-level performance changes after the fix. Separately reports EDT-only and EST-only results.

### Audit #3: Rolling Walk-Forward

3-month train / 1-month test rolling window across all available months. Reports per-fold OOS avgR for baseline and for the top filter combo (no_stack + no_opposing). A real edge should be positive in >60% of folds.

### Audit #4: Multiple Comparison Correction

- Bootstrap 95% CIs on all filter combos (1000 resamples) — flags if CI contains zero
- Permutation test on top 3 combos — shuffles filter labels 1000 times, reports how often random achieves the observed avgR

### Audit #5: Fill Rate & Selection Bias

Reports what % of 10+pt signals actually fill at the far edge entry. Breaks down by FVG size bucket and correlates with ATR. Also reports fill delay (ticks from signal to entry).

### Audit #6: Alternative Edges

Tests five new contextual filters that don't have look-ahead:
- **VWAP-based P/D**: developing VWAP at signal time (no future data)
- **Opening range**: first 30min RTH high/low as reference (DST-corrected)
- **Prior session range**: previous day high/low midpoint
- **Time-of-day**: ET-hour breakdown
- **FVG age**: bars since formation to fill opportunity

### Audit #7: FVG Size — 5+pt vs 10+pt

Runs the full pipeline on 5+pt FVGs and compares R/day against 10+pt. Also isolates the 5-10pt bucket separately to see if smaller FVGs have a different edge profile.

### Audit #8: Trailing vs Static on 10+pt

Runs trailing stop (0.5R trigger, 0.1R buffer) on 10+pt FVGs at the tick level and compares against static T3R and T5R. This was missing from earlier analysis — the trailing number was only run on all FVG sizes.

---

## What We Already Know Is Dead

| Edge | Status | Evidence |
|---|---|---|
| P/D (full day) | DEAD — look-ahead | `fvg_10pt_audit.py`: aligned=+0.241 vs rolling=+0.031 |
| P/D (rolling) | DEAD — no separation | Rolling aligned=+0.031 vs misaligned=+0.009 |
| P/D (prev day) | DEAD — OOS collapses | IS +0.209 → OOS -0.116 |
| Daily bias (Leppyrd) | DEAD | No separation in BOS_FVG context |
| Bar-based simulation | INFLATED | Bar sim +0.255 vs tick sim +0.046 avgR |
| HTF alignment | DEAD | No consistent edge in prior runs |

## What Might Be Real (needs this audit to confirm)

| Edge | Prior Evidence | Risk |
|---|---|---|
| No stacked FVGs | +0.052 vs -0.033 avgR | May be vol proxy |
| No opposing FVGs | +0.041 vs -0.057 avgR | Bug in unfilled check |
| Best combo (prev_day P/D + no_stack) | +0.102 avgR, Cal 1.5 | P/D part may be noise |
| Time-of-day windows | Some hours >0 | Not walk-forward tested |
