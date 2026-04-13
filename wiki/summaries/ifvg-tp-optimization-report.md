---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/IFVG_TP_Optimization_Report.docx
related:
  - wiki/concepts/ifvg
  - wiki/maps/tempo-moc
tags: [summary, take-profit, optimization, risk-reward]
---

# IFVG Take-Profit Optimization Report (v15.1) — Summary

> Session-aware partial exits (legacy) underperform simple 1.25R full exit by 2.3x (52.2R vs 122.3R); v15 implements 1.25R target.

## Key findings

**Problem:** Original system exited 55% at TP1 (17pts), 35% at TP2 (35pts), 10% runner to session end. 237 identical entry signals across 280 days showed 82% win rate but profit concentrated in 6% of trades (FLATTEN outcomes +100.3pts avg). Trade distribution: 63% BE_STOP (+6.7pts), 6% FLATTEN (+100.3pts all profit), 13% SOFT_STOP (-35.7pts loss), 13% LIQ_BE (+4.7pts), 5% EMERGENCY (-26.2pts).

**Solution:** Replace with simple 1.25R full-exit target (entry ± risk × 1.25). Comprehensive sweep across 1.0R–2.0R in 0.25R increments found 1.25R optimal: +122.3R total (0.437 R/day), Calmar 7.9, PF 2.16. vs original 52.2R (0.186 R/day), Calmar 2.4. Alternative 1.5R safer (Calmar 9.1) but lower R/day.

**Key insight:** Adding break-even at 1R converted would-be winners into zero-profit exits before reaching target (-18% to -20% impact). Full exit superior to partial management. Slippage + commission already included (0.75pt).

**Implementation:** IFVGRTarget parameter (default 1.25) in TempoPortfoliov15.cs. Pipeline: Entry → Risk/Reward calc → Position opened → Hard stop at entry - RiskSize → Exit at 1.25R or hard stop.

## Status / caveats

Report dated March 22, 2026. Chart comparing configurations shows 1.25R clearly dominant. Tested on 237 signals; no forward validation mentioned. Assumes current entry logic is sound (audit report flags 2 CRITICAL look-ahead biases; verify entry quality before trusting TP optimization). Legacy partial system still available as toggle if needed for risk management purposes.
