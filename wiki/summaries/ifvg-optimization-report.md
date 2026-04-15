---
created: 2026-04-14
updated: 2026-04-14
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT.md
related:
  - wiki/concepts/ifvg.md
  - wiki/entities/tempo-methodology.md
  - wiki/summaries/tempo-ifvg-research.md
  - wiki/summaries/lumi-strategy-spec.md
tags: [summary, trading, ifvg, lumi, optimization]
---

# IFVG Optimization Report — Summary

March 9-10, 2026 research report testing Tempo IFVG, LumiTraders HTF Sweep, and Leppyrd/Omega models across 312 days of NQ tick data.

## Baseline (no filters)

8,150 trades (26.1/day), 68.5% WR, +0.680 avg R, +17.8 R/day, $4,877/day per contract, Calmar 87.6. Walk-forward improving (6.8 to 12.1 R/day).

## Key filter findings

**FVG Size:** 7-10pt FVGs are the quality sweet spot (+0.961 avg R, 71.2% WR). FVG 1-3pt is noise (+0.500 avg R). Trade-off between volume (3pt min) and quality (7pt min).

**Mean60 Bias (Premium/Discount):** With-bias trades: 74.6% WR, Calmar 161.8. Against-bias: 62.3% WR, Calmar 13.4. Simple SMA filter doubles Calmar.

**Systems tested:** The report covers multiple filter dimensions including FVG size bins, directional bias, session timing, and risk parameters. Each evaluated on WR, avg R, R/day, Calmar, and walk-forward stability.

## Cross-reference to later audits

This report uses the IFVG backtester which was later validated at tick level. However, the V10o audit found look-ahead in multi-TF alignment — results here that use MTF alignment should be read with that caveat. The FVG size and Mean60 findings are not affected by the MTF bug.

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_OPTIMIZATION_REPORT]]
