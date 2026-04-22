---
created: 2026-04-16
updated: 2026-04-21
type: summary
sources:
  - /Users/harrisonwillis/Documents/strategies/tempo/TempoPortfoliov26.cs
related:
  - wiki/summaries/tempo-portfolio-v15.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/concepts/invalidation-rules.md
tags: [trading, tempo, ifvg, ninjatrader, live-strategy, current]
---

# Tempo Portfolio v26 — Current Live Strategy (Apr 10, 2026)

> **This is the current production NinjaTrader strategy.** Supersedes [[tempo-portfolio-v15|v15]] (Mar 22). v16–v25 exist on disk but have no vault summary — this gap was flagged on 2026-04-16.

## What is v26

`TempoPortfoliov26.cs` is an IFVG Multi-Timeframe Cascade strategy with **6 independent components** running in parallel on the same NQ chart. Each component detects its own sweeps and IFVGs on a specific LTF (15-second, 30-second, or 1-minute) and trades one direction. Position-level risk management is shared via portfolio circuit breakers.

## The 6 components

| ID | Key | LTF | Dir | RT | GapMin | GapCap | MinRisk | ReactionMaxBars | Allowed Sessions |
|---|---|---|---|---|---|---|---|---|---|
| 0 | 15S_short | 15S | short | 3.0R | 1.5pt | 6t (1.5pt) | 2.0pt | 20 | NY_AM, London_Close, NY_Close, NY_Open |
| 1 | 15S_long | 15S | long | 3.0R | 1.5pt | 6t | 2.0pt | 20 | NY_PreOpen, NY_Lunch, London_Close, London_Open |
| 2 | 30S_short | 30S | short | 2.0R | 2.0pt | 10t (2.5pt) | 2.0pt | 10 | NY_Lunch, NY_PM, NY_Close, NY_PreOpen, London_Mid, Asia_Late |
| 3 | 30S_long | 30S | long | 2.0R | 2.0pt | 10t | 2.0pt | 10 | London_Mid, NY_PM, NY_PreOpen, London_Close, NY_Open, NY_AM |
| 4 | 1M_short | 1M | short | 3.0R | 3.0pt | 20t (5pt) | 5.0pt | 5 | London_Close, NY_Lunch, NY_AM, NY_Close |
| 5 | 1M_long | 1M | long | 3.0R | 3.0pt | 20t | 5.0pt | 5 | London_Mid, Asia_Late, NY_AM, NY_Open |

Each component has independent:
- Session whitelist (asymmetric — longs and shorts allowed in different sessions)
- Daily R limit (-1 to -3R per component)
- Weekly R limit (-4R on some shorts, none on longs)
- Streak limit (2 consecutive losses on shorts, typically unlimited on longs)

## Entry logic

1. **Sweep detection** (on component's LTF): wick past a key level + close back inside, sweep ≥ `SweepMinPts` (default 1.0pt)
2. **FVG formation after sweep** (3-bar pattern): gap ≥ `GapMin`, ≤ `GapCap`, risk between `MinRisk` and `RiskCap`
3. **Reaction scoring** (between sweep and FVG, scans `ReactionMaxBars` of LTF bars):
   - Counts consecutive directional candles after the sweep bar
   - Tallies biggest body, cumulative displacement
   - Score 3 = biggest body ≥ 20pt (death candle) AND ≥ 1 consec, OR ≥ 3 consec with total displacement ≥ 10pt
   - Score 2 = ≥ 2 consec with total displacement ≥ 7pt
   - Score 1 = ≥ 1 consec
   - Score 0 = none
4. **Entry accept**: `MinReaction` ≥ 3 (default) — **death candle or 3+ aligned candles required**
5. **Session filter**: current session must be in component's `AllowedSessions`
6. **Portfolio guards**: daily R not below -6R, weekly not below -10R, streak counter not exceeded

## Trade management

- **Entry**: limit order at FVG edge (short: FVG bottom edge / gL; long: FVG top edge / gH)
- **Hard stop**: `FVG_opposite_edge ± Buffer` (buffer 1.5pt on 15S, 2pt on 30S, 4pt on 1M)
- **Target**: `entry ± (risk × RT)` — fixed R multiple, not session partial
- **No B/E, no trailing** — hard stop / fixed target only
- **Position sizing**: dynamic `qty = floor($DollarRiskPerTrade / (risk_pts × $pointValue))`, clamped to [MinQty, MaxQty]

## Sessions (v26 definitions, ET)

| Session | ET Range |
|---|---|
| Asia_Late | 00:00 – 02:59 |
| London_Open | 03:00 – 04:59 |
| London_Mid | 05:00 – 06:59 |
| London_PM | 07:00 – 08:29 |
| NY_PreOpen | 08:30 – 09:34 |
| NY_Open | 09:35 – 10:29 |
| NY_AM | 10:30 – 11:29 |
| London_Close | 11:30 – 11:59 |
| NY_Lunch | 12:00 – 12:59 |
| NY_PM | 13:00 – 14:59 |
| NY_Close | 15:00+ |

This is a much finer partition than v14/v15's London/NYAM/NYLunch/NYPM. Each 1-hour slice is treated as its own regime.

## v14 → v15 → v26 progression

- **v14**: 1 IFVG component, session-aware TP partials (TP1=17pt, TP2=35pt), B/E after +3R
- **v15**: IFVG + Lumi engine (HTF cascade), FVG soft stop, 2R from FVG for Lumi, news-day exclusion. IFVG still session-aware TPs.
- **v16–v19** (Mar 23): undocumented in vault
- **v20–v23** (Mar 30): undocumented
- **v24** (Apr 8): IFVG MTF cascade introduced — the 6-component structure
- **v25** (Apr 8): race condition + session close safety net fixes (IsExitOnSessionCloseStrategy=false, OnOrderUpdate cancellation handling)
- **v26** (Apr 10): dynamic position sizing — per-trade qty from $ risk instead of fixed 1 contract

Critical architectural shift at **v24**: abandoned session-partials / B/E in favor of fixed R targets with reaction-score entry gating. This aligns with [[bar-sim-trailing-bug|Rule 1]] (no bar-sim trailing) and [[be-trail-mechanism|Rule 3]] (no B/E on non-Tempo signals, but also simplifies here).

## How v26 differs from my 2026-04-12 backtest

The autonomous 2026-04-12 research (since pruned — see [[ifvg-composite-audit-20260329|canonical March audit]] for the current reference) tested v14 logic with post-hoc splits by `counter_trend_1h × body_over_gap`. That was the **wrong baseline** — v26 had already:

1. Replaced body/gap post-hoc filter with a canonical `ReactionScore ≥ 3` entry gate
2. Split by LTF timeframe (15S/30S/1M) not 1H-trend
3. Added fixed R targets per component (no session partials)
4. Added per-component session whitelists and streak/daily R limits
5. Switched to dynamic position sizing

The `Cal 419` / 89.7% DWR number from 2026-04-12 was based on (a) limit-fill assumption and (b) v14 session-partial mechanics. Both are superseded — v26 doesn't use session partials and real fills require a tick to trade through the level.

## Known issues / open questions

- **No vault doc for v16–v25**: full 10-version gap. Each version likely has its own fixes (v25 has a race condition fix, implying v16–v24 had that bug in production).
- **No current tick-level audit for v26 in the vault**: the last formal audit was V15 (which assumes limit fills). No equivalent for v26's MTF reaction-score structure.
- **Per-component session asymmetry is not explained**: why 15S_short allowed in NY_Close but 15S_long only in NY_PreOpen/Lunch? This looks like empirical backtest fit — needs provenance.
- **Fixed R targets may be arbitrary**: 15S=3R, 30S=2R, 1M=3R. Basis for these ratios is not documented.

## Validated Python audit (canonical reference)

The **canonical validated audit** is [[ifvg-composite-audit-20260329]], which uses `composite_strategy.py` (607 lines) on tick-level Databento data:

- 6-component combined: **+7.55 R/day, 60% WR, 1,707 trades/247 days**
- Market-entry variant (zero fill assumptions): **+17.9 PPD, 48% WR**
- 3 bugs found and fixed during Mar 29 audit; Python↔C# cross-reference verified

v26 C# configs match `composite_strategy.py` OPTIMIZED_RULES exactly. v26 adds dynamic position sizing and v25 race-condition fixes.

**Note on a separate 2026-04-16 Python port**: An independent v26 port was built and tested, but used a different (broken) fill mechanic that searched for retraces after bar close. Its results (-0.27 PPD tick-through) were incorrect and have been superseded by the canonical March audit. The March `market_vs_limit_summary.json` already addressed the fill question properly and showed +17.9 PPD at market entry.

## Cross-references

- [[ifvg]] — the entry mechanic
- [[mtf-alignment]] — v26 uses completed-bar only (Rule 2 compliant at code-level, but MTF cascade across 15S→30S→1M would benefit from an alignment audit)
- [[invalidation-rules]] — Rule 9 (NT8 `OnEachTick` accumulator bug) applies if v26 has any accumulators; v26 uses `Calculate.OnBarClose` which avoids this
- [[tempo-portfolio-v15]] — previous documented version
- [[ifvg-composite-audit-20260329]] — canonical validated audit (supersedes the pruned 2026-04-12 autonomous research, which tested the wrong baseline)

## File location
`/Users/harrisonwillis/Documents/strategies/tempo/TempoPortfoliov26.cs` (Apr 10, 2026, 69629 bytes)
