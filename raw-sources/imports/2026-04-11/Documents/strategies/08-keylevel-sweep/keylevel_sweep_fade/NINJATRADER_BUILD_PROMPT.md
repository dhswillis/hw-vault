# Prompt: Build NinjaTrader 8 Sweep Fade Strategy

Copy everything below this line and paste it as your first message to Claude.

---

Build a NinjaTrader 8 C# strategy called `SweepFade` based on a detailed spec document. The spec contains every parameter, every piece of logic, and every nuance needed.

**Step 1**: Read the build spec:
```
/Users/harrisonwillis/Documents/trading-system/keylevel_sweep_fade/NINJATRADER_BUILD_SPEC.md
```

**Step 2**: Read the Python backtester that this strategy must replicate (this is the source of truth for all logic):
```
/tmp/phase18_robustness.py
```
Focus on the `simulate_day()` function (lines 149-514). This is the exact logic the C# must match.

**Step 3**: Build `SweepFade.cs` and write it to:
```
/Users/harrisonwillis/Documents/trading-system/keylevel_sweep_fade/SweepFade.cs
```

**Requirements**:
- NinjaTrader 8, C#, single `.cs` file
- Unmanaged order mode (`IsUnmanaged = true`)
- Primary series: 1-minute bars. Secondary series: 60-minute bars (for 1H swing levels)
- `Calculate.OnEachTick` for real-time sweep detection
- 4 sub-strategies with priority system: PrevDay > 1H Wick > Round50 > VWAP
- One position at a time globally
- Trail stop (1R trigger / 0.5R distance) for strategies 1-3, fixed 2R target for VWAP
- All session times, wick filters, risk filters, max entries per level — every parameter is in the spec
- User-configurable NinjaScript properties for all key parameters
- Prev day H/L must track sweeps from overnight/globex data (not just RTH)

**Step 4**: After writing the file, review it 3 times:
1. First pass: Compare every parameter against the spec (wick ranges, risk ranges, max entries, trail values, session times). Flag any mismatch.
2. Second pass: Walk through the sweep detection state machine for each strategy type. Verify the reset logic, the sweep_extreme initialization for "below" directions (must be double.MaxValue not 0), and the entry trigger conditions.
3. Third pass: Check NinjaTrader-specific correctness — OnOrderUpdate/OnExecutionUpdate handling, order lifecycle (null checks, cancel before resubmit), BarsInProgress guards, session reset logic, and the 8 potential issues listed in the spec.

Fix any issues found during review before presenting the final file.

**Do not ask me questions. Build it, review it, fix it.**
