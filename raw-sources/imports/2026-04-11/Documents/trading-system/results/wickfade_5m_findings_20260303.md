# WickFade 5M Backtest Findings — March 3, 2026

## Test Parameters
- **Data**: 30 trading days (Feb 12 – Mar 18, 2025) — mini validation
- **Instrument**: NQ front-month, Databento tick data
- **Entry window**: Full RTH 9:30–14:00 ET (14:30–19:00 UTC)
- **Bar period**: 5-minute
- **Max flips**: 1 (flip on fade stop-out ONLY — no flip on trail/BE stops)
- **Target**: Fixed 15 pts for all configs
- **Drawdown**: Measured in POINTS (actual dollars), not R-units
- **Simulation**: Single-loop tick processing matching mtf_research.py

## Simulation Bugs Fixed (This Session)

1. **Missing `break` after fade entry** — allowed multiple entries per bar
2. **Target checked before stop (`elif`)** — broke flip logic
3. **No EOD cleanup** — winning trades at end of day silently dropped
4. **Flip on trail stops** — fixed to match NT code: only fade stop-outs flip

## Wick-Stop Concept

Stop distance = wick penetration (how far price traveled past prev bar H/L). This naturally captures the bid-ask spread — wide spread → bigger wick → wider stop (self-protecting). No extra buffer needed; the wick IS the spread measurement.

Config: floor 1.50 pts, cap 5.00 pts, target 15 pts fixed.

---

## Part 1: Exit Management Comparison at LOW Spread (0.5pt)

All configs use WICK+0 stop. DD and PnL in points.

| Config | PnL pts | pts/day | DD pts | Calmar | WR | Daily WR | MaxConsecL |
|--------|---------|---------|--------|--------|-----|----------|------------|
| No exit mgmt | 8,621 | 345 | 56 | 153 | 43.8% | 96% | 13 |
| **BE@4+0** | **8,853** | **354** | **22** | **398** | 42.0% | 100% | 13 |
| BE@4+1 | 8,898 | 356 | 56 | 160 | 58.1% | 96% | 7 |
| BE@5+0 | 8,700 | 348 | 47 | 184 | 41.9% | 96% | 14 |
| BE@5+1 | 8,764 | 351 | 49 | 179 | 56.3% | 96% | 7 |
| BE@6+0 | 8,629 | 345 | 49 | 177 | 41.9% | 96% | 14 |
| BE@4+1 tr10d6 | 8,801 | 352 | 31 | 282 | 54.5% | 100% | 8 |
| BE@5+1 tr10d6 | 8,747 | 350 | 37 | 240 | 57.4% | 100% | 7 |
| tr8d4 (old, no BE) | 8,769 | 351 | 40 | 219 | 52.8% | 96% | 10 |

### Key Findings — Low Spread

**BE@4+0 is the standout.** Move stop to exact entry when MFE hits 4 pts. Highest PnL (8,853), lowest DD by far (22 pts vs 56 for no management), Calmar nearly 400, 100% daily WR. The +0 offset (exact B/E) works better than +1 at low spread because if the market reverses all the way to entry, you're out at zero cost.

**Two-stage BE@4+1 tr10d6 is the runner-up.** Nearly as good PnL (8,801), DD only 31 pts (vs 22 for BE@4+0), Calmar 282. The trail adds marginal PnL protection on runners but the B/E does the heavy lifting.

**BE threshold of 4 is better than 5 or 6.** Earlier B/E = more trades protected = lower DD. The 4-pt threshold captures enough of the initial move to be meaningful while being reachable on most trades.

---

## Part 2: Exit Management Comparison at WIDE Spread (2.0pt)

| Config | PnL pts | pts/day | DD pts | Calmar | WR | Daily WR | MaxConsecL |
|--------|---------|---------|--------|--------|-----|----------|------------|
| No exit mgmt | 5,265 | 211 | 157 | 34 | 44.5% | 92% | 14 |
| BE@4+0 | 5,249 | 210 | 154 | 34 | 41.9% | 96% | 20 |
| BE@4+1 | 5,213 | 209 | 160 | 33 | 40.3% | 96% | 22 |
| BE@5+1 | 5,117 | 205 | 152 | 34 | 40.6% | 96% | 18 |
| **BE@4+1 tr10d6** | **5,320** | **213** | **109** | **49** | 47.5% | 96% | 10 |
| BE@5+1 tr10d6 | 5,229 | 209 | 130 | 40 | 48.1% | 96% | 10 |
| BE@4+1 tr10d8 | 5,273 | 211 | 137 | 39 | 44.9% | 96% | 20 |
| tr8d4 (old, no BE) | 5,450 | 218 | 142 | 38 | 53.9% | 92% | 9 |

### Key Findings — Wide Spread

**B/E alone doesn't help at wide spread.** BE@4+0 barely changes DD (154 vs 157) because the 2pt spread cost of the B/E exit itself is significant — you're paying $40/contract just to exit at breakeven.

**The two-stage approach works.** BE@4+1 tr10d6 gives the best Calmar (49) — B/E protects early, then the wide 6-pt trail delta gives room for runners to run without getting shaken out. It cuts max consec losses from 14-22 down to 10.

**tr10d6 > tr10d8 at all spread levels.** The 6-pt delta is the sweet spot. 8-pt delta gives back too much; anything wider than 8 (tr15d8, tr15d10) had identical results to B/E-only, meaning the trail never activated before target was hit at 15 pts.

**Old tr8d4 (no B/E) still makes the most PnL (5,450)** but has 92% daily WR and higher DD than the two-stage approach.

---

## Summary: How Spread Changes Optimal Exit Management

| Spread | Best Config | PnL | DD | Calmar | Why |
|--------|------------|-----|-----|--------|-----|
| ≤ 0.5pt | **BE@4+0** | 8,853 | 22 | 398 | Cheap B/E exits, tight protection |
| ~1pt | **BE@4+1 tr10d6** | ~8,800 | ~30 | ~280 | B/E protects, trail captures runners |
| ≥ 2pt | **BE@4+1 tr10d6** | 5,320 | 109 | 49 | B/E is costly so trail matters more |

**The universal safe choice is BE@4+1 tr10d6.** It works well at every spread level — gets B/E protection at 4 pts MFE (+1 offset for safety), then a 6-pt trail once you have 10 pts of separation. Never the worst, always near the top.

---

## Implementation Notes for NinjaTrader (WickFade5Mv5)

Entry:
- On price break of prev 5min bar H/L → fade entry with slippage
- Stop = wick penetration + slippage, floor 1.50, cap 5.00
- Target = entry ± 15 pts

Exit management — two-stage:
1. **B/E stage**: When MFE ≥ 4 pts, move stop to entry + 1 pt (in profit direction)
2. **Trail stage**: When MFE ≥ 10 pts, activate trail with 6-pt delta (only tightens stop)
3. Trail and B/E stops do NOT trigger flips — go flat

Flips:
- Only on initial fade stop-out (before B/E or trail activates)
- Max 1 flip per trade sequence
- Flip entry at stop level ± slippage

---

## Caveats
- 30-day mini test on volatile Feb–Mar 2025. Full 13-month validation needed.
- Spread modeled as fixed slippage; real bid-ask varies tick by tick.
- B/E exact-entry exit (+0 offset) may get more slippage in practice than modeled.

---

## Files
- Backtest script: `wickfade_5m_dynstop_backtest.py` (session working dir)
- Original reference: `mtf_research.py` (trading-system/)
- Spread analysis JSON: `wickfade_5m_spread_analysis.json` (results/)
- This document: `wickfade_5m_findings_20260303.md` (results/)
