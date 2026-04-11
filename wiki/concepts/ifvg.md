---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
related:
  - wiki/concepts/smt.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/bpr.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/concepts/bos-fvg.md
  - wiki/entities/tempo-methodology.md
  - wiki/summaries/tempo-cluster.md
tags: [trading, signal, ifvg, tempo, canonical]
---

# IFVG — Inverted Fair Value Gap

The **canonical [[tempo-methodology|Tempo]] entry signal**. Not to be confused with [[bos-fvg|BOS_FVG]] — which is a mining-era *interpretation* of IFVG that has been repeatedly shown not to work in Harrison's backtests.

## Definition

A 3-candle pattern where the middle candle creates a price gap (FVG), and price later closes back through that gap in the opposite direction to trigger entry. The "inversion" is price trading back through an FVG that already existed.

## The correct signal sequence

Per [[tempo-v14-corrections|v14 corrections]], the build spec had this WRONG. The correct sequence:

1. **Price moves aggressively** toward a key level
2. **On the way**, a 3-bar FVG forms in the direction of the move (momentum gap)
3. **A solid bar follows** (no stacked gaps — singular FVG preferred)
4. **Price continues and sweeps past the key level** (wick past, close back inside)
5. **Price reverses and closes back through** the same pre-existing FVG
6. **Entry at the FVG edge** (not at bar close, not at sweep level)
7. Use 30-second bars for precise entry detection near the edge

**Key insight:** the FVG forms BEFORE the sweep, not after. The sweep is the catalyst for the reversal back through the pre-existing gap. The "inversion" is semantic, not temporal.

## Gap sizing (NQ)

| Size | Treatment |
|---|---|
| < 5 pts | Skip (too tight) |
| 5–6 pts | Risky but valid with additional confluence |
| **8–15 pts** | **Optimal zone** |
| 20–30 pts | Strong setup, high conviction |
| > 30 pts | SKIP (stop too wide for risk) |

## Entry trigger rules

- **Close** through the gap — not wick touches
- **Singular gaps preferred** — stacked FVGs increase failure probability
- **Reject FVG into IFVG** = strong confidence (HTF rejection feeds into LTF inversion)
- **Half-position protocol:** enter half on close, add second half on pullback/retest

## Confirmation patterns (must see before entering)

- V-shaped recovery with instant momentum
- Death candles through gaps (80–100pt candles on NQ)
- Wick rejection without close past level
- Three consecutive directional candles after sweep
- **46+ pt death candle** = textbook momentum confirmation
- **Principle:** "Reaction is everything" — price reaction to the sweep matters more than the sweep itself

## Timeframes (V3, updated Oct 2025)

| TF | Use |
|---|---|
| **2-minute** | Preferred default |
| **30-second** | V3 upgraded to equally viable (was "risky" in V1) |
| 1-minute | Good, sometimes better entries |
| 15-second | ONLY for data wick / data candle entries |
| 5-minute | Wider setups, high-conviction days |

Check HTF (4H → 1H → 15m) for bias before dropping to 2m/30s for entry.

## Required confluences for a funded-account trade

Tempo raised the minimum from 2 to **3+ confluences** in Jan 2026. See [[setup-quality-grading]] for the A+/A/B+/B tier system.

**The 80% WR formula:** [[smt|SMT]] + IFVG + liquidity sweep. Without SMT, WR drops to ~60-65%.

## Stops

Per [[tempo-v14-corrections]]:
- **Soft stop (default, 90% of the time):** opposite IFVG edge, checked on 1-min bar CLOSES only. LONG = FVG low; SHORT = FVG high.
- **Emergency hard stop (safety net):** sweep wick ± 1pt, tick-based, catastrophic protection only
- **Risk = gap size** (entry at one FVG edge, stop at the other)
- **After B/E:** hard stop at entry price replaces both soft and emergency

Stops of **5–8 pts are too tight** — get wicked out. 20–30 pts is the typical real range when entering at the FVG edge.

## Session-aware trade management (per v14)

| Session | TP1 | TP2 | Runner | B/E trigger |
|---|---|---|---|---|
| London (pre-8:30 ET) | None | None | 100% | +17 pts |
| NY AM (9:30–11:00) | 30% at +17 | None | 70% | After TP1 |
| NY Lunch (11:00–13:00) | 55% at +17 | 35% at +35 | 10% | After TP1 |
| NY PM (13:00+) | 55% at +17 | 35% at +35 | 10% | After TP1 |

**London is broken for IFVG.** The soft stop (5–7pt gap) catches every trade before it can run to +17pts. Recommendation: skip London in production.

## Related failure modes

- **Multiple stacked FVGs** = high failure rate — skip
- **Small gap + tight stop** = get wicked out
- **First 2–5 minutes after open** = too volatile — wait
- **All overnight areas swept before open** = worst conditions, no edge
- **At ATH with no sell-side liquidity** = no targets, dangerous shorts

## IFVG vs BOS_FVG (important distinction)

| | IFVG | [[bos-fvg\|BOS_FVG]] |
|---|---|---|
| Origin | Tempo's canonical teaching | Harrison's mining interpretation |
| Status | Real strategy Tempo trades live at 88% WR | NOT validated for live trading per [[unified-brain-architecture]] |
| Best research | `tempo_rules_v3_complete.md` (281 transcriptions) | `COMPREHENSIVE_MINING_REPORT.md` V2 (contaminated era) |
| Entry | FVG forms before sweep, entry at FVG edge on close back through | Break of structure then FVG, entry on retest |
| Stop | Soft stop at opposite FVG edge (gap size) | Historically fixed R with BE+Trail |

The two signals are related — both involve FVGs and liquidity structure — but the mining-era BOS_FVG implementation does not match Tempo's taught IFVG, and per the [[unified-brain-architecture|Unified Brain doc]] the BOS_FVG signal does not work live. See [[bos-fvg-claim-vs-reality]].
