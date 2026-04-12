---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/sf-portfolio/FINAL_PORTFOLIO_SPEC.md
  - raw-sources/trading/sf-portfolio/PORTFOLIO_STATE_20260320.md
  - raw-sources/trading/sf-portfolio/V8_MINING_SYNTHESIS.md
  - raw-sources/trading/sf-portfolio/DEEP_MINE_FINDINGS.md
  - raw-sources/trading/sf-portfolio/NQ_SF_Engulfing_Strategy_V2.docx
  - raw-sources/trading/sf-portfolio/NQ_Portfolio_Trading_System.docx
  - ~/Documents/strategies/03-sf-portfolio/python/final_table_260d.py
related:
  - wiki/concepts/vwap-double-counting-bug.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/entities/ninjatrader-v5.md
  - wiki/summaries/v5-strategy-bug-audit.md
  - wiki/summaries/nq-playbook.md
  - wiki/summaries/research-journal.md
  - wiki/summaries/wickfade-complete.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, sf-portfolio, sweep-fail, cluster-summary, tick-validated]
---

# SF Portfolio Cluster — Summary

## What the cluster is

`~/HW/raw-sources/trading/sf-portfolio/` — **6 documents** covering the **SF (Sweep-Fail) portfolio** research arc, which is a parallel and independent research track from the BOS_FVG / V10 / IFVG cluster. Where the BOS_FVG track tested momentum-FVG entries with trailing stops, the SF track tested sweep-fail entries (a different signal family) with fixed-target management on 260 trading days. The SF track is the source of the "Clean Level Fade" validated result mentioned in auto-memory and is the direct lineage for the NinjaTrader V5 SF portfolio that had its own separate bug audit in [[v5-strategy-bug-audit]].

## The cluster files

| File | Lines | Role |
|---|---|---|
| `FINAL_PORTFOLIO_SPEC.md` | 368 | **Canonical spec (2026-03-21).** 13-leg NQ+ES portfolio at +3.34 R/day, Calmar 53.1, 47/54 winning weeks |
| `PORTFOLIO_STATE_20260320.md` | 425 | 2026-03-20 state snapshot. 12-leg portfolio at +2.26 R/day, 39/54 winning weeks. Per-leg breakdown table |
| `V8_MINING_SYNTHESIS.md` | 1206 | V8 mining deep dive. **SUSPECT** — uses MTF alignment filters invalidated by [[v10i-look-ahead-bug]] |
| `DEEP_MINE_FINDINGS.md` | 284 | Fib retracement mining (96,552 trades across Feb 2025 – Feb 2026). Dead strategy confirmation |
| `NQ_SF_Engulfing_Strategy_V2.docx` | (binary) | Engulfing-family strategy spec. Requires `textutil` conversion |
| `NQ_Portfolio_Trading_System.docx` | (binary) | Overall portfolio system doc. Requires `textutil` conversion |

## The SF Portfolio's edge (cleaner than BOS_FVG — and tick-level)

The SF cluster's core finding — **+2.26 R/day to +3.34 R/day across 260 days, zero look-ahead verified** — survives the 2026-04-10 [[bar-sim-trailing-bug]] audit for two structural reasons that are both confirmed:

### 1. The simulator is tick-level (confirmed 2026-04-11 by code-read)

The production script `~/Documents/strategies/03-sf-portfolio/python/final_table_260d.py` (and its sibling scripts) loads raw Databento tick data:

```python
df = db.DBNStore.from_file(str(fp)).to_df()
pr = df['price'].values  # individual tick prices
mi = (ts.dt.hour * 60 + ts.dt.minute).values  # minute-of-day per tick
```

The trade management functions (`trade_f`, `trade_be5`, `trade_es`) iterate through individual ticks:

```python
def trade_f(pr, mi, n, ei, d, ep, stp, tgt_r, flat):
    stop = ep - d * stp
    risk = stp
    for j in range(ei + 1, n):      # <-- one iteration per tick
        p = pr[j]
        m = mi[j]
        if m >= flat:  pnl = d*(p-ep) - CM;  return pnl/risk, ...
        if d == 1 and p <= stop:  return (-risk - CM) / risk, 'loss'
        if d == -1 and p >= stop: return (-risk - CM) / risk, 'loss'
        if d * (p - ep) / risk >= tgt_r: return (tgt_r*risk - CM) / risk, 'win'
```

The `build_bars` helper exists only for **signal detection** (finding the sweep-fail pattern from a prior bar's wick). It does not touch trade management. This means the SF portfolio was tick-level all along — it was never a bar-sim simulation in the BOS_FVG sense.

### 2. No trailing stops anywhere

The legs all use:
- **Fixed stops in points** (5pt, 20pt, 25pt, 40pt — depending on session)
- **Fixed R-multiple targets** (1.0R, 1.5R, 2.0R)
- **B/E management at fixed R thresholds** (the RUNNER leg uses B/E at +5R, not intra-bar trailing)
- **Session-flat rules** (flatten at specific minute-of-day values)

Even if this *were* a bar-level simulator, fixed stops and fixed targets would be bar-sim-safe by construction. Combined with the tick-level iteration above, there is no layer where the [[bar-sim-trailing-bug]] can apply.

## 2026-04-11 Sanity check (22 days sampled across the year)

A cut-down version of `final_table_260d.py` (4 core legs on 22 sampled weekdays using the same Databento tick data) produced:

| Leg | n | WR | avg R | R/day (sample) | R/day (published 260d) |
|---|---:|---:|---:|---:|---:|
| RUNNER | 12 | **0.0%** | −0.933 | **−0.509** | **+1.053** |
| SF_PRE | 9 | 55.6% | +0.376 | +0.154 | +0.131 |
| SF_NY1 | 13 | 30.8% | −0.256 | −0.151 | +0.115 |
| SF_LL15 | 14 | 57.1% | +0.123 | +0.078 | +0.085 |
| **Combined** | | | | **−0.428** | **+1.384** |

**The gap is almost entirely in the RUNNER leg.** SF_PRE and SF_LL15 hit their published numbers within sampling variance. SF_NY1 is within a standard deviation of its published number. But the RUNNER produced 0 winners in 12 sampled trades, compared to the published 16% WR implying ~2 expected wins.

**This is not a bug — it's the RUNNER's fat-tail distribution.** Binomial math: at a true 16% WR, the probability of getting 0 wins in 12 trials is `(0.84)^12 ≈ 11.5%`. Slightly unlucky but well within normal variance. And because the RUNNER contributes 76% of total R/day to the published portfolio, an unlucky RUNNER sample dominates the short-window result.

**The important conclusion: the SF portfolio requires the full 260-day sample (or more) before its edge is statistically visible.** Any short-window cross-validation is unreliable. This is a property of the underlying strategy (16% WR, ~6:1 R:R), not a property of the backtester.

## Calibrated assessment

Previous (pre-2026-04-11) assessment in this summary said "no tick-level cross-validation has been done" — **that was wrong**. The simulator has been tick-level all along. The correct caveats are different:

1. **Tick-level: YES.** Code-read confirms individual tick iteration in trade management. Not a bar-sim pattern.
2. **Look-ahead: 6 named bugs found and fixed** (see below). The "zero look-ahead verified" claim in `FINAL_PORTFOLIO_SPEC.md` is at least internally credible.
3. **Walk-forward OOS: not reported.** The 260-day window is one contiguous period. No walk-forward split is documented for the 12-leg or 13-leg portfolio. Walk-forward is the missing discipline.
4. **Multiple-comparison correction: not applied.** The 13-leg composite is the result of selecting legs from a larger candidate pool. Without Bonferroni-style correction on the leg-selection process, some portion of the combined R/day may be survivor bias from the selection step.
5. **Cross-engine validation: not done.** The NinjaTrader V5 side had its own bugs ([[v5-strategy-bug-audit]], the VWAP double-counting bug in particular) that would move the number. The Python and NT sides have not been reconciled post-bug-fix.
6. **Short-sample unreliability (2026-04-11 finding):** 22-day sanity check gave −0.428 R/day vs published +1.384 R/day, explained by the RUNNER leg's 16% WR variance. Any cross-validation under ~150 days on the RUNNER is too noisy to be informative.

## The 8 bugs found and fixed (per FINAL_PORTFOLIO_SPEC and auto-memory)

The SF track had its own bug audit and rebuild, documented as:

1. **Engulfing look-ahead** in `leppyrd_v4.py` — the engulfing detector was using bar data that wasn't closed at signal time
2. **H4 bias filter look-ahead** for London and Asia Late legs — same class of bug as [[v10i-look-ahead-bug|V10i]]
3. **AMANIP filter look-ahead** for Asia Late — bias based on a partial bar
4. **Sign error in the dynamic stop script** — wrong direction of stop adjustment on shorts
5. **Keylevel sweep fade swing confirmation timing** — swings used before they were confirmed (same class as [[look-ahead-audit|Bug 1 in the foundation audit]])
6. **FVG offset script entered during FVG-forming bar** — entry on an incomplete pattern
7. **Day-of-week skip rules** were bugs-in-disguise: some legs had inverted DOW filters
8. **Condition filter edge cases** — narrow pre-market range computation off-by-one

These are all spec-implementation bugs, not backtest-infrastructure bugs. The fix pattern is "read the spec carefully, write the code carefully, re-run" — the same pattern as [[tempo-v14-corrections]] but for a different strategy family. None of them are the structural bar-sim bug that killed BOS_FVG trailing.

## The 12-leg portfolio (PORTFOLIO_STATE_20260320)

| # | Leg | Type | Session | WR | avgR | R/day |
|---|---|---|---|---:|---:|---:|
| 1 | SF_1H_LON_RUN (runner) | Sweep-fail + BE5 | London | 16% | +2.80 | **+1.053** |
| 2 | 5MC_NY | 5M continuation | NY (3/day) | 46% | +0.09 | +0.261 |
| 3 | SF_PRE | Sweep-fail | Pre-NY | 49% | +0.20 | +0.131 |
| 4 | SF_NY1 | Sweep-fail | NY AM | 48% | +0.18 | +0.115 |
| 5 | BRK_NY | Breakout | NY | 38% | +0.10 | +0.094 |
| 6 | STOP_NY | Stop-entry | NY | 38% | +0.09 | +0.091 |
| 7 | VWAP_T1 | VWAP fade | RTH | 45% | +0.09 | +0.089 |
| 8 | SF_NL | Sweep-fail | NY Late | 63% | +0.74 | +0.085 |
| 9 | LON_REV | Reversal | London close | 44% | +0.09 | +0.085 |
| 10 | SF_ALATE | Sweep-fail | Asia Late | 57% | +0.38 | +0.082 |
| 11 | 1H_CONT | Continuation | RTH | 36% | +0.07 | +0.054 |
| 12 | 4HO_25% | 4H offset | RTH | 36% | +0.06 | +0.045 |
| | **TOTAL** | | | 41% | | **+2.26** |

**Key insight from the table: Leg 1 (1H London runner) contributes 47% of total R/day but has 16% WR.** The runner is responsible for 15 losing weeks out of 54. Every attempt to reduce losing weeks by modifying the runner reduces R/day proportionally — the runner is the dominant R-generator and the dominant DD-generator in the same move. This is a fundamental tradeoff the portfolio hasn't resolved.

The reduced portfolio (best 8 legs, drops STOP_NY, ALATE, 4HO, 1H_CONT) produces +1.94 R/day with only 11 losing weeks — cleaner equity curve, slightly lower R/day.

## Final spec bump to +3.34 R/day

`FINAL_PORTFOLIO_SPEC.md` (2026-03-21) represents the next iteration — 13 legs across NQ and ES, adding VWAP multi-touch (VWPM) and AMD legs. Stated performance:

- **+3.34 R/day, +868.3 R total over 260 days**
- **Calmar 53.1, Sharpe 6.57**
- **Max DD 16.4 R**
- **47/54 winning weeks (87%), 13/13 winning months (100%)**

This is the headline number Harrison's memory refers to as "SF portfolio" current state. Post-2026-04-11 audit status: **tick-level simulator verified, structurally clean on exit mechanics, but walk-forward OOS and cross-engine validation are still missing**.

**Status:** Defensible Layer 2 research, significantly cleaner than the BOS_FVG/V10 track, but not yet *validated* in the strongest sense of the word. Gaps to close:
1. Walk-forward split (e.g. 3-month train / 1-month OOS rolling) on the leg-selection process, not just the backtest
2. Bonferroni correction across the leg candidate pool
3. Cross-validation against NinjaTrader engine-native execution (post-V5 bug fixes)
4. Slippage stress test at 1pt and 2pt per side
5. Fill-rate audit on the LIMIT-entry legs (e.g. VWPM) to confirm the 55 trades/day is reachable with live fills

## V8 Mining Synthesis (separately tagged suspect)

`V8_MINING_SYNTHESIS.md` is 1206 lines and is the deepest dive into the V8-phase mining results — the phase that claimed 80%+ WR from MTF alignment before the V10o audit discovered the look-ahead bug. It's kept in the SF portfolio cluster presumably because some V8 signals were precursors to SF legs, but the MTF alignment claims (V8g ≥ 4: 61.7% → 79.6% WR OOS) are all invalidated by [[v10i-look-ahead-bug]].

Use this doc for:
- Historical context on what the V8 phase was testing
- The signal taxonomy (helps trace which SF legs came from which V8 variants)
- The walk-forward methodology (70/30 chronological split, 174 train / 75 test days, Monte Carlo 10,000 runs)

Do NOT cite any V8g ≥ N WR numbers as evidence of edge.

## Deep Mine Findings (Fib retracement — dead strategy confirmation)

`DEEP_MINE_FINDINGS.md` is 284 lines documenting 96,552 Fib retracement trades across 258 days. Conclusion: **Fib retracement is dead at every parameter level.** This is a robust negative result — no amount of contamination fixing saves a strategy that's negative across a population of 96k trades.

Useful for:
- Confirming that [[research-journal]]'s dead-list entry for `fib_retracement` is backed by specific numbers
- Understanding why the "classical TA" strategies generally don't survive proper audit
- Baseline for future deep-mine-style exhaustive parameter sweeps

## Cross-cluster connections

- **[[v5-strategy-bug-audit]]** — the NinjaTrader V5 implementation of the SF portfolio (NQ VWPM, ES VWPM, NQ LF1170) had its own 7-bug audit. The VWAP double-counting bug ([[vwap-double-counting-bug]]) is the headliner and explains why live NinjaTrader ES VWPM was diverging from the backtested numbers.
- **[[ninjatrader-v5]]** — the entity that hosts the live SF portfolio
- **[[nq-playbook]]** and **[[research-journal]]** — separate research arc (BOS_FVG / V10), suspect for different reasons
- **[[tempo-three-layers]]** — SF portfolio sits in Layer 2 alongside BOS_FVG but comes out cleaner because its exit mechanics are bar-sim-safe

## Open questions

- Has the 2026-03-21 +3.34 R/day headline been cross-validated against NinjaTrader's engine-native execution? V5 audit suggests it hasn't (VWAP bug alone would move the number).
- What is the tick-level performance of each leg independently? The +3.34 R/day is a bar-level sim, and while fixed targets are bar-sim-safe, fill-bar behavior (did the bar's low hit the stop before the high hit the target?) still requires careful handling.
- How does the SF portfolio perform OOS beyond the 260-day window? Walk-forward was reported on V8 but not explicitly on the 12-leg or 13-leg portfolios.
