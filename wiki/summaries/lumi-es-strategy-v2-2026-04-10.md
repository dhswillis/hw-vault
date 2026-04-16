---
created: 2026-04-16
updated: 2026-04-16
type: summary
sources:
  - /Users/harrisonwillis/Documents/strategies/tempo/LumiESStrategyV2.cs
related:
  - wiki/summaries/lumi-strategy-v2-2026-04-10.md
  - wiki/summaries/lumi-strategy-spec.md
  - wiki/entities/tempo-methodology.md
tags: [trading, lumi, ninjatrader, es, emini, current, live-strategy]
---

# LumiESStrategyV2 — ES Port of Lumi (Apr 10, 2026)

> **New deployment**: Lumi strategy ported to **ES (E-mini S&P 500)** — separate from the NQ version. File: `/Users/harrisonwillis/Documents/strategies/tempo/LumiESStrategyV2.cs` (26,764 bytes, Apr 10, 2026). **This is the first documented Tempo-family strategy running on a non-NQ instrument.**

## Why this is new

Every prior vault doc (V15, v26, LumiV2-NQ) focused exclusively on NQ. The ES version represents:
1. **Instrument diversification** — NQ + ES are correlated but not identical; can smooth daily PnL
2. **Validation of the method** — if Lumi is regime-agnostic it should work on ES too
3. **Expanded capital deployment** — same methodology, 2× the instruments

No vault doc previously mentioned Lumi running on ES. This was found by searching the `strategies/tempo/` directory during the 2026-04-16 vault audit.

## Parameter differences from NQ Lumi

| Param | NQ LumiV2 | **ES LumiV2** | Notes |
|---|---|---|---|
| TargetR | 1.75 | **2.0** | ES back at 2R — V15-era setting |
| MinSweep (pts) | 1.0 | **0.5** | ES tick-size is 0.25, smaller pt values |
| SLBuffer (pts) | 1.0 | **0.5** | Tighter buffer for ES |
| OB Min Body % | 0.25 | 0.25 | Same |
| FVG Min Gap | 0.5 | **0.25** | Smaller gaps acceptable on ES |
| SwingLB | 3 | 3 | Same |
| IsExitOnSessionCloseStrategy | false | **true** | Different safety net; ES uses NT8 session close |

**Why the parameter differences**: ES has a smaller tick size (0.25 vs NQ's 0.25 but different dollar value — ES is $50/pt vs NQ $20/pt) and different volatility profile (lower absolute range). Parameters are scaled down accordingly.

## Architecture

Same as NQ LumiV2:
- BIP 0: 1-min primary
- BIP 1-5: 3m, 5m, 15m, 30m, 60m secondary
- HTF sweep (M15/M30/H1) → MSS → LTF OB+FVG (M3/M5) → limit entry at FVG/OB edge
- Swing H/L stop + SLBuffer
- Target = entry ± (risk × TargetR)
- Session 8:00 AM – 4:00 PM ET, flatten 15:55 ET
- MaxTrades = 3 per day

## Performance

**Not yet audited at tick level in Python.** No `lumi_es_backtest*.py` or `lumi_es_*.json` in results directory. Only the NT C# code exists.

To produce a comparable number to NQ Lumi (+0.45R, PF 2.29):
- Need ES tick data (NOT in current Databento NQ dataset)
- Need to port `lumi_backtest_1.75r.py` to load ES ticks + adjust parameters
- Or rely on NT forward-test / replay on ES data

## Known issues

- **No Python audit**: all NQ strategies have Python tick-level backtests as validators. ES does not. Running live without a validator is a gap.
- **Parameter choices unexplained**: why TargetR=2.0 on ES when NQ moved to 1.75? MFE analysis may differ between instruments but no documented analysis.
- **Session close behavior differs** (`IsExitOnSessionCloseStrategy=true` on ES vs `false` on NQ). NQ LumiV2 uses explicit flatten logic; ES delegates to NT8. If ES has multiple components or nesting issues, this could surface the same race condition that v25 TempoPortfolio had to fix.

## Memory recommendation

When assessing Lumi live performance, remember to ask: "NQ or ES?" They are now two separate strategies with different parameters. Reports need to split them.

## Cross-references

- [[lumi-strategy-v2-2026-04-10]] — NQ sister strategy
- [[lumi-strategy-spec]] — V15 era, superseded
- [[tempo-portfolio-v26]] — NQ IFVG that runs alongside LumiV2 NQ
