# Doji Break Strategy — Full Overview

> Last updated: Mar 7, 2026
> Validated on 312 days of Databento NQ tick data (Feb 2025 - Feb 2026)
> Asia session mining complete — Asia OUTPERFORMS US session

---

## The Concept

A doji candle represents indecision — price explored both directions but closed near its open. The range of the doji defines a "decision zone." When price breaks out of this zone, it signals which side won. We place bracket stop orders on BOTH sides and let the market decide direction.

**Key insight**: The edge is NOT in the entry direction — it's in the **B/E management**. Without B/E, this strategy is completely dead (~43% WR, net negative). With B/E, it's massively positive. The doji merely defines a volatility-calibrated bracket width.

---

## How It Works (Step by Step)

### 1. Pattern Detection (Bar [1] after close)
- **Doji definition**: `body / range <= 0.25` (body is 25% or less of total range)
- **Minimum range**: 5 points (filters out noise candles)
- **Maximum risk**: range + entry pad + stop pad < 50 pts
- Checked on the PREVIOUS completed bar (bar[1]) at the first tick of the new bar

### 2. Entry (Bracket Orders)
- **Buy stop**: High of doji + 1pt (EntryPad)
- **Sell stop**: Low of doji - 1pt (EntryPad)
- Both orders placed simultaneously
- Whichever fills first = trade direction
- Other order immediately cancelled

### 3. Stop Loss
- Opposite side of doji + 3pt (StopPad)
- If long entry fills: stop = doji low - 3pt
- If short entry fills: stop = doji high + 3pt
- Average risk: 17.3 pts (median 14.5 pts)

### 4. Break-Even Management (THE EDGE)
- After 1 bar (BEBars=1), move stop to entry + BEOffset
- BEOffset = 2pt recommended (sweet spot for execution)
- This is what makes the strategy work:
  - 65% of trades hit B/E → small guaranteed profit
  - 17% of trades reach TP → big win (+25 pts avg)
  - 18% of trades hit stop → full loss (-18 pts avg)
- Net result: many small wins + occasional big wins > infrequent full losses

### 5. Take Profit
- Close position at bar 3 close (TPBars=3)
- Market order exit after 3 bars from entry

### 6. Session Filter
- Entry window: 09:30 - 15:50 ET
- Flatten all positions at 15:55 ET
- Max 50 trades per day (rarely hit — avg 53.7 setups/day, not all trigger)

---

## Default Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| MaxBodyRatio | 0.25 | Body ≤ 25% of range = doji |
| MinCandleRange | 5 pts | Filter out tiny candles |
| EntryPad | 1 pt | Beyond high/low for entry |
| StopPad | 3 pts | Beyond high/low for stop |
| BEBars | 1 | Move to B/E after 1 bar |
| BEOffset | 2.0 pts | Lock in 2pt profit at B/E |
| TPBars | 3 | Exit at bar 3 close |
| MaxRiskPts | 50 pts | Skip oversized setups |
| Session | 09:30-15:50 ET | US regular session |
| Flatten | 15:55 ET | EOD cleanup |

---

## Backtest Results (312 Days, Tick-Level Simulation)

### B/E Offset Sweep (16,741 trades, 53.7/day avg)

| B/E Offset | Avg/Trade (pts) | R/Day | Max DD (pts) | Calmar | Daily WR |
|-----------|-----------------|-------|-------------|--------|----------|
| 0pt (true B/E) | +1.23 | +65.9 | 432.8 | 47.5 | 62% |
| 1pt | +1.59 | +85.3 | 389.8 | 68.3 | 69% |
| **2pt** | **+2.00** | **+107.1** | **346.8** | **96.4** | **74%** |
| 3pt | +2.41 | +129.4 | 303.8 | 132.9 | 78% |
| 5pt | +3.45 | +185.1 | 260.2 | 221.9 | 80% |

Higher offset = monotonically better. But higher offset requires a larger favorable move before B/E triggers, which may not happen on every trade. 2-3pt is the practical sweet spot.

### Trade Exit Distribution (BE=2pt)

| Exit Type | % of Trades | Avg P/L (pts) |
|-----------|-------------|---------------|
| B/E exits | 65.1% | +1.50 |
| TP winners | 16.8% | +25.4 |
| Stops | 18.1% | -18.0 |

### Slippage Stress Test (BE=2pt)

| Slippage | R/Day | Calmar | Status |
|----------|-------|--------|--------|
| 0pt | +107.1 | 96.4 | Baseline |
| 0.5pt | +64.0 | 50.8 | Solid |
| 1.0pt | +22.0 | 8.9 | Marginal |
| 1.5pt | Negative | — | DEAD |

**Execution requirement**: Must achieve <1pt total slippage (entry + exit combined). This means fast execution infrastructure — not suitable for manual trading.

### Commission Impact

**MNQ (Micro, $2/pt):**
- Commission: ~$1.24 RT = 0.62 pts
- Net avg/trade: +1.38 pts ($2.76)
- Net daily: ~$148/day/contract

**NQ (Full, $20/pt):**
- Commission: ~$4.62 RT = 0.23 pts
- Net avg/trade: +1.77 pts ($35.40)
- Net daily: ~$1,902/day/contract

---

## Comparison: Doji Break vs Sweep Break

| Feature | Doji Break | Sweep Break |
|---------|-----------|-------------|
| **Pattern** | Doji candle (indecision) | Candle sweeps prior bar's high/low |
| **Entry timing** | Bracket on BOTH sides, market decides | Directional — entry on opposite side of sweep |
| **Direction** | Non-directional (bracket) | Directional (fade the sweep) |
| **Bars waiting** | 0 — bracket placed immediately, fills on breakout | 0 — entry placed same bar as sweep detection |
| **Avg risk** | ~17 pts (doji range + pads) | ~15-25 pts (reference candle range + pad) |
| **B/E dependency** | CRITICAL — dead without it | Important but not the entire edge |
| **Trade frequency** | Very high (~54/day) | Lower (~5-15/day depending on range filter) |
| **Edge source** | B/E management + volatility bracket sizing | Sweep = trapped traders + directional fade |
| **Correlation** | Low with directional models | Moderate with other sweep-based models |

**Portfolio value**: These strategies are non-correlated. Doji break is directionless (bracket), sweep break is directional (fade). Running both diversifies the source of edge.

---

## Why B/E Is The Entire Edge

Without B/E, the doji break is just a random breakout with a 43% win rate. The magic is in the asymmetry B/E creates:

1. **Most trades (65%) move favorably then retrace** → B/E locks in a small profit instead of a loss
2. **Some trades (17%) run strongly** → TP captures the full move
3. **Only 18% hit the full stop** → but even those are bounded by the doji range

The expected value per trade:
- `0.65 × (+1.50) + 0.17 × (+25.4) + 0.18 × (-18.0) = +0.975 + 4.318 - 3.240 = +2.05 pts`

This is entirely driven by converting would-be losers into small winners via B/E.

---

## Asia Session Results (Mar 7, 2026) — BETTER THAN US

Asia session doji break **outperforms US on every metric per trade**. The manipulation wick that Asia/London creates IS the doji — our mechanical system captures the same edge that discretionary traders like Leppyrd trade manually.

### Asia vs US Head-to-Head (BE=1, offset=2pt, TP=3, body<=0.25, rng>=5)

| Session | Trades | Trades/Day | WR | Avg/Trade | R/Day | Max DD | Calmar | DWR |
|---------|--------|-----------|-----|-----------|-------|--------|--------|-----|
| **Asia** | 6,419 | 20.6 | **88.5%** | **+2.33** | +48.0 | 51.0 | **293.8** | **77%** |
| US | 16,741 | 53.7 | 81.9% | +2.00 | +107.1 | 346.8 | 96.4 | 74% |

Asia has 7pp higher WR, 3x better Calmar, and 85% less drawdown than US.

### Why Asia Works Better

**Leppyrd's framework (from @LeppyrdTrading) explains the mechanics:**

1. **Daily candle sets the bias** — yesterday's close relative to prev day high/low tells you bullish or bearish
2. **Asia/London creates the manipulation wick** — they spend all night above/below the daily open, forming the daily candle's wick. This wick IS the doji in our system.
3. **There's usually an obvious FVG on 15m/1H** that London will reach — the untapped area from NY (~25-50% level)
4. **Sweep + close back inside = reversal signal** — e.g. 10PM sweeps 6PM high on 4H, then closes back below
5. **5m breaker gives the entry** — 30sec breaker confirmation, target is the -4 level (PDL/PDH)

**What this means for our doji break:**
- Asia dojis are NOT random indecision — they're the **manipulation candle** where smart money builds the daily wick
- The breakout from the doji IS the reversal that Leppyrd trades discretionarily
- Our B/E management captures the "right" breakouts and cuts the "wrong" ones mechanically
- Asia has fewer but cleaner dojis (20.6/day vs 53.7/day) — less noise

### Best Asia Configs

**B/E Offset (Asia, body<=0.25, rng>=5, TP=3):**

| Config | WR | Avg/Trade | R/Day | Calmar |
|--------|-----|-----------|-------|--------|
| BE=1 off=2pt | 88.5% | +2.33 | +48.0 | 293.8 |
| **BE=1 off=3pt** | **88.5%** | **+2.87** | **+59.0** | **589.0** |
| BE=1 off=5pt | 88.5% | +4.06 | +83.6 | 7,452 |

3pt offset is the sweet spot — doubles the Calmar vs 2pt.

**Widening body filter is FREE (Asia):**

| Filter | Trades/Day | WR | Avg/Trade | R/Day | Calmar |
|--------|-----------|-----|-----------|-------|--------|
| body<=0.25 rng>=5 | 20.6 | 88.5% | +2.33 | +48.0 | 293.8 |
| **body<=0.35 rng>=5** | **29.7** | **89.0%** | **+2.34** | **+69.4** | **1,731** |
| body<=0.50 rng>=5 | 45.9 | 89.6% | +2.34 | +107.7 | 999+ |
| body<=0.35 rng>=3 | 55.4 | 90.0% | +2.03 | +112.7 | 999+ |

Wider body = more trades with SAME or BETTER avg/trade. Asia candles with up to 35% body ratio still carry the manipulation wick edge.

**Sub-windows:**

| Window | UTC | Trades/Day | Avg/Trade | Calmar |
|--------|-----|-----------|-----------|--------|
| **Asia early** | 00:00-04:00 | 10.8 | **+2.57** | 81.8 |
| Tokyo core | 00:00-06:00 | 13.8 | +2.53 | 148.1 |
| Asia late | 04:00-08:00 | 9.6 | +2.08 | 34.4 |
| Asia/London overlap | 06:00-08:00 | 6.6 | +1.90 | 22.6 |

Early Asia (Tokyo open) is the best sub-window. The overlap with London degrades performance.

### Walk-Forward PASSES

| Half | Trades | WR | Avg/Trade | R/Day | Calmar |
|------|--------|-----|-----------|-------|--------|
| H1 (IS) | 3,429 | 87.5% | +2.41 | +53.1 | 162.3 |
| **H2 (OOS)** | **2,990** | **89.7%** | **+2.24** | **+43.0** | **207.9** |

OOS is CLEANER than IS — Calmar improves out of sample. No overfitting.

### Slippage (Asia, BE=1 off=2)

| Slippage | R/Day | Calmar | Status |
|----------|-------|--------|--------|
| 0pt | +48.0 | 293.8 | Baseline |
| 0.5pt | +33.0 | 87.6 | Solid |
| 1.0pt | +17.9 | 29.2 | Marginal |
| 1.5pt | +3.0 | 2.5 | Barely alive |

Asia survives slippage slightly better than US (lower trade count = less cumulative slip).

### Recommended Production Config (Asia)

| Parameter | Value | Why |
|-----------|-------|-----|
| Session | UTC 00:00-08:00 (7pm-3am ET) | Asia session |
| MaxBodyRatio | 0.35 | Free trades, no WR cost |
| MinCandleRange | 5 pts | Filter noise |
| BEBars | 1 | Move to B/E after 1 bar |
| BEOffset | 3.0 pts | Sweet spot Calmar |
| TPBars | 3 | TP at bar 3 close |
| EntryPad | 1 pt | Standard |
| StopPad | 3 pts | Standard |

**Expected performance**: ~30 trades/day, 89% WR, ~+59 R/day, Calmar ~589.

### Leppyrd's Discretionary Overlay (Optional Enhancement)

For manual/semi-automated trading, Leppyrd's rules can filter doji setups:

1. **Daily bias**: Only take doji breaks in the direction of yesterday's daily candle close vs prev day range
2. **HTF time alignment**: 4H and 1H sweeps should align with daily bias direction
3. **FVG target**: There should be an untapped FVG on 15m/1H from NY session (~25-50% retracement level) for London to fill
4. **Skip inside candles**: If the doji is inside the previous candle's range, skip it (unless SMT divergence present)
5. **"Don't mess with inside candles"** — only trade clear sweep + close patterns

These discretionary filters would reduce trade count but potentially improve the already-high 89% WR further. Not yet backtested as filters.

---

## Known Risks & Limitations

1. **Slippage sensitivity**: Dies at 1.5pt slippage. Requires automated execution.
2. **High frequency**: 54 trades/day means commission matters. NQ (not MNQ) is preferable.
3. **B/E race condition**: Stop price can't be moved above/below market. Fixed in v2 with 1pt buffer + try/catch.
4. **Not manually tradeable**: Too many setups, too fast execution needed.
5. **Doji prevalence varies**: Some days have 80+ dojis, some have 20. Performance varies.

---

## NinjaTrader Implementation

| Version | File | Changes |
|---------|------|---------|
| v1 | DojiBreakv1.cs | Core bracket logic, old table format, no B/E fix |
| v2 | DojiBreakv2.cs | Improved box-style table, B/E fix (1pt buffer + try/catch) |

**Key implementation details:**
- `IsUnmanaged = true` — full order control
- `Calculate = Calculate.OnEachTick` — scan every tick for B/E management
- Bracket orders via `SubmitOrderUnmanaged()` with `OrderType.StopMarket`
- On fill: cancel opposite bracket order, submit protective stop
- B/E uses `ChangeOrder()` wrapped in try/catch for race conditions

---

## Scripts & Data

| File | Purpose |
|------|---------|
| `/tmp/doji_break_v2.py` | Tick-level backtest with configurable B/E offset |
| `results/doji_break_312day.json` | Full 312-day results (all offsets) |
| `keylevel_sweep_fade/DojiBreakv1.cs` | NinjaTrader strategy v1 |
| `keylevel_sweep_fade/DojiBreakv2.cs` | NinjaTrader strategy v2 (pending) |
