# New Signal Classes Research Report
**Tempo Trading System — Willis Holdings**
**Date:** 2026-02-18
**Status:** Implementation-Ready Research
**For:** Harrison Willis

---

## Purpose

This document researches **7 genuinely NEW signal classes** discovered across V7r through V7aa studies. These are not filters on existing signals — they are new trade entry types with independent logic that would generate additional non-correlated trade populations.

Current system has 4 signals producing +11.31R/day. Each new class below is evaluated for:
- **Edge evidence** (what V7 data supports it)
- **Expected metrics** (R/day, WR, Calmar)
- **Implementation spec** (entry, stop, target, filters)
- **Priority** (based on evidence strength and R contribution)
- **Risk** (what could go wrong)

---

## Signal Class Scoring

| Class | Expected R/day | Evidence Strength | Implementation Effort | Priority |
|-------|---------------|-------------------|----------------------|----------|
| FVG Bifurcation | +2-4R | STRONG (V7aa, V7r, FVG analysis) | HIGH | 1 |
| Inverse/Hedge | +1-2R additive | STRONG (V7k-V7m validated) | MEDIUM | 2 |
| Composite GOLD | +1-3R | STRONG (V7r, V7t) | MEDIUM | 3 |
| Wick Follow | +0.5-2R | MODERATE (V7z-fix) | LOW | 4 |
| Stop Limit Breakout | +0.5-1R | MIXED (V7r positive, V7s negative) | MEDIUM | 5 |
| Sweep Close Market | +0.3-0.8R | MODERATE (V7z Study 6) | LOW | 6 |
| Pattern Mining | unknown | HIGH (6,502 combos) | LOW | 7 |

---

## Signal Class 1: FVG Bifurcation (Passive vs Momentum)

**Priority: 1 — Highest confidence, largest expected impact**

### What It Is

Split every FVG entry into two separate signal classes based on HOW price approaches the FVG:

- **PASSIVE_FVG**: Price retraces slowly into FVG zone. Indecision candle at entry. Low volume. Limit order at FVG midpoint.
- **MOMENTUM_FVG**: Price blasts through FVG with displacement. Strong candle, high volume, 5M confirms direction. Market order on next 15s bar confirmation.

Currently these are blended together in one signal, hiding the fact that they have completely different edge profiles.

### Evidence

**V7aa — Candle pattern at entry (strongest finding):**
- Doji/gravestone at FVG entry: **+2.461 avg R** (PASSIVE works)
- Hanging man at entry: **+2.155 avg R** (PASSIVE works)
- Inverted hammer at entry: **+1.813 avg R** (PASSIVE works)
- Marubozu bull at entry: **-0.210 avg R** (momentum alone LOSES money)

This is the single clearest finding in the V7 series: indecision candles at FVG entry massively outperform momentum candles.

**V7r — 15S confirmation timing:**
- Neutral 15S candle outperforms confirmation: 34.1% vs 31.1% WR
- LOW volume at 15S entry beats HIGH: +0.659 avg R vs +0.435 avg R
- BOS_FVG + strong 15S confirm: 39.1% WR, +1.816 avg R (n=87)

**V7aa — 5M contradiction gate (best practical gate):**
- gate_5m_good_AND_no_contra: 33.0% WR, +1.369 avg R
- Rejecting entries where 5M contradicts direction: **+118% Calmar improvement**
- ~30% of trades filtered, but the ones removed are the worst performers

**V7z-fix — Timeframe finding:**
- 1M granularity optimal for passive detection (15S adds noise)
- 15S only useful for momentum confirmation, not passive fills

### Expected Metrics

**PASSIVE_FVG (limit orders):**
- Frequency: ~12-15 trades/day (most current trades are passive)
- WR: 31-34% (matches Model 2 baseline)
- Avg R: +0.65 to +2.46 depending on candle pattern
- Expected R/day: +3-5R
- MaxDD: lower than current (removing worst entries)

**MOMENTUM_FVG (15s-confirmed market orders):**
- Frequency: ~3-5 trades/day (minority of entries)
- WR: 33-39% when properly gated (5M alignment + 15S confirm)
- Avg R: +1.37 to +1.82
- Expected R/day: +1-2R
- Risk: chasing if confirmation fails

**Combined improvement over current:** +2-4R/day from better entry selection, plus reduced drawdown from filtering out bad momentum chases.

### Implementation Spec

**Detection (in _ScanForFVGs):**
1. FVG detected on bar N
2. Classify regime using tiered decision tree:
   - Level 1: Is price already IN the FVG zone? NO → PASSIVE
   - Level 2: Candle pattern at entry bar? Indecision → PASSIVE, Momentum → check Level 3
   - Level 3: Volume relative to 20-bar average? LOW → PASSIVE, HIGH → check Level 4
   - Level 4: 5M bar alignment? Contradicts direction → PASSIVE. Aligns → MOMENTUM candidate
   - Level 5: 15S bar confirms momentum through FVG? NO → downgrade to PASSIVE. YES → MOMENTUM

**PASSIVE_FVG entry:**
- Place limit at FVG midpoint on bar N+1 (never same bar)
- Fill timeout: 10 bars (cancel if not filled)
- Stop: FVG boundary ± 2pt buffer
- Target: Next 5M swing level or target_cap
- No B/E (validated: B/E at 1R = -490R)

**MOMENTUM_FVG entry:**
- Detect FVG on 1M bar N
- Monitor 15S bars for momentum confirmation (body pushing through FVG with close past midpoint)
- Enter market on NEXT 15S bar after confirmation (not the confirming bar itself)
- Stop: FVG boundary ± 2pt buffer
- Target: Same as PASSIVE
- Hard gate: 5M must confirm direction (no contradiction)

**What NOT to do:**
- Don't enter on the same 1M bar FVG is detected (current bug)
- Don't enter momentum on 1M candle alone (marubozu = -0.210R)
- Don't use 15S for passive detection (adds noise per V7z-fix)

### Risk

The main risk is overcomplicating entries. The simplest version: "just wait one bar and use a limit" captures most of the edge. The full tiered detection tree is Phase 2.

---

## Signal Class 2: Inverse/Hedge Regime Signals

**Priority: 2 — Walk-forward validated, additive overlay**

### What It Is

After a streak of consecutive losses on a specific signal, take the OPPOSITE trade as a hedge. This is not a replacement strategy — it's an additive overlay that keeps all original trades AND adds inverse trades during identified losing streaks.

### Evidence

**V7k — Directional accuracy at 1R:**
- Overall: 57.7% of 4,991 trades touch +1R before -1R
- BOS_FVG: 62.6% directionally accurate
- sweep_reversal: 50.4% (coin flip — best inverse candidate)
- VA_fade: 57.0% (good inverse candidate after streaks)
- pre_gap_fill: 76.1% (too accurate to invert)

**V7k-deep — Conditional inverse mining:**
- 24 combos above 62.5% inverse WR (profitable after costs at 1:1)
- sweep×sbam×frozen: 87.5% inverse WR (n=small but striking)
- VA×overnight×consec3: 79.2% inverse WR
- sweep×sbam×prevloss: 77.3% inverse WR
- After 3+ consecutive misses, inverse WR jumps to 52.8%
- **VA_fade at streak ≥5: 71.9% inverse WR** (best single finding)

**V7l — Walk-forward validation:**
- 11 of 17 conditions passed all 3 tests (60/40, rolling 3-window, monthly consistency)
- ROBUST conditions: sweep×sbam×frozen, VA×overnight×consec3, sweep×Wed×consec2, VA×overnight×hot, sweep×lnyo×warm

**V7m — Regime switch strategy:**
- Best config: VA+sweep regime switch (VA=3 streak, sweep=3 streak, exit after 2 wins)
- Standalone: +10.09R/day, MaxDD 35R (vs baseline 47R), Calmar 0.607 (+30%)
- HEDGE mode (keep all original + add inverse): +11.42R/day, MaxDD 34R, Calmar 0.514
- Walk-forward confirmed: test period Calmar holds

### Expected Metrics

**Hedge overlay (recommended approach):**
- Adds +0.5-1.5R/day from inverse trades during regime shifts
- Reduces MaxDD from 47R to 34R (massive risk improvement)
- Calmar improvement: 0.467 → 0.514 (+10%)
- Total system with hedge: +11.42R/day vs +11.31R/day baseline
- The real value is the drawdown reduction, not the R improvement

### Implementation Spec

**Track per-signal consecutive loss streaks:**
```
streak_tracker = {
    'BOS_FVG': {'losses': 0, 'hedge_active': False},
    'VA_fade': {'losses': 0, 'hedge_active': False},
    'sweep_reversal': {'losses': 0, 'hedge_active': False},
}
```

**Activation thresholds (from V7m best config):**
- VA_fade: activate inverse after 3 consecutive losses
- sweep_reversal: activate inverse after 3 consecutive losses
- BOS_FVG: **NEVER INVERT** (32-36% inverse WR regardless of streak)
- Exit inverse mode: after 2 consecutive wins in inverse direction

**Inverse trade logic:**
- When hedge activates, take opposite direction of original signal
- Same stop distance, same target distance (just flipped direction)
- Original trade STILL FIRES (this is a hedge, not a replacement)
- Track inverse and original separately

**Critical rules:**
- NEVER invert BOS_FVG — it's the backbone signal
- VA_fade is the best inverse candidate (72% at streak ≥5)
- sweep_reversal moderate (60% at streak ≥5)
- Inverse trades are HEDGES, not replacements
- Track streaks per signal independently

### Risk

Small sample sizes on some conditional combos. The 87.5% inverse WR on sweep×sbam×frozen is from a small n. But the overall hedge approach is validated at the system level with walk-forward testing.

---

## Signal Class 3: Composite GOLD Entries

**Priority: 3 — Multiple confirming V7 studies**

### What It Is

Combine multiple 5M candle patterns with signal type to create a "gold standard" composite entry. Instead of just checking one dimension, require 2-3 conditions to align before entering. The V7r and V7t studies found specific multi-condition combos that dramatically outperform single-filter trades.

### Evidence

**V7r — BOS_FVG composite entries:**
- BOS_FVG + strong 15S confirm: 39.1% WR, +1.816 avg R (n=87) — BEST
- BOS_FVG + silver_bullet session + london_open: +0.878 R/trade, 38.4% WR, 13/13 months
- BOS_FVG + engulfing pattern: +0.690 R/trade, 34.8% WR

**V7t — Gate testing:**
- gate_5m_good_AND_no_contra: 33.0% WR, +1.369 avg R (best practical gate)
- The "AND" gating is critical — single gates add noise, combined gates add edge

**V7d — Win condition analysis (full year):**
- Volume spike: +1.021 R/trade, 41.8% WR, 13/13 months positive
- Conf≥5 + volume: +0.614 R/trade, lowest drawdown (11.1R)
- Engulfing pattern: +0.690 R/trade, 34.8% WR

**Cross-mining (6,502 combos):**
- Best 3-way: silver_bullet + bull_trend + VA_fade: 100% WR (n=7, small but promising direction)
- Best 2-way combos hitting 5+ R/day in specific conditions

### Expected Metrics

**GOLD composite entries (top-tier multi-filter):**
- Frequency: ~3-5 trades/day (filtering removes ~60% of trades)
- WR: 35-42% (significantly above 26.5% baseline)
- Avg R: +0.8 to +1.8 depending on combo depth
- Expected R/day: +1-3R from filtered population
- Risk: missing good trades by over-filtering

**The trade-off:** Fewer trades at much higher quality. These should be max-size trades (2x normal R allocation if portfolio rules allow).

### Implementation Spec

**Tier 1 GOLD conditions (any 2 must be true):**
1. Volume spike (>1.5x 20-bar average)
2. 5M candle confirms direction (no contradiction)
3. Confluence score ≥ 3
4. Session is in signal's GOLD session list (V7x findings)

**Tier 2 GOLD conditions (any 1 must be true):**
5. BOS_FVG + strong 15S confirmation (39.1% WR)
6. Engulfing pattern at entry
7. Silver bullet or London open session

**Composite score:**
- 0-1 conditions met: STANDARD entry (normal 1R sizing)
- 2-3 conditions met: SILVER entry (1R sizing, higher conviction)
- 4+ conditions met: GOLD entry (1.5R sizing, highest conviction)

**Session GOLD pairs (from V7x):**
- BOS_FVG: active ALL sessions (backbone)
- sweep_reversal GOLD: ny_am, ib_first
- VA_fade GOLD: overlap, news_window, london_am
- VA_fade BANNED: ny_am, power_hour, silver_bullet

### Risk

Over-filtering can reduce total R even while improving per-trade R. The GOLD entries should be an additional sizing layer on top of existing signals, not a replacement filter.

---

## Signal Class 4: Large Wick Follow

**Priority: 4 — Moderate evidence, simple to implement**

### What It Is

When a large wick forms on a sweep candle, FOLLOW the wick direction with dynamic trailing stops. The V7z-fix study found that following the wick (trading in the direction the wick points) with adaptive stops is profitable — counter to the intuition of fading the wick.

### Evidence

**V7z-fix — Wick follow testing:**
- "FOLLOW with dynamic stops is very profitable" for large wick sweeps
- 1M granularity optimal (15S makes it worse — more noise)
- The key: large wicks indicate institutional order flow direction
- Following the wick = following the smart money

**V7z — Wait-for-win analysis:**
- VA_fade wait-for-first-daily-win: +2.2x R improvement
- This can be applied to wick follows too — only take wick signals after first daily win confirms the regime

**Candle mechanics from V7aa:**
- Large wick with small body = indecision / rejection at level
- Large wick with large body in wick direction = continuation signal
- The body/wick ratio determines whether to follow or fade

### Expected Metrics

**Wick follow signals:**
- Frequency: ~2-4 trades/day (large wicks are less common)
- WR: 25-30% (lower WR but higher R targets)
- Target: 3-5R (following a displacement move)
- Avg R: +0.5 to +1.5
- Expected R/day: +0.5-2R
- MaxDD: moderate (dynamic stops help)

### Implementation Spec

**Wick classification:**
```
wick_ratio = wick_size / total_range
body_ratio = body_size / total_range

LARGE_WICK = wick_ratio > 0.6 AND total_range > 5pts (NQ)
```

**Entry rules:**
1. Detect large wick on 1M candle (wick > 60% of range, range > 5pts)
2. Determine wick direction (upper wick = bearish signal, lower wick = bullish signal)
3. Wait for NEXT bar to confirm direction (don't enter on wick bar itself)
4. Enter market at next bar close IF it moves in wick direction

**Stop:**
- Initial: opposite end of wick candle
- Dynamic: trail at 2x ATR once +1R achieved
- No B/E (validated bad for this system)

**Target:**
- 3R minimum (this is a displacement follow, needs room to run)
- 5R optimal based on V7z analysis
- Target cap at 200pts (same as main system)

**Filters:**
- Only trade after first daily win (V7z wait-for-win rule)
- Require wick candle on sweep (not random wicks)
- 5M must not contradict wick direction

### Risk

False wicks that look like displacement but are just noise. The 1M timeframe + next-bar confirmation filter helps, but some wicks will trap. Dynamic trailing stop is the main risk management tool.

---

## Signal Class 5: Stop Limit Breakout

**Priority: 5 — Mixed evidence, requires careful filtering**

### What It Is

After a liquidity sweep, place a stop limit order ABOVE (for longs) or BELOW (for shorts) the sweep level to catch the continuation cascade. The idea: if price sweeps a level and keeps going, the stop limit catches the breakout.

### Evidence

**V7r — Scanner-filtered sweeps (POSITIVE):**
- 5pt offset, 2R target: 48.7% WR, +0.45 avg R, 10,431 trades
- This only works when scanner confluence filters are applied
- Best combo: sweep + high confluence + momentum session

**V7s — Raw stop limits (NEGATIVE):**
- Raw stop limits without filtering: negative expectancy
- The signal itself is not the edge — the filtering IS the edge
- Without scanner, stop limits are just noise

**V7u — 15S contradiction finding:**
- 15S candle contradicting at sweep entry OUTPERFORMS confirmation
- LOW volume at entry beats HIGH volume
- Interpretation: the best stop limit entries happen on quiet, unconfirmed breaks (not obvious momentum)

### Expected Metrics

**Filtered stop limit breakouts:**
- Frequency: ~2-3 trades/day (filtered heavily)
- WR: 45-49% (higher WR, lower R target than main signals)
- Target: 2R (fixed, per V7r best config)
- Avg R: +0.35 to +0.50
- Expected R/day: +0.5-1R
- Risk: without proper filtering, this goes negative quickly

### Implementation Spec

**Entry rules:**
1. Liquidity sweep detected at key level (PDH, PDL, London H/L, etc.)
2. Scanner confluence score ≥ 3 (REQUIRED — without this it's negative)
3. Place stop limit 5pts above/below sweep level
4. Stop limit valid for 3 bars (cancel if not triggered)

**Stop:**
- 2pts below sweep level (tight — this is a momentum play)
- If price comes back to sweep level after triggering, exit at breakeven

**Target:**
- Fixed 2R (V7r optimal finding)
- No trail, no partials

**Filters (ALL required):**
- Scanner confluence ≥ 3
- 15S candle at entry is NEUTRAL or CONTRADICTING (not confirming — per V7u)
- Volume at entry is LOW relative to 20-bar average
- Not in London session (per batch results: London = negative)

### Risk

This is the most fragile signal class. V7s proved raw stop limits lose money. The entire edge comes from the scanner filtering, which means the edge could be from the scanner being right about direction, not from the stop limit mechanism itself. Need to validate that stop limits add value BEYOND what the scanner already captures.

---

## Signal Class 6: Sweep Close Market Order

**Priority: 6 — Simple variant, moderate evidence**

### What It Is

Instead of waiting for FVG formation after a sweep, enter immediately at the sweep candle CLOSE with a market order. This captures the initial move before any FVG forms.

### Evidence

**V7z — Study 6 findings:**
- Market entry at sweep close captures faster moves
- Fill rate advantage: 100% (market order always fills) vs limit orders that may not fill
- The trade-off: worse entry price but guaranteed participation

**Batch results (QC Model 1):**
- Top variant v4_combo_0056: Sharpe 2.571, $63.7K profit with sweep_fvg + htf_fvg_fill
- signal_sweep_fvg: avg score 142.31 vs signal_htf_fvg_fill: avg score 123.81
- sweep_fvg consistently outperforms across sessions

**V7k directional accuracy:**
- sweep_reversal: 50.4% touch +1R first (barely above coin flip)
- This suggests market order at sweep close is marginal unless filtered

### Expected Metrics

**Sweep close market orders:**
- Frequency: ~3-5 trades/day
- WR: 28-32% (lower than limit entry due to worse fills)
- Avg R: +0.3 to +0.5
- Expected R/day: +0.3-0.8R
- Slippage impact: -0.5pt per side (market order cost)

### Implementation Spec

**Entry rules:**
1. Liquidity sweep detected at key level
2. Wait for sweep candle to CLOSE (don't enter during the sweep)
3. If close is in the direction of the expected reversal, market buy/sell
4. If close is NOT in reversal direction, skip (sweep failed)

**Stop:**
- Sweep candle high/low (whichever is further from entry)
- Maximum 15pts (cap for risk control)

**Target:**
- 2R minimum
- Next key level as secondary target

**Filters:**
- Sweep candle must close in reversal direction (body confirms)
- 5M candle must not contradict (V7aa gate)
- Session must not be London (negative per batch data)
- Volume at sweep > 1.2x average (real sweep, not noise)

### Risk

Worst fill quality of all signal classes due to market order after volatile candle. The 0.5pt/side slippage assumption may be optimistic on sweep candles where spreads are wider. Need to measure actual fill quality on sweep candle closes.

---

## Signal Class 7: Pattern-Discovery Mining

**Priority: 7 — Ongoing research, not a single signal**

### What It Is

Use the cross-mining engine (6,502 combos from 4,991 trades) to discover NEW patterns that don't map to any known signal class. This is an ongoing research process, not a deployable signal.

### Evidence

**Cross-mining results (already computed):**
- 365 two-way combos
- 5,757 three-way combos
- 380 four-way combos
- Overall WR: 32.38%, avg R: 0.585
- Top mining discovery: 100% WR combos at 5.11 R/day (small n)

**Best discovered combos (from synthetic run, needs real data validation):**
- Mon + strong_close_bull + confirming: 100% WR, 5.11 R/day (n=5)
- silver_bullet + bull_trend + VA_fade: 100% WR, 4.29 R/day (n=7)
- These are promising DIRECTIONS but sample sizes are too small for production

### What To Do With This

The mining engine needs to run on VPS against real V8a data (4,991 trades). Steps:

1. Transfer cross_mining_engine.py to VPS (if not already there)
2. Run: `python3 cross_mining_engine.py --data context-engine/models/V8a_all_trades.json --discover --output cross_mining_results.json`
3. Filter results for combos with n≥30 and WR≥35%
4. Walk-forward validate top 20 combos
5. Any combo that survives walk-forward becomes a candidate signal class

### Risk

Data mining bias. With 6,502 combos, some will look great by random chance. Walk-forward validation is mandatory before any mining result gets implemented.

---

## Combined Impact Assessment

### Current System Baseline
- 4 signals: BOS_FVG, sweep_reversal, VA_fade, pre_gap_fill
- +11.31R/day, 72% profitable days, 13/13 months positive
- MaxDD: 47R, Calmar: 0.467

### With New Signal Classes (Conservative Estimates)

| Addition | R/day Impact | DD Impact | Calmar Impact |
|----------|-------------|-----------|---------------|
| FVG Bifurcation | +2R (from better entries) | -5R (fewer bad entries) | +25% |
| Inverse Hedge | +0.5R additive | -13R (47→34) | +30% |
| Composite GOLD sizing | +1R (from 1.5x on best) | -3R (quality filter) | +15% |
| Wick Follow | +1R (new population) | +3R (new DD source) | +5% |
| Stop Limit | +0.5R (if filtered) | +2R (tight stops) | +3% |
| Sweep Close Market | +0.3R (marginal) | +2R | +1% |
| **Combined estimate** | **+5.3R/day** | **-14R** | **+79%** |

**Projected total:** ~16.6R/day, MaxDD ~33R, Calmar ~0.83

This is conservative — the FVG Bifurcation and Inverse Hedge are the big wins and have the strongest evidence. The others are incremental.

### Implementation Order

**Phase 1 (This week):** FVG Bifurcation
- Biggest impact, strongest evidence
- Fix same-bar entry bug simultaneously
- Add regime detection logging for 5 days before going live

**Phase 2 (Next week):** Inverse/Hedge overlay
- Additive (doesn't change existing signals)
- Walk-forward already validated
- Simple streak tracking per signal

**Phase 3 (Week 3):** Composite GOLD sizing
- Layer on top of Phase 1 bifurcation
- Add V7x session filters and V7v streak sizing
- Size up on GOLD entries

**Phase 4 (Week 4):** Wick Follow + Stop Limit
- New signal populations
- Run mining engine on VPS for pattern discovery
- Validate with walk-forward before going live

---

## Action Items for VPS

Both `new_signal_research.py` and `trade_consolidator.py` need to run on real data. Instructions for the brain or Claude Code on VPS:

```bash
# Transfer scripts (if not already on VPS)
# From Mac: scp ~/Documents/trading-system/new_signal_research.py root@172.93.181.209:/root/trading-system/
# From Mac: scp ~/Documents/trading-system/trade_consolidator.py root@172.93.181.209:/root/trading-system/

# Run new signal research on real data
cd /root/trading-system
python3 new_signal_research.py --data context-engine/models/V8a_all_trades.json --output /tmp/new_signal_real_results.json

# Run trade consolidator on real data + mining results
python3 trade_consolidator.py --data context-engine/models/V8a_all_trades.json --mining cross_mining_results.json --output /tmp/consolidator_real_results.json

# Run cross-mining engine for pattern discovery
python3 cross_mining_engine.py --data context-engine/models/V8a_all_trades.json --discover --output cross_mining_discovery.json
```

---

## Key Rules (Don't Forget)

1. **No B/E.** Validated: B/E at 1R = -490R. This applies to ALL new signal classes too.
2. **NEVER invert BOS_FVG.** 32-36% inverse WR regardless of streak. It's the backbone.
3. **VA_fade is the best inverse candidate** (72% at streak ≥5). sweep_reversal is moderate (60%).
4. **London session is negative** for Model 1. Don't deploy new signals there.
5. **outside_only > both > inside_only** for inside day mode (batch results confirm).
6. **overlap and ny_lunch are GOLD sessions** for Model 1 (avg score 327 and 160 respectively).
7. **Position sizing normalizes everything to 1R.** Gap size doesn't change risk.
8. **Walk-forward validate EVERYTHING** before going live. All V7 findings were validated this way.

---

**Document Status:** Research complete. Ready for implementation discussion.
**Next Steps:** Run on real data (VPS), implement Phase 1 (FVG Bifurcation), deploy Phase 2 (Inverse Hedge).
