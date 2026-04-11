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

## The SF Portfolio's edge (why it's cleaner than BOS_FVG)

The SF cluster's core finding — **+2.26 R/day to +3.34 R/day across 260 days, zero look-ahead verified** — survives the 2026-04-10 [[bar-sim-trailing-bug]] audit for a structural reason: **the SF portfolio does not use trailing stops**. The legs all use:

- **Fixed stops in points** (5pt, 20pt, 25pt, 40pt — depending on session)
- **Fixed R-multiple targets** (1.0R, 1.5R, 2.0R)
- **B/E management at fixed R thresholds** (not intra-bar trailing)
- **Session-flat rules** (flatten at specific minute-of-day values)

All of these are bar-sim-safe by construction. A fixed-R target either reaches the target or doesn't; a fixed B/E trigger either reaches the trigger or doesn't. The path reconstruction ambiguity that kills BOS_FVG trailing results is absent here. The SF portfolio is the closest thing the Layer 2 mining corpus has to a "defensible" research track.

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

This is the headline number Harrison's memory refers to as "SF portfolio" current state. It has not been independently audited at tick level yet — the claim of "zero look-ahead" is based on the 8-bug fix list above, not on an independent tick-level rerun like the one done for BOS_FVG in 2026-04-10.

**Status:** Claimed-clean Layer 2 research. The *structural* reasons to trust it (fixed targets, not trailing) are solid. The *audited* status is less solid — no tick-level cross-validation has been done, and no Bonferroni correction across the 13-leg selection has been published. Treat as "probably defensible but not yet verified" rather than "validated."

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
