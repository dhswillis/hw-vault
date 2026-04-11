# Daily Range Capture Research — NQ Futures

## Objective
Build a portfolio of strategies that captures the daily range with minimal risk.
Scale into positions as the move develops (up to 20 contracts).
Exit based on H/L timing distribution.

---

## EXECUTIVE SUMMARY — FINAL (Phase 19)

### Slippage-Robust Core (TRADE THIS)

**2-signal portfolio: +3,368 pts, +11.3 pts/day, Calmar 7.5 clean, 11/13 months positive**

| Signal | N | WR | Cal@clean | Cal@1pt | Cal@2pt | How It Works |
|--------|---|-----|-----------|---------|---------|-------------|
| **trend_L_gapUP_t30** | 45 | 77.8% | 4.6 | **4.0** | **3.4** | Gap up >10, up >10 at 60min → LONG, stop open-10, trail 30pt |
| **early_S_p15x5** | 99 | 16.2% | 3.3 | 2.0 | **1.1** | Down >10 at 15min → SHORT, pyr every 15pts max 5, trail 50pt |

| Scenario | Total | Pts/Day | Calmar |
|----------|-------|---------|--------|
| **CLEAN** | +3,368 | +11.3 | 7.5 |
| **1pt slip** | +2,400 | +8.0 | **4.6** |
| **2pt slip** | +1,564 | +5.2 | **1.7** |

**Key features**:
- trend_L_gapUP_t30: **4/4 quarters positive**, 78% WR, survives 2pt slip at Cal 3.4
- early_S: fires on different days (only 21% overlap), captures big down moves
- 144 trades in 299 days = ~every other day
- 11/13 months profitable

### Full 4-Signal Portfolio (More Points, Less Robust)

**4-signal portfolio: +7,388 pts, +24.7 pts/day, Calmar 6.6 clean**

| Signal | N | WR | Avg | Total | Cal | How It Works |
|--------|---|----|----|-------|-----|-------------|
| **early_short** (pyr p20x10) | 99 | 19.2% | +26.2 | +2591 | 4.1 | 15min: if price <open-10 → short, pyramid every 20pts |
| **midnight_cross** (pyr p10x5) | 224 | 12.1% | +11.5 | +2566 | 3.2 | Sweep midnight open, cross back, pyramid every 10pts |
| **gap_fade** (pyr p15x3) | 202 | 18.8% | +5.8 | +1177 | 2.1 | Gap >15pts, fade through open, pyramid every 15pts |
| **trend_long** (single, t50) | 110 | 67.3% | +9.6 | +1054 | 1.8 | 60min: if price >open+10 → long, trail 50pts |

**Dies at 2pt slip** — midnight and gap_fade collapse. Only use with limit orders or very fast execution.

---

## Signal Definitions

### 1. trend_long — Calmar 7.6 (refined), 1.8 (portfolio)
- **Logic**: After 60 min of RTH, if price > open + 10pts → go LONG
- **Stop**: RTH open - 10 (best) or open - 5
- **Trail**: 50pt
- **Refinement**: w60_th10_open10_t50 → Cal 7.6, 73 trades, 63% WR, +15.2 avg
- **Gap filter**: Gap UP days → Cal 10.7 (+22.4 avg). Gap DOWN → Cal 0.4
- **Why it works**: NQ has structural long bias. After 1hr, if trending up, ride it.

### 2. early_short — Calmar 4.1
- **Logic**: After 15 min of RTH, if price < open - 10pts → go SHORT
- **Stop**: RTH open + 5
- **Trail**: 50pt, pyramid every 20pts, max 10 positions
- **Why it works**: Strong early weakness = big down day. Pyramid amplifies.
- **Caution**: Q1 2025 contributed +2311 of +2591. May be regime-dependent.

### 3. gap_fade — Calmar 2.1
- **Logic**: If gap > 15pts, wait for price to cross back through open → trade toward prev close
- **Stop**: open + gap*0.5 + 5
- **Trail**: 50pt, pyramid every 15pts, max 3
- **Slippage concern**: Collapses from +1177 to +92 at 1pt slip

### 4. on_long (overnight low) — Calmar 2.3
- **Logic**: If price sweeps below overnight low and crosses back above → go LONG
- **Stop**: sweep extreme - 3
- **Trail**: 75pt
- **Robust**: Survives 2pt slippage (+421)

### 5. midnight_cross — Cal 3.2 (FIXED in Phase 12)
- **Logic**: Sweep above/below ICT midnight open (5:00 UTC), cross back
- **CRITICAL**: Must use RTH-only sweep tracking (no pre-session). Pre-session kills signal.
- **Stop**: sweep extreme + 3pt buffer
- **Pyramid**: add every 10pts, max 5 positions, BE+1 on prior
- **Trail**: 50pt
- **Both directions**: long Cal 3.4, short Cal 2.2
- **Walk-forward**: 3/4 quarters positive (Q1 +1534, Q2 -462, Q3 +840, Q4 +653)

---

## Phase Results Summary

### Phase 6: Dynamic Wick Stops
- **Script**: `/tmp/dynamic_wick.py`
- Best: overnight_buf3_t50_long → 142 trades, +5.3 avg, Cal 1.7
- Open cross with dynamic stops: mostly flat

### Phase 7: Mega Miner
- **Script**: `/tmp/mega_miner.py` — 68,145 trades tested
- **WINNER**: trend_10am_LONG single t50 → Cal 6.0, +11.4 avg, 87 trades
- gap_fade pyramid p15x3_t50 → Cal 1.6
- orb_break single t75 → Cal 0.8
- Strong LONG bias across all signals. Shorts dead except gap_fade.
- Judas swing: DEAD. VWAP cross: dead as single, marginal with pyramid.

### Phase 8: IB + Silver Bullet + Midnight
- **Script**: `/tmp/ib_silver_miner.py` — 30,093 trades tested
- **WINNER**: midnight_cross_p10x5_t50 → Cal 4.2, +12.5 avg (but see Phase 11)
- IB breakout: marginal (Cal 0.2-0.8). IB wide LONG best.
- Silver bullet FVG: DEAD (shorts terrible, longs marginal)
- IB extension fade: DEAD
- Bias pullback: LONG decent (Cal 1.0)

### Phase 9: Trend Signal Refinement
- **Script**: `/tmp/trend_refine2.py` — 19,431 trades tested
- **BEST**: w60_th10_open10_t50_L → Cal 7.6, +15.2 avg, 73 trades
- open10 stop better than open5 (Cal 7.6 vs 6.0)
- 30pt trail works almost as well (Cal 7.4)
- Gap UP filter → Cal 10.7 (strongest filter found)
- **SHORT surprise**: w15_th10_p20x10_t50_S → Cal 3.6, +25.9 avg with pyramid
- Pyramid LONG: best is p20x10_t30 → Cal 4.3, but doesn't beat single (7.6)

### Phase 11: Final Portfolio Audit
- **Script**: `/tmp/final_portfolio.py`
- **CLEAN**: 870 trades, +5102 pts, +17.1 pts/day, Cal 4.6
- **1pt slip**: +990 pts (midnight and gap_fade collapse)
- **2pt slip**: -3709 pts (dead with midnight and gap_fade)
- **Walk-forward**: 3/4 quarters positive
- **Midnight crossback DEAD** — negative in clean, bugs vs IB miner

---

## Key Research Findings (Internet)

### H/L Timing (Our Data — 297 Days)
- ~30% of H/L set in first 30 min of globex
- U-shaped curve: extremes at open and close

### IB Statistics
- NQ breaks IB: 82%. 1.5x extension: ~30%. 2x: ~15%.

### ORB Statistics (6,142 days)
- 67-71.5% continuation rate. 37% of daily range by 10AM.

### Pyramiding (Turtle Method)
- Add every 0.5×ATR. Max 4 units. Move prior to BE.

### Exit Timing
- Best window: 9:30-11:30 AM ET. Avoid midday. Exit by 3:50 PM.

---

## Dead Ends (Do NOT re-test)
- FVG 5M entries — look-ahead bias
- sweep_wick_15m body_top — look-ahead bias
- Re-entry after stop — -0.220 avgR
- BE stops — kills edge (11 configs)
- Fixed stops on open_cross — marginal (Cal <1)
- Judas swing — DEAD in backtest
- IB breakout — marginal (Cal <1)
- Silver bullet FVG — DEAD
- IB extension fade — DEAD
- VWAP cross single — DEAD
- Midnight crossback — negative in clean, slippage-sensitive

---

## Phase 12: Midnight Fix + Pullback + Correlation (DONE)
- **Script**: `/tmp/midnight_fix.py`
- **Midnight fix**: RTH-only sweep tracking → Cal 3.2 (was -0.3 with pre-session)
- **Pullback entry**: trend_pb = Cal 1.3, but 96% overlap with trend_L → duplicate
- **on_L RTH-only**: DEAD (-92 pts) → removed
- **Correlation**: trend_L × early_S = 11% overlap (excellent diversification)
- **Portfolio**: 1022 trades, +9199 pts, +30.8 pts/day, Cal 11.6

## Phase 13: Final Slippage + Gap Filter + Scale to 20 (DONE)
- **Script**: `/tmp/portfolio_final2.py`
- **Gap UP filter**: trend_L_gapUP = 45 trades, 71.1% WR, Cal 4.6 clean, **Cal 2.2 at 2pt slip**
- **Scale to 20**: early_S_x20 = Cal 2.8 clean, **Cal 1.1 at 2pt slip** (better than x10!)
- **Midnight with slip**: Cal 0.8 at 1pt (marginal), dead at 2pt
- **Gap_fade with slip**: Cal 0.1 at 1pt (barely alive), dead at 2pt

## Phase 14: Expand (wait × threshold heatmap) — DONE
- **Script**: `/tmp/phase14_expand.py` — 43,134 trades tested
- **KEY FINDING**: Shorts work with SHORT waits (5-10 min). Longs work with LONG waits (30-60 min).
- **NEW**: S_w5_th5_p15x5 → Cal 9.3 clean, 84 trades, +30.3 avg — but collapses with slippage
- **Decreasing pyramid**: WORSE for shorts (Cal 1.4 vs 2.7), BETTER for longs (Cal 1.4 vs -0.1)
- **Early LONG (mirror of early_short)**: DEAD — all negative at w5/w10/w15

## Phase 15: Updated Portfolio — DONE
- **Script**: `/tmp/phase15_updated_portfolio.py`
- **ultra_S_w5** (5min, th5): Cal 6.0 clean → Cal 0.3 at 1pt → Cal 1.1 at 2pt (slippage-sensitive)
- **ultra_S_w5_th10**: Cal 4.5 clean → Cal 1.0 at 1pt → Cal 0.7 at 2pt
- **KEY**: ultra_S and early_S fire on 68% same days. Ultra makes more on overlap days (+3408 vs +2807)
- **PORT_ROBUST** (trend_L_gapUP + ultra_S_w5): Cal 8.1 clean, **Cal 2.2 at 2pt** — survives heavy slip
- **PORT_NEW** (midnight + trend_L + ultra_S + gap_fade): Cal 7.0, 10/13 months positive
- **PORT_MAX** (all 5 signals): +9824 pts, Cal 7.2 clean — best total but redundant shorts

## Phase 16: Confluence Entries — DONE
- **Script**: `/tmp/phase16_confluence.py`
- **ALL DEAD**: Overnight sweep (too many entries — 6400+), 5M FVG rejection (noisy), 15M sweep (0/4 Q+), BOS+FVG (too few trades)
- fvg5m_rej_S_t50: 6040 trades, Cal 1.1 — only marginally positive, dies at 1pt slip
- **Root cause**: Not selective enough. Too many entries dilute the edge.

## Phase 17: Selective Confluence — DONE
- **Script**: `/tmp/phase17_selective.py`
- **IB sweep LONG t75**: Cal 2.4, 150 trades, 3/4 Q+ — sweep below IB low by 5+, cross back
- **ON sweep SHORT t50**: Cal 2.6, 99 trades, 3/4 Q+ — sweep above ON high by 5+, cross back
- **ALL others dead**: Silver bullet window, lunch, PM reversals, RTH open sweeps all negative
- Both survivors die at 2pt slip

## Phase 18: Precision Entries — DONE
- **Script**: `/tmp/phase18_precision.py`
- Tests: 1M rejection candle entries, candle-level stops, fixed R:R targets
- **ib_rej_L_2R**: Cal 2.4, 429 trades, 38% WR, 3/4 Q+ — but dies at 1pt slip
- **sb_on_rej_S_2R**: Cal 2.3, 56 trades, 43% WR, **4/4 Q+ — most robust WF** — but too few trades
- BOS+FVG+rejection: only 5 trades. Midnight+IB: only 12-19 trades.
- **Key insight**: Candle-level stops are tighter (12pt avg) but 1M rejection is still too noisy

## Phase 19: Combined Portfolio — DONE (FINAL)
- **Script**: `/tmp/phase19_combined.py`
- **DISCOVERY: trend_L_gapUP_t30 = Cal 3.4 at 2pt slip** — 30pt trail better than 50pt for slippage
- **DISCOVERY: trend_L_t30 = 4/4 quarters positive** (only signal with perfect WF)
- **DISCOVERY: early_S_p15x5 = Cal 1.1 at 2pt** (better than p20x10 at Cal 0.9)
- **PORT_ROBUST2**: +3,368 clean, +2,400 at 1pt (Cal 4.6), +1,564 at 2pt (Cal 1.7)
- **IB sweep LONG**: Cal 1.7 clean but adds little after slippage
- **IB range stats**: mean 116.5, median 98.0 pts

---

## Slippage Understanding
Why slippage kills pyramid signals:
- 80% of trades are losers (12-19% WR)
- Each loser pays slippage entry + exit = 2pts per position
- With pyramids: multiple positions per trade = multiplied slip
- 635 trades × 80% losers × 2pts ≈ 1000pts of slippage drag
- Single-position signals (trend_L) survive because only 1 round-trip of slip

## Dead Ends (Do NOT re-test)
- FVG 5M entries — look-ahead bias
- sweep_wick_15m body_top — look-ahead bias
- Re-entry after stop — -0.220 avgR
- BE stops — kills edge (11 configs)
- Fixed stops on open_cross — marginal (Cal <1)
- Judas swing — DEAD in backtest
- IB breakout — marginal (Cal <1)
- Silver bullet FVG — DEAD
- IB extension fade — DEAD
- VWAP cross single — DEAD
- Midnight crossback with pre-session — negative
- Early LONG (mirror of early_short) — all negative
- 5M FVG 1M rejection — too noisy (6000+ trades, Cal 1.1 max)
- 15M swing sweep + crossback — 0/4 quarters positive
- BOS + FVG simplified — too few trades
- Silver bullet window sweep — dead
- Lunch hour reversal — dead
- PM reversal (2PM window) — dead
- RTH open sweep + crossback — dead (0/4 Q+ for longs)
- IB rejection candle entries — dies at 1pt slip (429 trades, edge too thin)
- Midnight + IB direction confluence — too few trades (12-19)

---

## Next Steps
1. Build NinjaTrader strategy for robust core: trend_L_gapUP_t30 + early_S_p15x5
2. Combine with BOS_FVG from V10z (different entry style, could diversify)
3. Early_short regime analysis — is Q1 windfall repeatable?
4. Live paper trading validation
5. Test limit order pyramid adds (instead of market) to reduce slippage further

---

## File Index
| File | Description | Status |
|------|-------------|--------|
| `/tmp/dynamic_wick.py` | Phase 6: dynamic wick stops | Done |
| `/tmp/mega_miner.py` | Phase 7: mega miner + pyramid | Done |
| `/tmp/ib_silver_miner.py` | Phase 8: IB + silver bullet + midnight | Done |
| `/tmp/trend_refine2.py` | Phase 9: trend signal refinement | Done |
| `/tmp/final_portfolio.py` | Phase 11: final portfolio audit | Done |
| `/tmp/midnight_fix.py` | Phase 12: midnight fix + pullback + correlation | Done |
| `/tmp/portfolio_final2.py` | Phase 13: slippage + gap filter + scale to 20 | Done |
| `/tmp/phase14_expand.py` | Phase 14: heatmap of wait × threshold | Done |
| `/tmp/phase15_updated_portfolio.py` | Phase 15: ultra-early short + portfolio compare | Done |
| `/tmp/phase16_confluence.py` | Phase 16: confluence entries (all dead) | Done |
| `/tmp/phase17_selective.py` | Phase 17: selective sweeps + time windows | Done |
| `/tmp/phase18_precision.py` | Phase 18: precision entries, candle stops, fixed R:R | Done |
| `/tmp/phase19_combined.py` | Phase 19: final combined portfolio + slippage | Done |
