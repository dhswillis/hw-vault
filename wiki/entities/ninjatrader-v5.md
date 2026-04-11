---
created: 2026-04-10
updated: 2026-04-10
type: entity
sources:
  - raw-sources/V5_Strategy_Bug_Audit_2026-03-23.md
related:
  - wiki/summaries/v5-strategy-bug-audit.md
  - wiki/concepts/vwap-double-counting-bug.md
  - wiki/entities/tempo-trading-system.md
tags: [trading, ninjatrader, sf-portfolio, v5, live]
---

# NinjaTrader V5 Strategies (SF Portfolio)

The V5-generation NinjaTrader strategies form the "SF portfolio" stack — the live-execution side of the trading operation, distinct from the [[tempo-trading-system|Tempo]] research/backtest pipeline on [[quantconnect|QuantConnect]]. Three strategies, all in the same `/strategies/03-sf-portfolio/ninjatrader/` directory.

## The strategies

| Strategy | Instrument | Type |
|---|---|---|
| `V5NQVWPMStrategy` | NQ | VWAP Point Mean-revert (VWPM) |
| `V5ESVWPMStrategy` | ES | VWAP Point Mean-revert (VWPM) |
| `V5NQLF1170Strategy` | NQ | Liquidity-fade 11-70 (LF1170) |

`V5NQVWPMStrategy` and `V5NQLF1170Strategy` run on the same NQ chart simultaneously — this matters for the chart-stats collision bug below.

## Python reference backtests

The corresponding Python backtests live at:
- `es_vwpm_260d.py`
- `detailed_stats_260d.py`
- `portfolio_clean_260d.py`

The Python loops `for i in range(len(pr))` — once per minute bar — which is the intended computational model. The NinjaTrader strategies use `Calculate.OnEachTick` but don't gate their bar-once accumulators, producing the [[vwap-double-counting-bug|VWAP double-counting bug]] that explains live/backtest divergence.

## Live vs backtest divergence

Observed on NinjaTrader Sim101 over 02/27–03/23/2026:

| | Backtest | Live |
|---|---|---|
| ES VWPM WR | 66% | **44.4%** |
| ES VWPM R/Day | +0.66 | **−0.292** |
| ES VWPM PF | — | 0.68 |

The [[v5-strategy-bug-audit|audit]] pinpoints the [[vwap-double-counting-bug|VWAP tick-counting bug]] as the primary cause.

## Known bugs (as of 2026-03-23)

Seven bugs identified, fix priority:

1. **Bug 1 (CRITICAL)** — VWAP double-counting in live. See [[vwap-double-counting-bug]].
2. **Bug 3 (HIGH)** — Chart stats tag collision. Both NQ strategies use `"V5Stats"` as the `DrawTextFixed` tag, so LF1170 overwrites NQ_VWPM's stats on the shared NQ chart. Harrison is currently flying blind on NQ_VWPM live performance.
3. **Bug 2 (CRITICAL)** — No protective stop orders on the exchange. Stops and targets managed purely in-code via `ManageTrade()`. If NinjaTrader disconnects, crashes, or the PC loses power mid-trade, positions sit in the market with zero protection.
4. **Bug 6 (MODERATE)** — Session-flat uses `Close[0]` which is the first bar of the **new** session (includes overnight gap). Should use `Close[1]` or `IsExitOnSessionCloseStrategy = true`.
5. **Bug 4 (MODERATE)** — Profit Factor displays `0.00` when all trades win because `grossLoss == 0` isn't handled.
6. **Bug 5 (MODERATE)** — Virtual P&L diverges from actual fills. No `OnExecutionUpdate` handler reconciling real fill prices with the strategy's `entryPrice = price + VSlip * dir` model. Stop exits always record P&L as exactly `-StopPts`.
7. **Bug 7 (LOW)** — Side establishment paused during active trade (VWPM only). `if (inTrade) return;` skips the side-tracking block, so after exit the strategy must wait for price to move ≥ threshold from VWAP again. Python continues side tracking during trades, so Python re-enters sooner. Reduces live trade frequency.

## Development rule (from memory)

Per Tempo practice: **always create a new version file** (e.g., `SweepBreakv17.cs`) — never edit existing version files in place. Historical versions must remain reproducible for audit and comparison.

## Audit context

The bug audit was triggered by the observable live/backtest divergence on ES VWPM. Fixes will ship in priority order (Bug 1 first, then Bug 3 for visibility, then Bug 2 for risk management).
