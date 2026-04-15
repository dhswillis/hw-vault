---
created: 2026-04-14
updated: 2026-04-14
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE.md
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/tempo-methodology.md
  - wiki/concepts/ifvg.md
  - wiki/concepts/invalidation-rules.md
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/summaries/unified-brain-architecture.md
  - wiki/maps/strategies-moc.md
tags: [summary, canonical, trading, tempo, v13, or-fail]
---

# Trading System START_HERE — Summary

The master context document for every Claude session working on Tempo. Last updated March 6, 2026. Contains the single most complete state snapshot of all strategy research, validated systems, and infrastructure.

## Critical findings NOT yet captured elsewhere in the wiki

### V13 OR_FAIL Production System (Feb 22-23, 2026)

A completely separate production model from IFVG/Lumi. Based on Opening Range failure patterns on 5M bars with trail exit management.

**V13g Production Spec (OR_FAIL):** 164 trades/year, 89.0% WR, +65.6R, +0.26 R/day, Calmar 22.9, max DD 2.9R. SHORT-ONLY is the primary edge (84.4% WR). Longs rescued by delta filter (OR delta < 0). Trail: 0.3R trigger, 30% trail.

**V13k Extended:** Re-entry 3x variant: 395 trades, 83.5% WR, +109.9R, 0.43 R/day, Calmar 21.6. Third entries at 85.2% WR — better than second entries.

**V13m Final Portfolios:**
- SNIPER (OR_FAIL only): 154 trades, 88.3% WR, 0.25 R/day, Calmar 22.3
- SHARPSHOOTER (OR_FAIL 3x): 737 trades, 83.9% WR, 0.94 R/day, Calmar 59.5
- FULL (OR+IB+EXTREME): 525 trades, 84.6% WR, 0.54 R/day, Calmar 20.6, 96.3% weekly WR

**V13t DUAL OR System (final update):**
- DUAL_OR (Fail+Break): 1,048 trades, 83.1% WR, 1.20 R/day, Calmar 50.3, 94.4% weekly WR
- Every month green. Walk-forward PASS (H1 +180.8R, H2 +151.6R).

**V13h Bias Audit:** 8/10 checks passed. No look-ahead bias. Survives 3x commission. Random entries with same trail = 79% WR (trail asymmetry), but OR_FAIL adds +10pp WR AND 2.6x avg R per trade vs random.

**V13o Bug Found:** MICRO_SWEEP had look-ahead (5M bar start used as entry time). After fix: 78.6% WR = random trail baseline. MICRO_SWEEP is DEAD. OR_FAIL/IB_FAIL/SESSION_EXTREME verified clean.

### V7 Deep Mining (Feb 15-16, 2026)

**V7aa SKIP_CONTRA is the biggest single finding:** skipping trades where 5M candle contradicts direction yields: Portfolio Calmar 54.6 to 119.3 (+118%). sweep_reversal Calmar 4.8x improvement.

**V7y Optimized Portfolio:** Baseline 2917.9R, 9.35R/day, Calmar 61.6. Optimized: 2261.2R, 7.25R/day, MaxDD 16.4R, Calmar 137.8. Walk-forward stable.

**V7z VA_fade wait-for-first-win:** 50.9% WR, 1.268 avg R after first daily win. Production rule: ONLY trade VA_fade after it wins once that day.

### V8 Overnight Mining (Feb 17, 2026)

Walk-forward validated combined Strategy A+B: +12.07 R/day, 94.4% weekly WR, Calmar 31.4 on 94 OOS test days.

**V8g Multi-TF Candle Alignment:** Monotonic relationship — 0/6 confirming TFs = 5.7% WR, 6/6 = 40.2% WR. BOS_FVG + 0 contradicting = 77.9% WR.

### V9 Production Specification (Feb 18, 2026)

**V9f Production spec:** Tier 1+2 = 70% WR, +5.08 R/day, Calmar 235.7.

### Model 2 status (V10t clean, post-audit)

6 validated signals: engulfing_key_level, [[bos-fvg|BOS_FVG]], VA_fade, pre_gap_fill, ORB_breakout, vol_spike_FVG. Combined: 2,423 trades, +1.44 R/day, Calmar 17.4. Note: +1.44 R/day is approximately breakeven after commissions for 1 NQ contract.

7 dead signals: london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion.

### Doji Break (Mar 6-7, 2026)

Asia session doji break: 89% WR, ~59 R/day, Calmar 589. Walk-forward: H1 Cal 162 to H2 Cal 208 (OOS better than IS). B/E IS the edge — without B/E completely dead.

### Infrastructure

VPS at 172.93.181.209, QuantConnect project 28083727, Databento 312-day tick data (1.6GB), Context Engine V2 fully built (Bayesian session classifier, 68.6% accuracy, strategy bridge, Markov model), OpenClaw/Telegram bot configured.

## Suspect data warnings

All V7/V8/V9 Model 2 results use the local backtester (micro_structure.py / intraday_scanner.py). V10o audit found multi-TF alignment used forming bar closes (look-ahead bias). V10t is the clean version but only produces +1.44 R/day (near breakeven after commission). The V7-V9 headline numbers (Calmar 137+, +12 R/day) predate the V10o audit and should be cited with the look-ahead caveat.

V13 OR_FAIL is a separate codebase and was independently audited (V13h). It does NOT use the contaminated V7-V9 backtester.

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/START_HERE]]
