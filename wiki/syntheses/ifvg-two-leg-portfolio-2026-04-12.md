---
created: 2026-04-12
updated: 2026-04-12
type: synthesis
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/LUMI_STRATEGY_SPEC.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/smt.md
  - wiki/concepts/invalidation-rules.md
  - wiki/summaries/tempo-v14-corrections.md
  - wiki/summaries/tempo-portfolio-v15.md
  - wiki/summaries/lumi-strategy-spec.md
tags: [trading, synthesis, ifvg, portfolio, backtest]
---

# IFVG Two-Leg Portfolio (2026-04-12 Autonomous Research)

> An 8-hour autonomous research session on 283 trading days of Databento NQ tick data discovered a two-leg IFVG portfolio with daily-PnL correlation -0.013. Uses only the existing [[ifvg]] mechanic — splits signal population by 1H trend alignment to create orthogonal return streams.

## Headline

From the **same** [[ifvg|IFVG]] signal generator, split the trade population into two disjoint subsets with opposite [[mtf-alignment|1H trend filters]]:

- **Leg A (Sharpshooter)**: `with_1h_trend AND body/gap ≥ 0.5` → 306 trades, 88.6% WR, +2.11R, +29.5 pts/day, Cal 95
- **Leg B (Counter-momentum)**: `counter_trend_1h AND body/gap ≥ 0.7` → 249 trades, 89.2% WR, +2.08R, +19.4 pts/day, Cal 54

Daily PnL correlation **-0.013** (effectively zero).

**Combined**: 555 trades, +48.96 pts/day, DWR 74.4%, Cal **216.4**, MDD 50.4 pts over 223 days.

Portfolio Cal 216 materially exceeds sum of individual Cals (95 + 54) — a real diversification benefit, not an additive sum. Individual MaxDD (69 + 81) compresses to 50 combined because losses on the two legs don't line up temporally.

## How this contradicts vault canon (and why it holds up)

Per [[ifvg]] and [[smt]] canon: "IFVG is a reversal; fade against 1H trend." Per MEMORY.md: "CT+M60+FVG≥5 = 82.9% WR, +1.044 avgR, 4.9 trades/day (sniper)".

The new data shows `with_1h_trend` (the opposite of CT) is the stronger single filter:

| Filter | Trades | WR | avg_r | PPD | Cal | 5Q pass |
|---|---|---|---|---|---|---|
| BASELINE (risk≥5) | 946 | 84.9% | +1.54 | +59.6 | 65.3 | 5/5 |
| **with_1h_trend** | 478 | 84.7% | +1.72 | +44.2 | **152.2** | 5/5 |
| counter_trend_1h | 468 | 85.0% | +1.35 | +26.6 | 29.6 | 5/5 |

**Mechanism hypothesis**: The v14 engine's IFVG logic is ALREADY a reversal (sweep a level → FVG in the REVERSAL direction → close through). Stacking `counter_trend_1h` on top makes the trade fight BOTH the sweep and the 1H trend — double-contrarian, which shows up as lower edge. `with_1h_trend` selects the liquidity-grab-then-continue pattern: 1H trend pressured the level → level gets swept → the IFVG reversal of the sweep continues in the 1H direction.

The two populations (with_1h vs counter_trend_1h) describe two distinct market regimes:
1. **Continuation after liquidity grab** (with_1h): most common; sweeps get rejected as traps, continuation resumes
2. **Exhaustion reversal** (counter_trend_1h): less common; 1H trend has exhausted at the level, genuine reversal

Both regimes are real but rarely fire on the same day. That's the source of the -0.013 correlation.

## Robustness

All tests apply the [[invalidation-rules|Rule 10]] risk floor (risk ≥ 5pt) and use tick-sim execution (bar-sim-safe per [[bar-sim-trailing-bug|Rule 1]]). RT is v14 default.

| Check | Result |
|---|---|
| Quarter-by-quarter (2025Q1–2026Q1) | 5/5 positive for portfolio |
| Month-by-month | 12/12 positive for portfolio |
| Outlier-month exclusion (drop best: Oct 2025) | Portfolio still +36.6 PPD ex-outlier |
| Walk-forward split-half (H1 vs H2) | OOS Cal > IS Cal (not overfit) |
| Cost sensitivity (+0 to +2 pt extra) | Cal remains 139–216 across range |
| Realistic cost (+1pt: 0.5 comm + 0.5 slip) | Cal 192, PPD 47.7 |
| Bonferroni (30 explicit hypotheses) | `with_1h_trend` CI_lo > 0 |

## Interaction with Tempo canon

**What still holds:**
- [[ifvg|IFVG]] as the entry mechanic is sound
- Soft stop at opposite FVG edge (close-through) is sound
- Session-aware TP structure works as-is
- Tick-level execution is required (not bar-sim)

**What the data suggests revising:**
- "CT is the primary confluence" — data says `with_1h_trend` is the baseline; CT is the secondary leg
- "Body/gap ≥ 1.0 (entry bar engulfs the FVG) is the SMT proxy" — lifts WR 85% → 94%, Cal 152 → 183
- Diversification can come from WITHIN IFVG (different trend filters on the same mechanic), not requiring [[tempo-portfolio-v15|Lumi]] or a second strategy
- Existing [[lumi-strategy-spec|v14 Lumi]] was a diversification tax (-1.66 pts/day on the shared 199-day window vs IFVG's +81 pts/day). V15 Lumi rewrite may have fixed it but isn't tested here.

## Interaction with [[smt|SMT]] concept

ES tick data is not available on this machine — SMT cannot be tested directly. The 2026-04-12 session used reaction-quality filters (body/gap, consec candles, vol expansion) as a post-hoc proxy for SMT's "did the reversal actually commit" check. The `body/gap ≥ 0.7` filter pushes WR from 85% → 93% with 258 trades remaining and 5/5 positive quarters — consistent with [[smt|SMT + reaction-quality]] lifting WR from 64% to 83% in the Tempo research corpus.

The working hypothesis: `body/gap ≥ 0.7` captures the same signal as "3 consecutive directional candles" or "46+ pt death candle" — it's a price-action proxy for the commit signature that SMT is supposed to confirm.

## Third leg (marginal)

A third disjoint leg — `ct=False AND 0.3 ≤ body/gap < 0.5` (excluded from Leg A and Leg B) — adds 74 trades with +5.18 PPD at correlation -0.012 to A and -0.038 to B. Combined A+B+C: Cal 256, PPD 54.14, DWR 78.5%. Marginal lift, but a genuine third orthogonal contribution.

## Not-validated caveats

- This is a backtest, not live/paper performance. Per [[invalidation-rules|Rule 15]]: "NEVER claim a strategy is validated based on backtesting alone."
- October 2025 contributes ~25% of Leg A's total — portfolio survives outlier removal but concentration warrants paper-trade before live.
- ES data absent — actual SMT confluence cannot be verified. The body/gap proxy may not generalize.
- The extraction used v14 TP defaults. TP sensitivity is in progress as of this synthesis write time; not yet validated.

## Artifacts

- Research output: `~/Documents/trading-system/results/autonomous_20260412/`
  - `MASTER_LOG.md` — full findings with every table
  - `rich_trades_full_260d.csv` — 1,455 trades with 33 features
  - `floored_risk5_260d.md` — walk-forward with pair/triple filters
  - `validate_primary.md` — month/quarter/outlier stability
  - `reaction_quality.md` — SMT proxy grid
  - `orthogonal_signals.md` — correlation search that found Leg B
  - `portfolio_validate.md` — two-leg portfolio validation
  - `cost_sensitivity.md` — commission/slippage stress test
  - `triplet_search.md` — third-leg search

- Scripts: `~/Documents/trading-system/scripts/autonomous/`

## Cross-references

- [[ifvg]] — the mechanic both legs share
- [[mtf-alignment]] — the single binary filter that separates the two legs
- [[smt]] — what body/gap proxies when ES data is unavailable
- [[invalidation-rules]] — the rule set this work was built against
- [[tempo-portfolio-v15]] — the live portfolio target this replaces if validated forward

## Memory cross-reference

Updates `~/.claude/projects/-Users-harrisonwillis/memory/MEMORY.md` — adds a "with_1h_trend beats counter_trend_1h for IFVG" finding that contradicts the existing "CT+M60+FVG≥5 = 82.9% WR sniper" claim. Both are in-session findings; the 2026-04-12 finding uses tick-level sim + full-year data + 5/5 positive quarters + cost sensitivity. The prior finding was from `/tmp` scripts that no longer exist and can't be re-verified.
