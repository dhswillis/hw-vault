# Cowork Prompt: Build TempoPortfoliov1.cs

Read these files in this directory:
1. `TEMPO_PORTFOLIO_BUILD_SPEC.md` — the full build spec for what to create
2. `SweepBreakv17.cs` — the reference NinjaTrader strategy to use as a template for unmanaged orders, order handling, execution updates, stats display, and lifecycle

Then create `TempoPortfoliov1.cs` in this same directory.

This is a NinjaTrader 8 strategy (C#) that runs TWO independent signal engines simultaneously:

**Engine 1 — IFVG**: Sweep at key level → Inverse FVG → candle close through → SOFT stop reversal entry → session-aware partial profits. Runs on 1-minute bars.

**Engine 2 — Lumi**: HTF (60-min) sweep of HTF level → STRICT MSS confirmation on 3-min → LTF (3-min) order block + FVG → limit entry on retrace → HARD stop → 3R target.

Key requirements:
- IsUnmanaged = true (same pattern as SweepBreakv17.cs)
- Both engines can have trades open at the same time (separate TradeInfo objects)
- IFVG uses SOFT stops (check on bar close, NOT stop-market order)
- Lumi uses HARD stops (normal stop-market order)
- STRICT MSS for Lumi: MSS bar index must be < fill bar index (look-ahead bug fix)
- Multi-timeframe: AddDataSeries for 3-min and 60-min
- On-chart stats table showing both engines separately and combined
- All parameters configurable with defaults from the spec

The build spec has every detail — entry logic, stop logic, TP logic, parameters, levels, MSS rules, stats format. Follow it exactly. Do not simplify or skip any part.

ALWAYS create a new version file — never edit SweepBreakv17.cs.
