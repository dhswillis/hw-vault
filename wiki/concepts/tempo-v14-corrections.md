---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/tempo-v14-corrections.md
  - wiki/summaries/tempo-cluster.md
  - wiki/entities/ninjatrader-v5.md
tags: [trading, tempo, implementation, bug, ninjatrader]
---

# Tempo v14 Corrections

Four critical implementation bugs in the v14 C# NinjaTrader implementation of the Tempo [[ifvg|IFVG]] strategy, documented in a 2026-03-20 audit. The v14 code was following a sequence that *sounded* like Tempo's methodology but got the trade mechanics wrong in ways that systematically destroyed edge.

## The four bugs

### 1. Signal sequence reversed — "FVG before sweep, not after"

**What v14 did:** waited for a liquidity sweep, then scanned for a new FVG to form, then entered on a close through the fresh FVG.

**What Tempo actually teaches:** a momentum gap (FVG) forms *first*, on the way to the key level. Then price sweeps the key level. Then price reverses back through the *pre-existing* FVG. Entry happens at the FVG edge when price trades back through it.

The mechanical difference is subtle but critical:
- v14 version is a "new-FVG-after-sweep" confirmation entry — momentum continuation dressed up as reversal
- Correct version is a "pre-existing FVG inverted by the reversal" entry — genuine liquidity reversal

The correct version is also the origin of the name *inverted* FVG — the inversion is the reversal through a pre-existing gap, not a new FVG printing after the sweep. v14's name was right; its implementation was doing a different strategy.

### 2. Stop placement — FVG edge, not sweep wick

**What v14 did:** placed stops at the sweep wick ± 1 point. On NQ this created fixed stops of 20–30+ points regardless of gap size.

**What Tempo actually teaches:** soft stop at the **opposite IFVG edge**. This means stop distance equals gap size (5–15 points typical). Risk = gap size, by construction. The sweep wick is an emergency hard stop — a safety net, not a primary stop — and is tick-based rather than close-based.

v14's wider-than-correct stops meant:
- Risk-per-trade was 2–5× what it should have been
- The session-aware TP rules (TP1 at +17 points) never lined up with the risk model
- Position sizing was miscalibrated — 0.4–0.6% per trade at v14 stops meant massively smaller size than Tempo intended

### 3. Entry price — FVG edge exactly, not bar close

**What v14 did:** entered at the bar close when the close crossed back through the FVG. On 1-minute or larger bars, this routinely meant entries 13+ points past the FVG edge.

**What Tempo actually teaches:** enter at the **FVG edge exactly**. Tempo uses 30-second bars specifically to get precise detection near the edge. On a 1-minute bar, the close can be far past the edge by the time it prints.

Effect on edge: v14's late entries gave up ~13pt per trade of the move, and increased the soft-stop distance relative to the edge because the stop (gap size) was measured from the bar close rather than from the FVG edge itself.

### 4. Emergency stop on the wrong side for fade trades

**What v14 did:** on fade shorts (30 trades in the audit period), the emergency hard stop was placed *above* the entry price instead of above the sweep wick. Because the sweep wick is above entry for a short, and the hard stop should be just above the wick, v14 placed it correctly for one case and inverted for the other.

**The bug in concrete terms:** for fade shorts, the emergency hard stop ended up on the *profit* side of the trade, which effectively disabled the hard stop — the price would reach the hard stop level only after a winning move.

**The fix:** `max(wick + 1pt, entry + 3pt)` for shorts, `min(wick − 1pt, entry − 3pt)` for longs. The emergency stop is now guaranteed to be on the loss side, regardless of the geometric relationship between entry, wick, and gap.

## Post-correction backtest (30 days, 2025-03-03 → 2025-04-04)

The audit re-ran v14 with all four corrections on Databento NQ tick data, 30 trading days:

| Slice | Trades | WR | Pts/day | Calmar |
|---|---|---|---|---|
| **US sessions (NY AM/Lunch/PM)** | 105 | **70.5%** | **+24.5** | **5.7** |
| London only | 58 | 46.6% | +10.9 | — |
| Combined (US + London) | 163 | 62.0% | +31.9 | 4.8 |

The US-session result is the only clean edge. London is a structural problem even with corrections: 247% of London's total PnL came from 3 end-of-day runners. Non-EOD London is **−12.7 pts/day** — deeply negative when you strip out the session-end flatten. The audit's recommendation is "skip London or fundamentally different stop logic."

Lumi engine (a separate strategy shipped as part of v14) tested at 35 trades, 20% WR, −3.4 pts/day. Below break-even (Lumi targets 3R and needs ~25% WR). Recommendation: exclude from production or test HTF_OB-only subset.

## Status as of 2026-04-10

v14 with corrections is the current NinjaTrader sim reference for IFVG implementation. It is **not yet proven live** — the 30-day audit is an in-sample fix, not walk-forward OOS evidence. Harrison's `CLAUDE.md` at `~/Documents/trading-system/` notes that "v24 (IFVG MTF cascade) is the ONLY strategy currently on sim" — v24 appears to be a subsequent, MTF-cascade version that supersedes v14. The v14 corrections are historically important as documentation of what the C# code was doing wrong, but the active implementation is now v24.

## Why this is a different bug class from [[bar-sim-trailing-bug]]

The v14 bugs are **spec-implementation divergence** — the C# code was doing a different strategy than the one described in Tempo's rules. The fix is to read the spec carefully and rewrite the code. You can validate the fix in-sample because the signal itself is deterministic once you get the rules right.

The bar-sim trailing bug is **backtest-execution divergence** — the simulator's model of intra-bar price action is fundamentally wrong, and no amount of code reading helps because the problem is in the data representation (OHLC) rather than the logic. The only fix is tick-level simulation or replacing the trailing stop with a close-based exit.

IFVG's native soft stop (close past gap edge) is bar-sim-safe by construction, which is why the v14 corrected results don't need a tick-level re-audit the way BOS_FVG does. See [[bar-sim-trailing-bug]] for the structural explanation.
