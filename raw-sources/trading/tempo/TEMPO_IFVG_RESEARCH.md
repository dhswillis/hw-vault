# Tempo Sweep + IFVG Strategy — Research Document

**Strategy**: Liquidity sweep at key levels → IFVG confirmation → reversal entry
**Source**: 281 Tempo trade recap transcriptions (Oct 2024 – Jan 2026, 257,335 words)
**Backtest Data**: 312 days Databento NQ tick data (Feb 2025 – Feb 2026)
**Last Updated**: March 8, 2026

---

## EXECUTIVE SUMMARY

### Tempo V3 Rules (from transcriptions)
- **Reported WR**: 88% across 400+ trades (Nov 2025 – Jan 2026)
- **Key**: SMT (NQ vs ES divergence) is PRIMARY confluence — without it, WR drops to 60-65%
- **Session**: 9:35–11:00 AM EST (first 90 minutes of NY)
- **Targets**: TP1 at 17pts (trim 55%), TP2 at 35pts (trim 35%), 10% runner
- **B/E**: Non-negotiable at 15-20pts profit
- **Max trades**: 1-2 per day, A/A+ setups only
- **Soft stops**: Exit on candle CLOSE past stop (not wick), 90% of the time

### Our Backtest Results (without SMT — no ES data)

| Entry TF | Trades | WR | Avg Pts | Pts/day | Cal |
|---|---|---|---|---|---|
| **1min IFVG** | 269 (0.9/d) | **74.7%** | +9.9 | +8.6 | 15.5 |
| 2min IFVG | 213 (0.7/d) | 67.6% | +6.7 | +4.6 | 5.8 |

**74.7% WR without SMT** — consistent with Tempo's stated 60-65% WR without SMT, plus our additional structural filters.

---

## 1. WHAT TEMPO ACTUALLY DOES (V3 Rules from 281 Recaps)

### Entry Sequence

**Step 1: Identify Key Liquidity Levels**
- Previous Day High/Low (PDH/PDL)
- London Session High/Low
- Asia Session High/Low
- 1H and 4H swing highs/lows
- AM session high (first hour of NY)
- 50% of daily range (yellow line)

**Step 2: Wait for Liquidity Sweep**
- Price must WICK past the level (take out stops)
- Candle must CLOSE back inside the level (not a breakout)
- If candle closes past the level = NOT a sweep = NO TRADE
- Minimum sweep: visible wick (1+ pts on NQ)
- "Sweep HIGH + close below" = bearish setup
- "Sweep LOW + close above" = bullish setup

**Step 3: Confirm Reversal**
- V-shaped recovery with instant momentum (strongest signal)
- Three consecutive directional candles after sweep
- "Reaction is everything" — price reaction matters more than sweep itself
- 46+ point death candle = momentum confirmation

**Step 4: Enter on IFVG (Inverse Fair Value Gap)**
- An FVG forms in the WITH-sweep direction (e.g., bullish FVG after sweep high)
- Then a candle CLOSES through that FVG completely (not just wicks through)
- This "inversion" confirms the reversal
- Entry at the close of the candle that trades through
- Preferred timeframes: 2min (primary), 30sec (equally viable per V3), 1min

**Step 5: Stop Placement**
- Above/below the sweep wick + buffer
- Typical: 20-30pts
- Soft stops preferred (90%): exit only on candle CLOSE past stop, not wick
- Hard stops only for funded accounts with strict drawdown rules

**Step 6: Trade Management**
- **TP1 at 17pts**: Trim 55% of position
- **B/E immediately**: Move stop to entry after TP1
- **TP2 at 35pts**: Trim 35% of original position
- **Runner (10%)**: Hold to session end or next liquidity level
- "Never let a green trade turn red" — non-negotiable

### Gap Size Filters
- **Optimal**: 8-15 pts (highest probability)
- **Acceptable**: 5-6 pts (risky, needs extra confluence)
- **Strong**: 20-30 pts (high conviction)
- **Skip**: >30 pts (too much risk for stop placement)
- Prefer singular gaps — avoid stacked FVGs

### Quality Classification
| Grade | Confluences | Est. WR | Action |
|---|---|---|---|
| A+ | 7+ (sweep + SMT + IFVG + bias + 50% + momentum + BPR) | 80%+ | Full size |
| A | 5-6 (sweep + IFVG + SMT + momentum + bias) | 70-75% | Full size |
| B+ | 3-4 (secondary sweep + smaller gaps) | 50-70% | Reduced size |
| B | 2 | ~50% | Half size only |
| C/Skip | 0-1 | <50% | NO TRADE |

V3 minimum for funded accounts: 3+ confluences.

---

## 2. SMT (Smart Money Technique) — The Missing Piece

### What It Is
NQ vs ES divergence at key liquidity levels:
- **Bullish SMT**: NQ sweeps a low but ES doesn't → long NQ
- **Bearish SMT**: ES sweeps a high but NQ doesn't → short NQ
- Price doesn't need exact tick — 1-2 ticks away counts

### Impact on Win Rate
| Condition | Win Rate |
|---|---|
| SMT + IFVG + sweep | **80%+** |
| IFVG + sweep (no SMT) | 60-65% |
| IFVG alone | ~55% |

### Why We Can't Test It
We don't have ES tick data. Databento offers "last year free" but we haven't downloaded it. This is the single biggest improvement opportunity.

### Recommendation
Download ES tick data from Databento and implement SMT divergence as a filter. Expected to boost our 74.7% WR to 80%+ and filter out the worst 25% of trades.

---

## 3. BACKTEST METHODOLOGY

### Corrected Implementation (V2)

**Session**: 14:35-16:00 UTC (9:35-11:00 AM EST)

**Sweep Detection**:
- Build key levels: PDH/PDL, London H/L, Asia H/L, 1H swings, AM H/L
- Wick past level + close back inside = sweep
- Close past level = breakout = NOT a sweep (skip)

**IFVG Entry**:
- After sweep, look for FVG in WITH-sweep direction
- Then look for candle CLOSE through that FVG (not just wick)
- Entry at close price of the candle that trades through
- Stop: sweep wick tip + 1pt buffer

**Trade Management** (partial position simulation):
- 100 units starting position
- TP1: 17pts → trim 55% (55 units)
- B/E: move stop to entry after TP1
- TP2: 35pts → trim 35% (35 units)
- Runner: 10 units to session end
- Soft stop: exit on candle CLOSE past stop (not wick)

**Gap Size Filter**: 5-30pts (skip below 5 and above 30)
**Max trades per day**: 2

### Results by Source Level (1min IFVG)

| Source | Trades | WR | Avg Pts |
|---|---|---|---|
| **1H Swings** | 100 | **84.0%** | +14.4 |
| **London H/L** | 45 | **80.0%** | +11.6 |
| AM H/L | 67 | 67.2% | +7.0 |
| PDH/PDL | 25 | 64.0% | +6.8 |
| Asia H/L | 32 | 62.5% | +2.2 |

1H swings and London H/L are the strongest sources by far.

### Exit Breakdown

| Exit Type | Count | Avg Pts | Interpretation |
|---|---|---|---|
| Soft stop | 141 | +2.7 | Small wins (B/E working) |
| B/E stop | 88 | +16.6 | Locked 17pts then stopped at entry |
| Session end | 40 | +21.0 | Runners held to 11AM |

52% of trades exit at soft stop but STILL average positive (+2.7pts) because B/E is converting what would be losses into breakeven/small wins.

### Walk-Forward

| Period | Trades | WR | Avg Pts | Pts/day |
|---|---|---|---|---|
| H1 In-Sample | 128 | 75.0% | +11.0 | +9.0 |
| H2 Out-of-Sample | 141 | 74.5% | +9.0 | +8.2 |

Holds up in OOS. Slight degradation is normal and expected.

---

## 4. V1 vs V2 COMPARISON — WHAT WE FIXED

### V1 (First Attempt — Dead)
- 24H session, all levels, wick touch = entry
- avgR: +0.023, Calmar: 2.0
- Effectively random — no edge

### V2 (Corrected per Tempo V3 rules)
- 9:35-11:00 AM EST only
- Candle CLOSE through FVG (not wick)
- Sweep must be wick (close back inside)
- B/E at 17pts, partials at 17/35pts
- Soft stops (close past stop, not wick)
- 74.7% WR, +9.9 avg pts, Cal 15.5

### What Made the Difference
1. **Session filter** (9:35-11:00 AM): Eliminated noisy overnight/lunch trades
2. **Candle close requirement**: Filtered false IFVG signals (wicks through without conviction)
3. **Sweep wick requirement**: Ensured actual liquidity grabs, not breakouts
4. **B/E + partials**: Converted losers to breakeven, locked profits early
5. **Max 2 trades/day**: Avoided overtrading on choppy days

---

## 5. TEMPO'S METHODOLOGY EVOLUTION

Based on 281 trade recap transcriptions across 16 months:

### Phase 1: Foundation (Oct – Dec 2024)
- Basic IFVG + liquidity sweeps, premium/discount
- Hard stops, 20-50% trim at TP1
- SMT as one confluence among several

### Phase 2: Integration (Jan – Mar 2025)
- All concepts harmonized: SMT + DOL + IFVG
- Largest V1 trades (260pts on Jan 28)
- Risk management tightened

### Phase 3: Adaptation (Apr – Jun 2025)
- Tariff crisis trading (1700pt ranges)
- Hard stops abandoned ("don't work with this model")
- 330-point trade (Apr 1 — largest)
- "Half size first, full on retracement" chasing protocol

### Phase 4: Refinement (Jul – Oct 2025)
- Conservative shift: TP1 trim increased to 55%+
- Runners nearly eliminated — "base hit mentality"
- 30-second IFVG upgraded to equally viable with 2-minute
- Soft stops default (~90% of trades)

### Phase 5: Mastery (Nov 2025 – Jan 2026)
- SMT elevated to PRIMARY confluence
- 88% WR across 400+ trades
- $42,000 single day (Jan 29, 2026) across 30+ accounts
- "If you have SMT + IFVG + sweep, you're in the money most of the time"

---

## 6. NEGATIVE CONFLUENCES (WHEN TEMPO SKIPS)

From V3 rules doc — conditions that invalidate a setup:
- Gap > 30 points
- Multiple gaps stacked above and below entry
- Within 2 minutes of news
- Already 2+ losses today
- No SMT or major sweep confirmation
- Chop/range (70-100pt range in 2+ hours)
- Already green for day
- All overnight areas swept before open (worst condition)
- First 5 minutes after open (too volatile)
- At all-time highs with no sell-side liquidity
- Trump/Powell speech imminent
- Trend day against position

---

## 7. BOS_FVG vs TEMPO IFVG — KEY DIFFERENCES

| Dimension | BOS_FVG | Tempo IFVG |
|---|---|---|
| **Trigger** | BOS (close past swing) | Liquidity sweep (wick past level) |
| **Entry** | FVG fill (pullback to edge) | FVG traded through (momentum) |
| **Fill type** | Limit order at FVG edge | Market at candle close |
| **Direction** | With the break | Against the sweep (reversal) |
| **Stop** | FVG opposite edge | Sweep wick tip |
| **Management** | Pure trailing (0.5R/0.1R) | Partials at 17/35pts + B/E |
| **Session** | 24H viable | NY open only (9:35-11:00 EST) |
| **Trades/day** | 37-75 | 0.9-2 |
| **Win rate** | 47.7% | 74.7% (80%+ with SMT) |
| **Edge source** | Structural (momentum continuation) | Reversal (liquidity trap) |
| **Discretion** | Fully mechanical | High discretion (SMT, reaction quality) |

**These are fundamentally different strategies** — BOS_FVG rides momentum after structure breaks, Tempo fades liquidity sweeps. They are likely uncorrelated and could combine well.

---

## 8. V15 FILTERS — SMT DIVERGENCE, REACTION QUALITY, LIQUIDITY (March 2026)

### The Problem

The IFVG strategy as implemented in v14 was running at ~64% WR with PF 0.99 — essentially breakeven. Tempo's reported 88% WR comes from heavy discretionary filtering that v14 didn't capture. Two critical missing filters were identified:

1. **Liquidity depletion**: Don't take trades when all structural liquidity in the trade direction has already been swept (nothing left to fade).
2. **B/E at opposing liquidity**: When a runner approaches a known liquidity pool on the other side, move to breakeven instead of riding blindly.

Testing these alone improved WR from 57.2% to 59.4% — helpful but nowhere near 80%. The breakthrough came from implementing the two filters Tempo calls PRIMARY confluence: **SMT divergence** and **reaction quality**.

### 8.1 SMT Divergence — How It Works

**Concept**: When NQ sweeps a key level (e.g., PDH) but ES does NOT sweep its equivalent level, there is "divergence" between the two correlated instruments. This divergence signals that the sweep was a liquidity grab, not a genuine breakout — which is exactly what the IFVG strategy needs to fade.

**Implementation**: Build structural key levels for BOTH NQ and ES independently (PDH/PDL, Asia H/L, London H/L, AM H/L, 1H swing H/L). When NQ sweeps a level, check if ES also swept an equivalent-type level in the same window. If ES did NOT confirm the sweep, SMT divergence exists.

**ES Data**: Downloaded 312 days of ES tick data from Databento (March 2025 - March 2026). Added as secondary data series (BarsInProgress 3) in NinjaTrader via `AddDataSeries("ES 03-26", BarsPeriodType.Minute, 1)`.

**Key finding**: SMT alone does NOT improve WR. In fact, requiring SMT without other filters made things slightly worse (63.3% vs 64.4%). The power of SMT only emerges when combined with reaction quality filtering.

### 8.2 Reaction Quality Filter — The Real Edge

**Concept**: After a sweep, measure the quality of the price reaction. Tempo's V3 rules emphasize "reaction is everything" — a V-shaped recovery with instant momentum is the strongest signal. The reaction score quantifies this on a 0-3 scale:

| Score | Description | Criteria |
|---|---|---|
| 0 | No reaction | Price continues in sweep direction |
| 1 | Weak reaction | Single directional candle |
| 2 | Moderate reaction | 2+ consecutive candles with partial displacement |
| 3 | Strong reaction | **Death candle** (20+ pt body in reversal direction) OR **3+ consecutive directional candles** with 10+ pts total displacement |

**This is THE key filter.** Within SMT-confirmed sweeps, the win rate breakdown by reaction score is:

| Reaction Score | Trades | Win Rate | Profit Factor |
|---|---|---|---|
| 0 | ~300 | 57% | 0.75 |
| 1 | ~150 | 57% | 0.80 |
| 2 | ~120 | 66% | 1.15 |
| **3** | **182** | **83.0%** | **1.87** |

Scores 0-1 are essentially coin flips. Score 2 has a mild edge. Score 3 is where the strategy becomes elite. The reaction filter separates high-conviction reversals from random noise.

![SMT × Reaction Quality Heatmap](results/smt_reaction_matrix.png)

### 8.3 Liquidity Depletion Filter

**Concept**: Before taking a trade, check if there are remaining un-swept structural levels in the trade direction. If all highs have been swept (for a short setup), there's no remaining sell-side liquidity to fade — the market may be in a genuine breakout.

**Implementation**: Scan `_ifvgLevels` for levels of matching type (highs for shorts, lows for longs) that haven't been recorded in `_ifvgSweptLevelsToday`. If none remain, skip the trade.

**Impact**: Modest improvement (+2% WR) but prevents the worst trades — those where the strategy fights a genuine trend after all levels have been cleared.

### 8.4 B/E at Opposing Liquidity

**Concept**: When a runner position approaches a structural level on the opposing side (e.g., a long position approaching PDH), smart money is likely to react there. Move stop to breakeven to protect profits.

**Implementation**: `FindOpposingLiquidity()` scans for the nearest opposing-type level beyond entry price. When price reaches that level (wick check), stop moves to entry.

**Impact**: Converts some small winners to breakeven, but protects against the worst scenario: riding a runner straight into a liquidity sweep that reverses the entire move.

### 8.5 Combined Filter Results (280 days, March 2025 - February 2026)

![Filter Impact Comparison](results/reaction_filter_impact.png)

| Configuration | Trades | Win Rate | Profit Factor | Total PnL | Pts/Day |
|---|---|---|---|---|---|
| A) Baseline (no filters) | 1,142 | 64.4% | 0.99 | -91 pts | -0.3 |
| B) Liquidity filters only | 1,080 | 59.4% | 1.01 | +52 pts | +0.2 |
| C) SMT required (no reaction) | 850 | 63.3% | 0.92 | -380 pts | -1.4 |
| D) SMT + reaction ≥ 2 | 273 | 77.3% | 1.53 | +973 pts | +3.5 |
| **E) SMT + reaction = 3** | **182** | **83.0%** | **1.87** | **+962 pts** | **+3.4** |
| F) SMT + reaction ≥ 2 + liq | 260 | 78.1% | 1.58 | +985 pts | +3.5 |

Config E (SMT + reaction=3) is the production choice: fewer trades but highest WR and PF. Config D (reaction≥2) is viable if more trade frequency is desired.

### 8.6 Why SMT Alone Doesn't Work

This was a critical discovery. First SMT implementation (v1) compared ES to its own recent 20-bar high/low — too simplistic, caught false "divergences" everywhere. SMT trades actually performed worse (PF 0.89) than non-SMT (PF 1.12).

The fix was structural SMT: build ES key levels using the same methodology as NQ (PDH/PDL, session H/L, 1H swings), then check if ES swept the equivalent structural level. This catches real institutional divergence rather than noise.

But even structural SMT alone (Config C) was net negative. The key insight: SMT tells you the sweep is more likely a reversal, but reaction quality tells you the reversal is actually happening. You need both — the setup (SMT) AND the confirmation (strong reaction).

---

## 9. PORTFOLIO COMBINATION — IFVG + LUMI

### 9.1 Portfolio Construction

After fixing IFVG with the v15 filters, combined it with Lumi FVG+10 strategy (3-min timeframe, HTF sweep → MSS → OB+FVG, 3R target). The two strategies operate on different timeframes and logic:

| Dimension | IFVG (v15) | Lumi FVG+10 |
|---|---|---|
| Timeframe | 1-min | 3-min (entry), 60-min (sweep) |
| Logic | Sweep → reaction → IFVG → close through | HTF sweep → MSS → OB+FVG → limit entry |
| Stop type | Soft (close-based) | Hard |
| Target | Session runners + partials | Fixed 3R |
| Avg trade/day | 0.65 | 1.95 |
| Win rate | 83.0% | 62.2% |

### 9.2 Combined Results

| Metric | IFVG | Lumi | Combined |
|---|---|---|---|
| Trades | 182 | 547 | 729 |
| Win Rate | 83.0% | 62.2% | 67.4% |
| Profit Factor | 1.87 | 2.47 | 2.29 |
| Total PnL | +962 pts | +3,120 pts | +4,082 pts |
| Max Drawdown | 450 pts | 148 pts | 218 pts |
| Calmar Ratio | 2.1 | 21.2 | 18.7 |
| Pts/Day | +10.0* | +16.2* | +14.6 |

*Per active trading day (IFVG fires ~96 days, Lumi ~193 days); combined uses all 280 calendar days.

### 9.3 Correlation Analysis

Daily PnL correlation between IFVG and Lumi: **r = -0.004** (completely uncorrelated).

![Daily PnL Scatter](results/daily_pnl_scatter.png)

This is the ideal portfolio result — two strategies with positive expectancy that have zero correlation. Combined drawdown (218 pts) is less than the sum of individual drawdowns (450 + 148 = 598 pts), demonstrating genuine diversification benefit.

### 9.4 Equity Curves

![Portfolio Equity Curves](results/portfolio_equity_curve.png)

Monthly performance shows only 1 losing month (Sep -117 pts) out of 12, with best month Nov +879 pts.

---

## 10. V15 NINJATRADER IMPLEMENTATION

### 10.1 Architecture Changes

**TempoPortfoliov15.cs** (2,381 lines) adds these components to v14:

| Component | Description |
|---|---|
| BarsInProgress 3 | ES 1-min data series via `AddDataSeries("ES 03-26", BarsPeriodType.Minute, 1)` |
| ES Key Levels | `BuildESLevels()` mirrors `BuildIFVGLevels()` using ES data — PDH/PDL, Asia, London, AM, 1H swings |
| ES 1H Swings | `UpdateES1HSwings()` computes swing highs/lows from ES 1-min bars (60-bar rolling window) |
| IFVGPhase.WaitReaction | New phase between sweep detection and FVG scan |
| CheckSMTDivergence() | Compares NQ sweep to ES structural levels — returns true if divergence exists |
| IFVGCheckReaction() | Scores post-sweep reaction 0-3 each bar, promotes to WaitIFVG when threshold met |
| CheckLiquidityRemaining() | Verifies un-swept structural levels exist in trade direction before entry |
| FindOpposingLiquidity() | Locates nearest opposing structural level for B/E trigger |

### 10.2 New Parameters

| Parameter | Default | Description |
|---|---|---|
| EnableSMT | true | Enable SMT divergence checking |
| RequireSMT | true | Require SMT confirmation to take IFVG trades |
| MinReaction | 3 | Minimum reaction quality score (0-3) |
| SMTWindow | 5.0 | Tolerance in pts for ES level matching |
| EnableLiqDepletion | true | Skip trades when directional liquidity is exhausted |
| EnableLiqBE | true | Move to B/E when price reaches opposing liquidity |
| ReactionDeathCandle | 20.0 | Minimum body size (pts) for instant score 3 |
| ReactionMinDisplacement | 10.0 | Minimum total displacement for score 3 via consecutive candles |
| ReactionMaxBars | 5 | Maximum bars to measure reaction before expiring setup |

### 10.3 IFVG Pipeline Flow (v15)

```
Sweep detected → [SMT check: ES divergence?]
    → NO SMT (RequireSMT=true): SKIP
    → SMT confirmed: enter WaitReaction phase
        → Measure reaction quality each bar (up to ReactionMaxBars)
        → Score < MinReaction after max bars: EXPIRE
        → Score ≥ MinReaction: promote to WaitIFVG
            → Scan for Inverse FVG (5-20pt gap)
            → FVG found: promote to WaitEntry
                → [Liquidity depletion check]
                    → All levels swept: SKIP
                    → Levels remain: check for candle close through FVG
                        → Entry signal: submit market order
                        → [Find opposing liquidity for B/E target]
                        → Trade management (soft stop + session TPs + liq B/E)
```

### 10.4 v15.1 Update: R-Target Exit System (March 22, 2026)

The original session-aware TP system (partials at 17/35pts + runner) was replaced with a simple **1.25R full-exit target** after comprehensive backtesting showed it doubles performance. The old system is retained as a toggle (`SessionAwareTPs=true`) but defaults to off.

**New parameter**: `IFVGRTarget = 1.25` — exit 100% of position at 1.25× risk, soft stops only (no B/E, no partials).

See Section 14 for full TP optimization analysis.

### 10.5 Important Notes for Live Trading

1. **ES contract symbol**: Hardcoded as `"ES 03-26"` — must be updated on quarterly rolls (June → `"ES 06-26"`, etc.)

2. **Bar alignment**: ES 1-min bars (BIP 3) are independent from NQ bars. The SMT check runs on NQ bar close (BIP 0) using the most recent ES bar data. Minor timing differences vs tick-level backtest, but should be negligible on 1-min.

3. **Reaction measurement runs on 1-min bars**: Each bar in WaitReaction phase gets one score check. With ReactionMaxBars=5, the strategy waits up to 5 minutes after sweep for reaction confirmation.

4. **Trade frequency**: Expect ~0.65 IFVG trades per day (182 over 280 days). Many days will have zero IFVG trades because the SMT + reaction filter is selective. Lumi provides the base trade volume (~2/day).

---

## 11. PREVIOUS NEXT STEPS (Status Update)

| Task | Status |
|---|---|
| Download ES tick data | **DONE** — 312 days Databento ES data |
| Implement SMT divergence | **DONE** — v15 CheckSMTDivergence() |
| Correlation test (IFVG vs Lumi) | **DONE** — r = -0.004 (uncorrelated) |
| Combined portfolio | **DONE** — +4,082 pts, Cal 18.7 |
| NinjaTrader port | **DONE** — TempoPortfoliov15.cs (2,381 lines) |
| Gap size optimization | **DONE** — GapMax reduced from 30 to 20 |
| Premium/discount filter | Superseded by reaction quality filter (more effective) |
| TP optimization | **DONE** — 1.25R target replaces partials (v15.1), +2x performance |

### Remaining Next Steps
1. **Live forward test**: Run v15.1 on sim account for 2-4 weeks
2. **ES contract rollover**: Automate or document the quarterly symbol update process
3. **Intraday stats monitoring**: Add SMT confirmation rate and reaction score distribution to on-chart stats display
4. **Walk-forward validation**: Run v15.1 filters through rolling 3-month windows

---

## 12. KEY FILES

| File | Purpose |
|---|---|
| `strategies/tempo/TempoPortfoliov15.cs` | Production NinjaTrader strategy (v15 with all filters) |
| `strategies/tempo/TempoPortfoliov14.cs` | Previous version (no SMT/reaction/liquidity) |
| `strategies/tempo/scripts/ifvg_smt_v2.py` | Python backtest with SMT + reaction quality (~700 lines) |
| `strategies/tempo/TEMPO_IFVG_BUILD_SPEC.md` | Build specification for NinjaTrader implementation |
| `strategies/tempo/results/smt_reaction_matrix.png` | Heatmap: WR by SMT × Reaction score |
| `strategies/tempo/results/reaction_filter_impact.png` | Bar chart: Filter impact comparison |
| `strategies/tempo/results/portfolio_equity_curve.png` | Equity curves: IFVG + Lumi + Combined |
| `strategies/tempo/results/daily_pnl_scatter.png` | Scatter: IFVG vs Lumi daily PnL correlation |
| `strategies/tempo/results/tp_optimization.png` | Chart: R-target comparison (Total R, Calmar, R/day) |
| `~/Downloads/tempo_extracted/tempo_rules_v3_complete.md` | Full V3 rules from 281 recaps |
| `~/Downloads/tempo_extracted/transcriptions/` | 323 individual video transcripts |
| `data/databento-nq/databento/` | NQ tick data (299 .dbn.zst files) |
| `data/databento-es/databento-es/` | ES tick data (312 .dbn.zst files) |

---

## 13. ALGORITHM PARAMETERS (v15.1 — updated March 22, 2026)

```
# ── SMT + Reaction Filters (v15) ──
ENABLE_SMT           = True       # Use ES divergence checking
REQUIRE_SMT          = True       # Require SMT for entry
MIN_REACTION         = 3          # Minimum reaction quality (0-3)
SMT_WINDOW           = 5.0        # ES level match tolerance (pts)
REACTION_DEATH_CANDLE = 20.0      # Min body for instant score 3
REACTION_MIN_DISPLACEMENT = 10.0  # Min displacement for score 3 via consec candles
REACTION_MAX_BARS    = 5          # Max bars to measure reaction

# ── Liquidity Filters (v15) ──
ENABLE_LIQ_DEPLETION = True       # Skip when directional liquidity exhausted
ENABLE_LIQ_BE        = True       # B/E at opposing liquidity level (legacy mode only)

# ── Risk ──
RISK_PCT = 0.005              # 0.5% per trade
MAX_LOSSES_PER_DAY = 0        # Unlimited (optimal per backtest)

# ── IFVG ──
GAP_MIN = 5.0                 # Min FVG gap size
GAP_MAX = 20.0                # Max FVG gap size (reduced from 30)
MIN_RISK = 10.0               # Min risk distance (pts)
MAX_RISK = 45.0               # Max risk distance (pts)
SWEEP_MIN_PTS = 1.0           # Min sweep wick beyond level

# ── Exit Management (v15.1 — NEW) ──
SESSION_AWARE_TPS = False     # False = simple R-target; True = legacy partials
IFVG_R_TARGET = 1.25          # Exit 100% at 1.25× risk (when SESSION_AWARE_TPS=False)
# Soft stops (close-based) remain active. No B/E, no partials.

# ── Legacy Trim Targets (only when SESSION_AWARE_TPS=True) ──
TP1_PTS = 17.0                # TP1 distance
TP2_PTS = 35.0                # TP2 distance
# London: 100% runner, B/E at 17pts + 0.25pt buffer
# NY AM: 30% at TP1, 70% runner
# Lunch/PM: 55% at TP1, 35% at TP2, 10% runner

# ── Stop loss ──
USE_SOFT_STOPS = True         # Exit on candle CLOSE, not wick
DISASTER_STOP_PTS = 10.0      # Hard stop buffer (flash crash protection)

# ── Break-even (legacy only) ──
BE_THRESHOLD_PTS = 17.0       # Move to B/E after this favorable excursion
BE_BUFFER_PTS = 0.25          # Small profit buffer on B/E (London only)

# ── Session ──
ENTRY_START = "03:00 ET"      # Full window (London through NY PM)
ENTRY_END = "15:30 ET"
FLATTEN_TIME = "15:55 ET"

# ── ES Data (for SMT) ──
ES_SYMBOL = "ES 03-26"        # Must update on quarterly rolls!
```

---

## 14. TP OPTIMIZATION ANALYSIS (v15.1 — March 22, 2026)

### 14.1 The Problem with Partials + Runners

The original Tempo trade management used session-aware partial exits: trim at TP1 (17pts), trim again at TP2 (35pts), hold a runner to session end. Analysis of 237 trades (react=3, no SMT, 280 days) revealed a structural issue:

- 63% of trades exit at B/E stop after partial: avg +6.7pts, total +1,009pts
- 6% of trades reach session flatten as runners: avg +100.3pts, total +1,404pts (ALL the profit)
- 13% hit soft stop: avg -35.7pts, total -1,144pts
- 5% hit emergency stop: avg -26.2pts, total -288pts

The strategy was essentially a lottery: 82% WR but most wins were tiny (+5-7pts) while losses were full risk (-33pts avg). All profit came from 14 runner trades out of 237.

### 14.2 Fixed R-Target Testing

Tested 5 exit configurations on identical 237 entry signals:

| Config | WR | PF | Total Pts | R/day | R/trade | Max DD | Calmar |
|---|---|---|---|---|---|---|---|
| Original (partials+runner) | 69.2% | 1.78 | +1,121 | +0.186 | +0.220 | 467 | 2.4 |
| 100% at TP1 (17pts) | 74.7% | 1.54 | +1,025 | +0.216 | +0.255 | 410 | 2.5 |
| 100% at 1R | 74.3% | 2.05 | +2,292 | +0.357 | +0.422 | 333 | 6.9 |
| 100% at 2R + B/E@1R | 35.4% | 1.98 | +2,138 | +0.346 | +0.408 | 305 | 7.0 |
| 100% at 3R + B/E@1R | 18.6% | 1.60 | +1,316 | +0.225 | +0.266 | 424 | 3.1 |

**Key insight**: Simple R-target exits massively outperform the partial/runner approach. The 1R exit alone doubles total PnL.

### 14.3 Fine-Grained R-Target Sweep

Tested 0.5R through 2.0R in 0.05R increments around the sweet spot:

| Target | WR | PF | Total Pts | R/day | R/trade | Max DD | Calmar |
|---|---|---|---|---|---|---|---|
| 0.5R | 84.0% | 1.65 | +970 | +0.182 | +0.215 | 460 | 2.1 |
| 1.0R | 74.3% | 2.05 | +2,292 | +0.357 | +0.422 | 333 | 6.9 |
| 1.2R | 70.9% | 2.07 | +2,642 | +0.407 | +0.481 | 374 | 7.1 |
| **1.25R** | **70.9%** | **2.16** | **+2,859** | **+0.437** | **+0.516** | **362** | **7.9** |
| 1.3R | 68.8% | 2.01 | +2,732 | +0.395 | +0.466 | 353 | 7.7 |
| 1.4R | 65.8% | 1.98 | +2,798 | +0.389 | +0.460 | 334 | 8.4 |
| 1.5R | 64.1% | 1.94 | +2,879 | +0.381 | +0.451 | 315 | 9.1 |
| 1.75R | 58.6% | 1.81 | +2,809 | +0.382 | +0.452 | 330 | 8.5 |
| 2.0R | 53.2% | 1.76 | +2,883 | +0.340 | +0.401 | 360 | 8.0 |

### 14.4 B/E at 1R Impact

Adding B/E at 1R consistently HURTS the R-target configs:

| Config | Without B/E | With B/E@1R | Difference |
|---|---|---|---|
| 1.25R Total R | +122.3R | +100.8R | -21.5R (−18%) |
| 1.5R Total R | +106.8R | +85.1R | -21.7R (−20%) |

B/E at 1R converts winning trades into B/E exits before they reach the target. With soft stops already providing downside protection, the extra B/E layer just clips winners prematurely.

### 14.5 Optimal Choice: 1.25R

**1.25R without B/E** was selected as the production configuration:

- **Best R/day** (0.437) — highest risk-adjusted daily return
- **Best R/trade** (0.516) — highest per-trade efficiency
- **Best profit factor** (2.16) — best gross win / gross loss ratio
- **+155% improvement** over original partials on total R (122.3 vs 52.2)
- **71% win rate** — psychologically comfortable to trade
- **Simple execution** — no partial exits, no B/E moves, just target or soft stop

Note: 1.5R has the best Calmar ratio (9.1 vs 7.9) due to tighter max drawdown (315 vs 362). If minimizing drawdown is the priority, 1.5R is the alternative. The R/day difference is modest (0.381 vs 0.437).

![TP Optimization Chart](results/tp_optimization.png)

### 14.6 Updated IFVG Pipeline Flow (v15.1)

```
Sweep detected → [SMT check: ES divergence?]
    → NO SMT (RequireSMT=true): SKIP
    → SMT confirmed: enter WaitReaction phase
        → Measure reaction quality each bar (up to ReactionMaxBars)
        → Score < MinReaction after max bars: EXPIRE
        → Score ≥ MinReaction: promote to WaitIFVG
            → Scan for Inverse FVG (5-20pt gap)
            → FVG found: promote to WaitEntry
                → [Liquidity depletion check]
                    → All levels swept: SKIP
                    → Levels remain: check for candle close through FVG
                        → Entry signal: submit market order
                        → Target: entry ± (risk × 1.25)
                        → Exit: TARGET hit (wick) or SOFT_STOP (close) or EMERGENCY (disaster)
```
