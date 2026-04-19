---
created: 2026-04-18
updated: 2026-04-18
type: summary
sources:
  - /Users/harrisonwillis/Documents/strategies/tempo/LumiStrategyV3.cs
related:
  - wiki/summaries/lumi-strategy-v2-2026-04-10.md
  - wiki/summaries/lumi-es-strategy-v2-2026-04-10.md
  - wiki/summaries/tempo-portfolio-v26.md
  - wiki/summaries/lumi-strategy-spec.md
  - wiki/concepts/ifvg.md
  - wiki/maps/strategies-moc.md
tags: [trading, lumi, ninjatrader, current, nq]
---

# LumiStrategyV3 — Production-grade Lumi (Apr 18, 2026, NQ)

> **Replaces [[lumi-strategy-v2-2026-04-10]]** as the current Lumi implementation. File: `/Users/harrisonwillis/Documents/strategies/tempo/LumiStrategyV3.cs` (902 lines, Apr 18, 2026). Runs standalone alongside [[tempo-portfolio-v26]].

## What changed from V2

V2 was functional but had production-readiness gaps. V3 ports [[tempo-portfolio-v26]]'s battle-tested order patterns into Lumi:

| Area | V2 | V3 |
|---|---|---|
| Order handling (BT) | Internal sim (`trade.Filled = true`) | Real unmanaged orders, same path BT and live |
| Position sizing | Fixed `Qty` parameter | Dynamic sizing: `floor(DollarRiskPerTrade / (riskPts * pointValue))`, clamped `[MinQty, MaxQty]` |
| Race condition | None | `IsExiting` flag blocks new entry until cancel confirmation (v2.5 pattern) |
| Orphan safety | None | Every 1m bar checks active trade still has working stop; emergency exit if not |
| MSS enforcement | MSS checked but bar index not tracked | Strict: `MSSBar` recorded, entry rejected if MSS bar invalid |
| Circuit breakers | Max trades/day only | + Daily R limit (`-3R`), + max consecutive losses (`3`) |
| Entry timeout | Internal counter on `CurrentBars[0] - PosID` | Same logic but via real order cancel |
| Order names | `LUMI_{posID}` | `LUMI_Entry/SL/TP/Exit/ErrExit_{posID}` for Trade Performance filtering |
| Emergency exits | None | On unexpected stop/target cancel, immediate market exit + counterpart cancel |
| Day reset | Simple clear | Flattens leftover positions BEFORE clearing state (v2.5 fix #4) |

## Signal chain (unchanged from V2)

Same multi-TF Lumi logic: HTF sweep (M15/M30/H1) -> MSS on LTF (3m/5m) -> OB + FVG -> limit entry at FVG edge -> swing stop -> R-target exit.

BIP layout identical to V2: 0=1m, 1=3m, 2=5m, 3=15m, 4=30m, 5=60m.

## Default parameters

| Param | Default | Group | Notes |
|---|---|---|---|
| FixedQty | 1 | General | Fallback when dynamic sizing off |
| UseDynamicSizing | false | General | Toggle for per-trade qty computation |
| DollarRiskPerTrade | 50.0 | General | Target dollar loss per -1R trade |
| MinQty | 1 | General | Floor for dynamic qty |
| MaxQty | 20 | General | Ceiling for dynamic qty |
| EntryStartCT | 70000 | Session | 7:00 AM CT (8:00 AM ET) |
| EntryEndCT | 150000 | Session | 3:00 PM CT (4:00 PM ET) |
| FlattenCT | 145500 | Session | 2:55 PM CT |
| MaxTradesPerDay | 3 | Risk | Hard daily cap |
| DailyRLimit | -3.0 | Risk | Stop trading if daily R <= this |
| MaxConsecLoss | 3 | Risk | Stop trading after N consecutive losses |
| TargetR | 1.75 | Setup | R-multiple for profit target |
| MinSweep | 1.0 | Setup | Min wick past level (pts) |
| SLBuf | 1.0 | Setup | Buffer past swing stop (pts) |
| OBBody | 0.25 | Setup | OB min body as fraction of range |
| FVGMin | 0.5 | Setup | Min FVG gap (pts) |
| MinRisk | 1.0 | Setup | Min acceptable risk (pts) |
| MaxRisk | 60.0 | Setup | Max acceptable risk (pts) |
| SwLB | 3 | Setup | Swing lookback bars |
| MSSMaxBars | 20 | Setup | Max bars after sweep to find MSS |

## v26 patterns ported

### IsExiting race condition fix (v2.5)

When stop or target fills, the counterpart order is cancelled async. Between fill and cancel confirmation, a new entry could sneak in on the same instrument. V3 sets `trade.IsExiting = true` after stop/target fill — blocks any new trade until the cancel callback arrives in `OnOrderUpdate`. Safety valve: force-clears after 5 minutes if cancel never confirms.

### Orphaned position safety sweep

On every 1m bar (`BarsInProgress == 0`), checks: if `trade.IsActive == true` but `StopOrder` is not in `Working`/`Accepted` state -> emergency market exit. Prevents positions sitting without stop protection.

### Dynamic position sizing (v2.6)

`qty = floor(DollarRiskPerTrade / (riskPts * pointValue))`, clamped to `[MinQty, MaxQty]`. Each `LumiTrade` stores its own `Qty` so stops/targets/exits use the same size.

## Tick-through caveat (inherited from V2)

V2's tick-through audit (2026-04-16) showed the limit-fill assumption accounts for the entire +0.45R edge. V3 inherits the same signal chain, so the same caveat applies: **needs paper-trade validation before scaling**. V3's improvements are structural (order safety, sizing, race conditions) not signal-level.

## Open questions

- V2 tick-through audit showed -0.065R with strict tick execution. Does NT depth-of-book fill enough limits to restore edge?
- Correlation between V3 Lumi and v26 IFVG in live: the Python audit found 0.130 correlation (strict MSS, different R target) — needs re-measurement with V3's 1.75R target.
- ES port: V2 has an ES version (`LumiESStrategyV2.cs`). V3 does not yet have an ES port.

## Cross-references

- [[lumi-strategy-v2-2026-04-10]] — V2, now superseded by V3
- [[lumi-es-strategy-v2-2026-04-10]] — ES port (still V2, no V3 ES yet)
- [[tempo-portfolio-v26]] — source of order patterns ported to V3
- [[lumi-strategy-spec]] — V15-era spec, historically superseded
- [[invalidation-rules]] — Rule 1 (bar-sim trailing) not applicable; Lumi uses hard stop
