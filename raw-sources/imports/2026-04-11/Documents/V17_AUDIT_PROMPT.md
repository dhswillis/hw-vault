# Audit Request: BOS_FVG 10+pt Tick-Based Backtest

## Context

We have a BOS_FVG (Break of Structure + Fair Value Gap) strategy backtested on 312 days of NQ tick data from Databento (Feb 2025 – Feb 2026). The tick data is at `~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/` — one `.dbn.zst` file per day.

Scripts are now in `~/Documents/`:
- `fvg_10pt_deep_mine.py` — main mining script (3min, far entry, T3R/T5R, all filters)
- `fvg_10pt_audit.py` — P/D look-ahead fix + opposing FVG fix (already run)
- `fvg_tick_sim.py` — tick vs bar simulation comparison
- `fvg_tick_modes.py` — tick-based entry/management mode comparison
- `fvg_entry_modes.py` — bar-based entry mode comparison (edge/mid/far/confirmed)
- `fvg_passthru_audit.py` — fill-bar penetration analysis
- `confirmed_audit.py` — confirmed entry audit for look-ahead
- `fvg_10pt_filters.py` — 30-day filter overlay (first pass)
- `fvg_10pt_sweep.py` — timeframe x trailing x entry depth sweep
- `fvg_entry_tf.py` — confirmed vs edge across timeframes

## What We Already Found

### P/D Look-Ahead: CONFIRMED DEAD
We already ran the P/D audit (`fvg_10pt_audit.py`). Results:

| P/D Method | Aligned avgR | Misaligned avgR | Verdict |
|---|---|---|---|
| **Broken (full day high/low)** | **+0.241** | -0.107 | **LOOK-AHEAD — uses future data** |
| **Rolling (up to signal time)** | +0.031 | +0.009 | **DEAD — no separation** |
| **Prev day range** | +0.061 | -0.014 | **Marginal — OOS collapses (+0.209→-0.116)** |

The entire +0.267 avgR "edge" from P/D was future data. With clean P/D, baseline reverts to +0.015 avgR.

### Bar vs Tick Simulation: Bar Inflates by +0.2R
On 30 days, bar-based sim showed +0.255 avgR, tick-based showed +0.046 avgR. Bar simulation inflates results because:
1. Trailing stop uses bar high/low which overstates favorable excursion
2. Intra-bar ordering is ambiguous on fill bars — bar sim optimistically assumes entry happens before stop

### Surviving Filters (marginal)
- **No stacked FVGs**: +0.052 vs -0.033 avgR, walks forward
- **No unfilled opposing FVGs**: +0.041 vs -0.057 avgR
- **Best clean combo**: prev_day P/D + no_stack = +0.102 avgR, Cal 1.5, but OOS degrades

## What Still Needs Auditing

### 1. Stacked FVG Filter — Is It a Volatility Proxy?
The "no stacked" filter uses `abs(sf['idx'] - fvg['idx']) <= 5` (within 5 bars) and `abs(sf['gl'] - gl) < 20` (within 20pts). Questions:
- Is 20pt threshold just catching high-volatility regimes where stops get blown?
- Test tighter thresholds (5pt, 10pt proximity)
- Correlate stacked count with ATR or realized volatility to see if it's just a vol filter

### 2. Session Classification DST Bug
UTC session boundaries are hardcoded for EST (UTC-5) but don't account for EDT (UTC-4) in summer. Data spans Feb 2025 – Feb 2026 so ~7 months are EDT. NY open is 14:30 UTC in winter, 13:30 UTC in summer. Summer NY AM trades are being classified as `ny_pre`. Fix and re-run session breakdown.

### 3. Walk-Forward Rigor
Current WF is a simple 50/50 time split. Need rolling walk-forward:
- 3-month train / 1-month test, 8+ folds
- Report per-fold avgR for the top filter combos
- Check if any filter combo is consistently positive across folds

### 4. Multiple Comparison Correction
~50 filter combos tested against baseline of +0.015 avgR. Need:
- Bootstrap confidence intervals on top combos (1000 resamples)
- Permutation test: shuffle filter labels, report how often random achieves +0.052 or +0.102 avgR
- How many "positive" filters survive at p < 0.01?

### 5. Fill Rate & Selection Bias
Far edge entry means many signals never fill. Need:
- What % of signals fill?
- Do "easier" setups (smaller FVGs, less volatile) fill more/less often?
- Average time from signal to fill

### 6. Explore Alternative Edges
Since P/D died and remaining filters are marginal (+0.05 avgR), explore:
- **VWAP-based P/D**: above/below developing VWAP at signal time
- **Opening range**: first 30 min high/low as reference range
- **Prior session range**: previous NY session (not calendar day) high/low
- **Volume profile**: is entry near a high-volume node?
- **Time-of-day edge**: are certain 2-hour windows consistently better?
- **FVG age**: does the time since FVG formation matter?

### 7. Is 10+pt the Right Size?
The initial size sweep on tick data showed 2-5pt FVGs had the most R/day with trailing stops. Is 10+pt actually optimal for static targets, or would 5-10pt be better? Run the tick sim with 5+pt FVGs and compare.

### 8. Trailing vs Static on Tick Data
The earlier sweep showed trailing (0.5R trigger, 0.1R buffer) at +0.061 avgR on ALL FVGs, but static far T3R at +0.391 on 10+pt. The trailing number was across all sizes. Run trailing on 10+pt only and compare properly against static T3R.

## Data Access
```python
import databento as db
from pathlib import Path
DATA_DIR = Path.home() / "trading_operator/data2/GLBX-20260213-YFP9CFN8HF"
files = sorted(DATA_DIR.glob("*.dbn.zst"))
# Each file: store = db.DBNStore.from_file(fp); df = store.to_df()
# Columns: ts_event (UTC timestamp), price, size, symbol
```

## Python Environment
Use: `/Users/harrisonwillis/Documents/trading-system/.venv/bin/python`

## Output
Save results to `/Users/harrisonwillis/Documents/trading-system/results/BOS_FVG_10PT_AUDIT.md`.
