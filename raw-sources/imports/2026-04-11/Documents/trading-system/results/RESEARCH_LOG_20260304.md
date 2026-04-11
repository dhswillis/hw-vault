# Research Log — March 4, 2026 (Overnight Session)

## Study 1: Wick Reversion Scalper

### Quick (17-day) Results
- **Small target wick fades are DEAD** — 1.5pt target / 10pt stop needs 91.3% WR to break even, best achieved was 87.2%
- The R math kills it: tiny wins can't overcome rare but huge losses

### Wide Structure Sweep — 312-Day FULL RESULTS
- **Best config**: midpoint tgt=20 stop=40 sweep=1.0 → 3025 trades, 67.4% WR, **+0.29 R/day**, Cal 2.7
- Quick test was +1.47 R/day — degraded 5x on full year (quick period was outlier)
- **Midpoint entry dominates** — all top 8 configs
- **Hour filter opportunity**: 1PM ET (18 UTC) is -34.2R, all others positive. Excluding 1PM → ~+0.42 R/day
- **3PM ET close** is strongest: +42.9R from 302 trades (WR 62.3%)
- **Your 20pt/60pt combo**: 73.9% WR, +0.10 R/day, Cal 0.9 — works but tighter stop (40pt) is better
- **Verdict: MARGINAL EDGE** — +0.29 R/day / 75R per year. Not enough for standalone strategy.
- Could combine with other signals for portfolio diversification

## Study 2: Candle Fade Stop-Market — DEAD
### 1M Quick (17 days)
- Best config: body>=5pt, risk<=15pt, short-only → 91 trades, 45.1% WR, +0.17 R/day
### 5M Full (258 days) — CONFIRMED DEAD
- **Every config negative or flat** — best is 0.00 R/day with 33 trades/year
- Fading candle direction with stop-market at opposite side has ZERO edge on NQ
- Adding to confirmed dead ends list

## Study 3: 9:00 AM Candle Behavior (258 trading days)

### Key Findings:
1. **Direction by DOW**: Near 50/50 every day. Friday slightly bearish (47% bull). No tradeable DOW bias.
2. **8:00 bar → 9:00**: Bullish 8:00 → 48% bull 9:00. Bearish 8:00 → 58% bull 9:00. **Slight REVERSAL tendency after bearish 8:00.**
3. **8:30 bar → 9:00**: Bullish 8:30 → 56% bull 9:00. Bearish 8:30 → 50%. **Slight continuation after bullish 8:30.**
4. **Combined 8:00 + 8:30**:
   - Bear 8:00 + Bull 8:30 → **61.8% bull 9:00** (76 samples) — BEST SIGNAL
   - Bull 8:00 + Bear 8:30 → 54.1% bear 9:00
   - Both bull or both bear → ~50/50
5. **9:00-9:30 range**: Mean 91.7 pts, median 69.5 pts (huge variance)
6. **High first vs Low first**: Exactly 50/50 overall. No DOW pattern.
7. **Continuation rate 8:30→9:00**: 53.1% overall — barely above coin flip
8. **Large 8:30 bars**: Same continuation rate as small ones (52.7% vs 53.5%)

### Actionable Signal:
- Bear 8:00 + Bull 8:30 → 62% chance 9:00 is bullish, avg range 104.6 pts
- This is 76 occurrences out of 258 days (29% of days)
- Need to test: can we enter long at 9:00 with tight stop, target 20+ pts?

## Study 4: Open Fakeout (9:00 AM + 9:30 RTH) — 312 days

### 128 configs tested: market fade + stop-opposite entries, with/without 8:00+8:30 conditioning

**Best: 9:00 market fade, tgt=20 stp=30, bear8+bull830 → 76 trades, 71.1% WR, +0.04 R/day, Cal 2.8**
- Conditioning works (71% WR) but only 76 trades/year — too rare
- Unconditioned 9:00/9:30 fades are flat (+0.01 R/day)
- 9:30 RTH open fakeout: no edge at any config
- **Verdict: DEAD for standalone use** — the 62% bull probability from Study 3 is real but not large enough for R-positive trading after commissions

## OVERNIGHT SESSION SUMMARY (March 4-5, 2026)

### Confirmed Dead:
1. **Small-target wick fades (1M)** — R math impossible (need 91%+ WR)
2. **Candle fade stop-market (1M + 5M)** — zero edge at any config
3. **9:00/9:30 open fakeout** — flat R/day, conditioning too rare
4. **8:00/8:30 → 9:00 direction prediction** — max 62% accuracy, not enough for trading

### Marginal Edge Found:
5. **5M wick reversion, midpoint entry, 20pt target / 40pt stop** — +0.29 R/day, Cal 2.7, 67% WR
   - 1PM ET hour is poison (-34R), excluding it improves to ~+0.42 R/day
   - **Could add 0.3-0.4 R/day to portfolio as supplementary signal**
   - Needs walk-forward validation before any production consideration

---

## Study 5: FVG Reaction — Unmitigated Zone Play (March 5, 2026)

### Concept:
- Detect FVGs on 5M/15M timeframes, track unmitigated zones
- When price returns to zone, enter on 15s or 1M rejection candle (or touch)
- Stop below zone + buffer, target at R multiples
- 480 configs tested: 2 FVG TFs × 2 entry TFs × 5 R targets × 4 min FVG sizes × 2 entry modes × 3 stop buffers

### Quick (25-day) Results:
- **5M FVG + 1M rejection dominated** — all top 10 configs
- Best: fvg=5min entry=1min R=2.0 gap>=1 rej buf=0.5 → 26 trades, 50% WR, **+0.46 R/day**
- R=1.0 with rejection: 69% WR — highest win rate configs

### Full 312-Day Results:
- **Best config**: fvg=5min entry=15s R=2.0 gap>=5 rej buf=2.0 → 556 trades, 38.8% WR, **+0.22 R/day**, Cal 3.9
- Quick test was +0.46 → degraded to +0.22 (same pattern as wick reversion)
- **15s entry now beats 1M** (quick test had 1M dominating — overfitting on small sample)
- **Larger FVGs (gap>=5 pts) more reliable** than small ones
- **9AM ET hour (+31.6R from 280 trades)** is where half the edge lives
- **10AM ET slightly negative** — could filter
- **Rejection entry > touch** for most configs
- **15M FVG + 15s touch** at R=1.0 gives highest WR (57.1%) but only +0.16 R/day
- **Verdict: MARGINAL EDGE** — +0.22 R/day / ~69R per year. Not standalone-worthy.

### Cross-Study Pattern:
- Every concept tested degrades 3-5x from quick to full year
- Best R/day across all studies: wick reversion +0.29, FVG reaction +0.22
- Neither is strong enough alone. Combined they might add +0.4 R/day to a portfolio.
- The search continues for a higher-edge signal.

---

## UPDATED SESSION SUMMARY (March 4-5, 2026)

### Confirmed Dead:
1. **Small-target wick fades (1M)** — R math impossible (need 91%+ WR)
2. **Candle fade stop-market (1M + 5M)** — zero edge at any config
3. **9:00/9:30 open fakeout** — flat R/day, conditioning too rare
4. **8:00/8:30 → 9:00 direction prediction** — max 62% accuracy, not enough for trading

### Marginal Edges Found:
5. **5M wick reversion, midpoint entry, 20pt/40pt** — +0.29 R/day, Cal 2.7, 67% WR
6. **5M FVG reaction, 15s rejection, R=2.0, gap>=5** — +0.22 R/day, Cal 3.9, 39% WR

### Scripts Built:
- `/tmp/wick_reversion_scalper.py` — full 8-phase backtester (1M + 5M)
- `/tmp/candle_fade_stop.py` — candle direction fade backtester
- `/tmp/nine_am_candle_study.py` — 9AM behavioral analysis
- `/tmp/open_fakeout_bt.py` — open fakeout backtester
- `/tmp/fvg_reaction_bt.py` — FVG reaction backtester (unmitigated zones)

### Data saved:
- `results/wick_reversion/full_312d_5min_structural.json`
- `results/candle_fade/candle_fade_5min.json`
- `results/open_fakeout/fakeout_results.json`
- `results/nine_am_study/nine_am_data.csv`
- `results/fvg_reaction/fvg_reaction_results.json`
