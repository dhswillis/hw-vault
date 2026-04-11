# Tempo V14 Audit — Corrections & Implementation Notes

**Last Updated**: March 20, 2026
**Script**: `trading-system/results/tempo_v14_audit.py`

---

## CRITICAL CORRECTION #1: Signal Detection Sequence

### WRONG (what build spec & V14 C# code implement)
1. Sweep a key level (wick past, close back inside)
2. Wait for NEW FVG to form AFTER the sweep (reversal direction)
3. Wait for candle to close through that new FVG
4. Entry at bar close price

### CORRECT (actual Tempo pattern)
1. Price moves aggressively toward a key level
2. On the way, a 3-bar FVG forms in the direction of the move (momentum gap)
3. A solid bar follows (no stacked gaps — singular FVG)
4. Price continues and sweeps past the key level (wick past, close back inside)
5. Price reverses and closes back through that same pre-existing FVG
6. Entry at the **FVG edge** (not wherever the bar happened to close)
7. Use 30s bars for precise entry detection near the FVG edge

**Key difference**: The FVG forms BEFORE the sweep, not after. The "inversion" is price trading back through an FVG that already existed. The sweep is the catalyst for the reversal.

### Implementation in audit script
- FVGs are tracked continuously on every 1-min bar (not just in the entry window)
- On sweep detection, we look BACK through `recent_bull_fvgs` / `recent_bear_fvgs` for a qualifying FVG
- Sweep of HIGH → look for bullish FVG (gap up) below the level → SHORT setup
- Sweep of LOW → look for bearish FVG (gap down) above the level → LONG setup
- The V14 C# code's post-sweep FVG scan is now a testable quality filter (`has_confirm_fvg`) — not the core entry logic

---

## CRITICAL CORRECTION #2: Stop Placement

### WRONG (what build spec says)
- Stop = sweep wick ± 1 point (fixed distance, often 20-30pts)
- Risk = entry to sweep wick

### CORRECT
- **Soft stop** = opposite IFVG edge (close-based, checked on 1-min bar closes only)
  - LONG: FVG low (close below = thesis invalidated)
  - SHORT: FVG high (close above = thesis invalidated)
- **Emergency hard stop** = sweep wick ± 1pt (tick-based, catastrophic safety net only)
- **Risk = gap size** (entry at one edge of the IFVG, stop at the opposite edge)
- After B/E: hard stop at entry price (tick-based, replaces both soft and emergency)

---

## CRITICAL CORRECTION #3: Entry Price

### WRONG
- Entry at whatever price the 1-min bar closed at (often 13+ pts past the FVG edge)

### CORRECT
- Entry at the **FVG edge price** exactly
- Use 30s bars for detection: when a 30s bar closes through the FVG, entry = FVG edge
- For SHORT: entry = FVG low (gL)
- For LONG: entry = FVG high (gH)

---

## Session-Aware Trade Management

| Session | TP1 | TP2 | Runner | B/E Trigger |
|---|---|---|---|---|
| London (pre-8:30 ET) | None (0%) | None | 100% | +17pts |
| NYAM (9:30-11:00 ET) | 30% at +17pts | None | 70% | After TP1 |
| NYLunch (11:00-1:00 ET) | 55% at +17pts | 35% at +35pts | 10% | After TP1 |
| NYPM (1:00+ ET) | 55% at +17pts | 35% at +35pts | 10% | After TP1 |

---

## Post-Sweep Confirmation FVG (Quality Filter)

The V14 C# code's `IFVGScanGap()` pattern looks for a second FVG forming AFTER the sweep in the reversal direction. This is now tracked as a quality flag `has_confirm_fvg`:

- SHORT setup: after sweep of HIGH, look for a BEARISH FVG (gap down) between sweep bar and entry bar
- LONG setup: after sweep of LOW, look for a BULLISH FVG (gap up) between sweep bar and entry bar

This is tested as a stacked filter in Section 5 of the audit report. It is NOT a gate — just additional confluence to measure.

---

## London 0% WR — Diagnosis

**Observed**: London trades consistently show 0% WR across all test periods.

**Root cause hypothesis**: London session uses 100% runner with no TP1. B/E only activates at +17pts. The soft stop (IFVG edge = gap size) is tight relative to London volatility. Trades that don't immediately run 17+ pts get caught by the soft stop or fail to reach B/E before reversing.

**B/E exits at -0.2 (commission only)** are counted as losses because pnl_pts ≤ 0.01. These are technically neutral, not true losses.

**Potential fixes to test**:
1. Count B/E as neutral (not loss) — changes WR calculation
2. Add TP1 to London (even small, like 20% at +17pts) to get early profit protection
3. Use wider soft stop (e.g., 1.5x gap) for London to handle pre-market volatility
4. Skip London entirely (most discretionary traders avoid pre-market for this reason)

---

## Key Files

| File | Purpose |
|---|---|
| `trading-system/results/tempo_v14_audit.py` | Main tick-level audit script (corrected) |
| `trading-system/results/tempo_v14_tick_audit.csv` | Per-trade output from latest run |
| `trading-system/results/TEMPO_IFVG_RESEARCH.md` | Original research document (V2 bar-level results) |
| `trading-system/results/TEMPO_V14_CORRECTIONS.md` | This file — all corrections documented |
| `strategies/08-keylevel-sweep/keylevel_sweep_fade/TEMPO_IFVG_BUILD_SPEC.md` | Build spec (contains wrong patterns, see corrections above) |
| `strategies/08-keylevel-sweep/keylevel_sweep_fade/TempoPortfoliov14.cs` | V14 C# code (implements wrong sequence, post-sweep FVG used as quality filter) |

---

## Algorithm Parameters (Current Audit Script)

```python
# Entry window
ENTRY_START   = 30000    # 3:00 AM ET (London start)
ENTRY_END     = 153000   # 3:30 PM ET
FLATTEN_TIME  = 155500   # 3:55 PM ET

# FVG filters
GAP_MIN       = 3.0      # Minimum gap size (pts)
GAP_MAX       = 20.0     # Maximum gap size (pts)
MIN_RISK      = 3.0      # Minimum risk (pts)
MAX_RISK      = 60.0     # Maximum risk (pts)
SWEEP_MIN_PTS = 1.0      # Minimum sweep wick past level

# Trade management
TP1_PTS       = 17.0     # Take profit 1
TP2_PTS       = 35.0     # Take profit 2
BE_THRESH_PTS = 17.0     # Break-even threshold
BE_BUFFER_PTS = 0.25     # Buffer above/below entry for B/E
COMMISSION    = 0.5      # Commission per trade (pts)

# Signal detection
MAX_FVG_AGE        = 40   # How far back to look for FVGs when sweep happens
MAX_BARS_CLOSE_BACK = 20  # Bars after sweep to wait for close-back-through
```

---

## Run History

| Date | Range | Days | Trades | WR | Avg Pts | Notes |
|---|---|---|---|---|---|---|
| (previous) | Feb-Feb 2025-26 | 312 | 269 | 74.7% | +9.9 | V2 bar-level, wrong entry logic |
| Mar 19, 2026 | Mar 3-7, 2025 | 5 | 17 | 58.8% | — | Corrected IFVG pattern, FVG edge entry |
| Mar 19, 2026 | Mar 3 - Apr 4, 2025 | 29 | 71 | 40.8% | -0.9/d | 30-day run, London 0% WR (26 trades) dragging total. Excl London: ~62% WR, positive |
| Mar 20, 2026 | Mar 3 - Apr 4, 2025 | 29 | 105 | 70.5% | +24.5/d | US M60 only (NYAM/Lunch/PM + M60 filter). Emergency stop fix applied. |
| Mar 20, 2026 | Mar 3 - Apr 4, 2025 | 29 | 58 | 46.6% | +10.9/d | London fade: TP1=17→BE+1pt, session flatten 8:30 AM. 3 EOD runners carry all PnL. |
| Mar 20, 2026 | Mar 3 - Apr 4, 2025 | 29 | 163 | 62.0% | +31.9/d | **Combined portfolio**: US M60 + London fade. Calmar 4.8. EOD dependence: 4 trades = 139% of total. |

### 30-Day Key Findings (Mar 3 - Apr 4, 2025)

**Overall**: 71 trades, 40.8% WR, +0.098 avg R — breakeven. London kills it.

**By Session**:
- London: 26 trades, 0% WR. 13 B/E at -0.2 (commission only), 11 soft stops (-7.3 avg), 2 emergency (-16.5 avg)
- NYAM: 24 trades, 66.7% WR, +0.874 avg R — strong
- NYPM: 18 trades, 66.7% WR, +0.376 avg R — solid
- NYLunch: 3 trades, too few to judge

**Best Filters**:
- Gap 5-7pt: 15 trades, 66.7% WR, +2.512 avg R
- Sweep ≥10pt: 33 trades, 51.5% WR, +1.145 avg R
- Confirmation FVG (post-sweep): 21 trades, 42.9% WR, +1.485 avg R (vs 40.8% baseline)
- CT+M60+FVG≥5: 15 trades, 60.0% WR, +0.110 avg R

**London Diagnosis**: Structural problem — 100% runner with B/E at +17pts, soft stop at IFVG edge (avg gap 5-7pts). Trades need +17pts before protection but soft stop catches them at ~5-7pts first. 50% of London trades hit B/E (neutral, -0.2 commission), 42% hit soft stop (real losses).

**Run 2 — No B/E for London (let them run)**:
- Removed B/E entirely for London. Result: ALL 26 trades now hit soft stop or emergency. Zero reach EOD.
- 24 soft stops (avg -10.2), 2 emergency (-16.5)
- Confirms: the issue is NOT B/E management. The IFVG soft stop catches every London trade before it can run.
- The setup itself fails pre-market — price revisits the opposite FVG edge 100% of the time before making a move.

**B/E buffer for US sessions**: Changed from 0 to +1pt. B/E stops now 100% WR (net +0.5 after commission). Good fix.

**Recommendation**: Skip London session in production. If keeping London, needs fundamentally different stop logic (wider stop, not IFVG edge) or a completely different entry criteria. The IFVG edge as soft stop is too tight for pre-market volatility.

**US sessions excluding London (30-day)**:
- 45 trades, 64.4% WR, +0.674 avg R, +4.7 pts/day
- "Gap 5-7" sweet spot: 90.9% WR (11 trades)
- "Sweep ≥10": 68.0% WR, +1.705 avg R (25 trades)
- Confirmation FVG: 69.2% WR, +2.772 avg R (13 trades)

---

## CRITICAL CORRECTION #4: Emergency Stop on Wrong Side (Mar 20, 2026)

### Bug
For fade trades, `emergency_stop = wick ± 1pt` could end up on the **profit side** of entry when price moved significantly between the sweep and the FVG formation. Example: sweep wick at 19707, entry at 19812 for a short — emergency stop was 103pts below entry, meaning it would never trigger on a loss.

30 fade trades had this bug. The emergency stop was effectively disabled for these trades.

### Fix
```python
# For short fade trades:
if s.get('sweep_type') == 'high':  # fade short
    emerg_stp = max(s['wick'] + 1.0, entry_px + 3.0)
else:  # breakout short after low sweep
    emerg_stp = soft_stp + 3.0

# For long fade trades:
if s.get('sweep_type') == 'low':  # fade long
    emerg_stp = min(s['wick'] - 1.0, entry_px - 3.0)
else:  # breakout long after high sweep
    emerg_stp = soft_stp - 3.0
```

Guarantees emergency stop is always on the loss side: `max(wick+1, entry+3)` for shorts, `min(wick-1, entry-3)` for longs. Validated: 0 wrong-side stops after fix.

### Impact
US M60 performance unchanged at +24.5 pts/day — the 30 adjusted trades were not materially inflating results. London breakout, which previously appeared profitable at +16.5 pts/day, collapsed to -2.1 to -5.4 pts/day after the fix. The breakout "edge" was entirely due to the wrong-side emergency stop bug.

---

## M60 Filter Optimization (Mar 20, 2026)

The 60-bar SMA filter on 1-min bars (price above SMA = only take shorts, below = only longs) was tested across all sessions:

- **US sessions (NYAM/Lunch/PM)**: M60 is essential. Without it, US trades are +14.2 pts/day. With it, +24.5 pts/day. The filter removes low-quality counter-trend trades.
- **London session**: M60 **hurts** London. London fade goes from +10.9 pts/day without M60 to worse with it. London price action is choppy and the SMA alignment doesn't predict direction.

**Recommendation**: M60 ON for US sessions, OFF for London.

---

## London Session — Resolution (Mar 20, 2026)

### What was tested
After the initial finding of 0% WR in London, extensive testing was done:

1. **London breakout** (trade WITH the sweep): Uniformly negative across every configuration — every BE threshold, every trailing stop, every TP1 distance, every source. The IFVG pattern does not work with the trend in London. Initial +16.5 pts/day result was entirely due to the emergency stop bug.

2. **London fade with tight BE** (original config): Scratches every trade at B/E. No runners survive.

3. **London fade with 1-contract runner** (final config): Enter 1 contract, if price reaches TP1 distance (17pt), move stop to entry+1pt (BE). Let the full position run to session end (8:30 AM ET). No partials. No trailing stop.

### Winning London config
- **Mode**: Fade (against the sweep)
- **TP1**: 17pt → triggers BE at entry+1pt
- **Trail**: None (trailing stops hurt performance in every configuration tested)
- **Flatten**: Session end at 8:30 AM ET (critical — without this, holding into US hours destroys from +10.9 to -16.0 pts/day)
- **M60**: OFF
- **Implementation trick**: `london_tp1_pct=1` (1% partial triggers BE mechanism) with `be_r_mult_london=999` (prevents R-based early BE)

### London fade results
- 58 trades, 46.6% WR, +10.9 pts/day, Calmar 1.2
- 24 BE scratches (+0.66 each), 18 emergency stops, 13 soft stops, 3 EOD/flatten runners
- 3 EOD runners = 621pts = **247% of total PnL**. Non-EOD = -12.7 pts/day.
- Best source: 1HSH at +22.4 pts/day. Worst: 1HSL at -14.9.
- Positive days: 7/23 (30%). Median day: -12.7. Two big days (3/12: +310.9, 3/20: +287.4) carry everything.

### London reality check
The London fade "works" but is entirely dependent on runners that happen to be open at 8:30 AM flatten. Without those 3 trades, the strategy loses -12.7 pts/day. This is a high-variance, low-probability setup — it bleeds most days and wins big on rare runners.

---

## Combined Portfolio (Mar 20, 2026)

### Configuration
| Component | Trades | Config |
|---|---|---|
| US M60 | 105 | TP1=12pt, BE=3R, M60 filter ON, sessions NYAM/Lunch/PM |
| London Fade | 58 | TP1=17pt→BE+1pt, session flatten 8:30 AM, M60 OFF, no trail |

### Results
| Strategy | Trades | WR | Pts/Day | Calmar | Max DD |
|---|---|---|---|---|---|
| US M60 | 105 | 70.5% | +24.5 | 5.7 | 91.0 |
| London Fade | 58 | 46.6% | +10.9 | 1.2 | 203.1 |
| **Combined** | **163** | **62.0%** | **+31.9** | **4.8** | **160.7** |

### US M60 breakdown
- **By session**: NYAM +24.8 pts/day (core), Lunch +5.6 (marginal), PM +0.5 (flat)
- **By source**: LondonH +45.0 pts/day (1 EOD monster), AsiaH +11.0, LondonL +5.2, 1HSH +2.9, 1HSL +1.2, AsiaL -10.3
- **By direction**: Shorts +39.1 pts/day, Longs -2.2
- **EOD dependence**: 1 EOD trade = 445pts = 86% of total PnL. Non-EOD = +2.4 pts/day.
- **Positive days**: 16/21 (76%), median +6.8

### London Fade breakdown
- **By source**: 1HSH +22.4 pts/day (best), AsiaL +25.0 (small sample), AsiaH -6.0, 1HSL -14.9
- **By direction**: Shorts +10.0 pts/day (58.8% WR), Longs +4.2 (29.2% WR)
- **EOD dependence**: 3 EOD trades = 621pts = 247% of total PnL. Non-EOD = -12.7 pts/day.
- **Positive days**: 7/23 (30%), median -12.7

### Core concern: EOD dependence
Combined portfolio has 4 EOD trades producing 1,066pts = 139% of total PnL of 767pts. The non-EOD portion is -10.3 pts/day. Both strategies are net negative without runners that happen to be open at flatten time. This is the fundamental risk — the strategy depends on rare large winners.

---

## Lumi Engine — Python Tick-Level Audit Complete

The Lumi engine has been implemented in the Python audit script (`tempo_v14_audit.py`) and runs alongside the IFVG engine.

### Lumi Signal Chain (implemented)
1. **HTF Levels** — Built from 60-min bars: FVGs (gap ≥ 3pt), swing points (lookback=2), order blocks (body ≥ 5pt). Dedup within 3pts. Last ~120 bars (~3 days).
2. **HTF Sweep** — 60-min bar sweeps an HTF level (wick past + close back inside, penetration ≥ 1pt). One sweep tracked at a time.
3. **STRICT MSS** — Market structure shift on 3-min bars. SHORT: Low < min(Low) of 5 bars before sweep. LONG: High > max(High) of 5 bars before sweep. Must occur BEFORE entry fill.
4. **LTF Order Block** — 3-min bar with body ≥ 3pt in trade direction (bearish for short, bullish for long).
5. **LTF FVG** — 3-bar gap (2-40pt) on 3-min after OB found.
6. **Limit Entry** — At FVG edge for retracement. Simulated on raw ticks.
7. **Hard Stop** — At swing extreme ± 1pt (tick-based). No soft stop.
8. **3R Target** — Target = entry ± risk × 3.0. No partials.

### Lumi Protections
- Max 3 trades/day
- 2 × 60-min bar cooldown after exit
- Swept level memory (5pt tolerance)
- 30-bar expiry for sweep → OB+FVG chain

### Lumi Bug Fixes (v2)
Three bugs were found and fixed in the initial implementation:
1. **sweep_bar_3m timing**: Was using FIRST 3-min bar of 60-min period, should be LAST (= `CurrentBars[1]` in C#). This made the 30-bar expiry fire ~20 bars early, killing most setups.
2. **Swing stop / MSS ref lookback**: Same timing bug — looked back from wrong 3-min bar position, computing wrong stop levels and MSS references.
3. **Limit expiry after submission**: In C#, once `_lumiTrade` is set (MSS+OB+FVG confirmed), `LumiLTFUpdate` returns early — no more expiry. Python was still expiring pending limits after 30 bars even when all conditions were met.

### Lumi Results (Mar 3 – Apr 4, 2025, 29 trading days, v2 corrected)

| Metric | Value |
|---|---|
| Trades | 35 |
| T/day | 1.2 |
| Win Rate | 20% |
| Avg R | -0.228 |
| Pts/day | -3.4 |
| Total PnL | -99.5 pts |
| Max DD | 284.0 pts |
| Calmar | -0.4 |

### Lumi Exit Reasons
- STOP: 28 trades, 0% WR (all losers at ~-1R)
- TARGET: 7 trades, 100% WR (all winners at ~3R)
- Math: 20% WR × 3R + 80% × -1R = -0.2R expected = slightly negative edge

### Lumi by Session
- London: 17 trades, 23.5% WR, +1.5 pts/day — only positive session
- NYAM: 6 trades, 16.7% WR, -2.1 pts/day
- NYPM: 11 trades, 18.2% WR, -1.9 pts/day

### Lumi by Source
- HTF_OB: 11 trades, 18.2% WR, +0.5 pts/day — only positive source
- HTF_SW: 14 trades, 21.4% WR, -1.9 pts/day
- HTF_FVG: 10 trades, 20.0% WR, -2.0 pts/day

### Lumi Assessment
At 3R target, needs ~25% WR for breakeven. Current 20% WR is slightly below. London session and HTF_OB levels show the best edge. US sessions consistently unprofitable with Lumi.

### Lumi vs NinjaTrader Expectations
NinjaTrader backtest expected ~14 trades/day, 38.1% WR. Python tick-level audit found 1.2/day, 20% WR over 29 days. The frequency gap is due to the strict signal chain and differences in bar construction between NinjaTrader and pandas resampling. The WR gap suggests the NinjaTrader backtest may have been overfitted or tested on a different market regime.

---

## Full Portfolio: IFVG + Lumi

| Component | Trades | Total PnL | Pts/day |
|---|---|---|---|
| IFVG (US M60 + London Fade) | 163 | +766.6 | +26.4 |
| Lumi | 35 | -99.5 | -3.4 |
| **Full Portfolio** | **198** | **+667.1** | **+23.0** |

**Daily PnL correlation (IFVG vs Lumi): -0.113** — negatively correlated (good for diversification), but Lumi is net negative.

Full portfolio max DD: 359.7pts | Calmar: 1.9

**Key finding**: Adding Lumi HURTS the portfolio — pts/day drops from +26.4 to +23.0, max DD worsens from ~200 to 360pts. Lumi needs parameter tuning (London-only? HTF_OB-only?) or should be excluded until WR exceeds 25%.

---

## Key Files (Updated)

| File | Purpose |
|---|---|
| `strategies/tempo/tempo_v14_audit.py` | Main tick-level audit script (IFVG + Lumi, with London flatten) |
| `strategies/tempo/tempo_portfolio_breakdown.xlsx` | Per-strategy portfolio breakdown spreadsheet |
| `trading-system/results/tempo_v14_tick_audit.csv` | Per-trade output — IFVG trades (163 trades) |
| `trading-system/results/tempo_v14_lumi_audit.csv` | Per-trade output — Lumi trades (35 trades) |
| `trading-system/results/TEMPO_IFVG_RESEARCH.md` | Original research document (V2 bar-level results) |
| `strategies/tempo/TEMPO_V14_CORRECTIONS.md` | This file — all corrections documented |
| `strategies/08-keylevel-sweep/keylevel_sweep_fade/TEMPO_PORTFOLIO_BUILD_SPEC.md` | Build spec (IFVG + Lumi) |
| `strategies/08-keylevel-sweep/keylevel_sweep_fade/TempoPortfoliov14.cs` | V14 C# NinjaTrader code |
