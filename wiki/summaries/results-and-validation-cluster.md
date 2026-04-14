---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS_V2.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/VALIDATION_SUMMARY.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/FINAL_SUMMARY.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/RESEARCH_LOG_20260304.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/BOS_FVG_RESEARCH.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/wickfade_5m_findings_20260303.md
  - raw-sources/imports/2026-04-11/Documents/strategies/tempo/scripts/AUDIT_RESULTS_20260329.md
  - raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/BACKTEST_ISSUES_LOG.md
  - raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/NT_AUDIT.md
  - raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec.md
related:
  - wiki/maps/audit-history-moc.md
  - wiki/summaries/ifvg-full-year-verification-memo.md
  - wiki/summaries/ifvg-optimization-report-v2.md
  - wiki/summaries/wickfade-5m-research-findings.md
  - wiki/summaries/bos-fvg-failure-consolidated.md
  - wiki/concepts/invalidation-rules.md
tags: [summary, cluster, results, validation, audit]
---

# Results & Validation — Cluster Summary

> 11 results/audit documents covering IFVG audit results (V1 and V2), validation summaries, BOS_FVG research, WickFade findings, SF portfolio backtest issues, and strategy-specific audit outputs.

## Contents

Collected results and validation artifacts from across the trading system:

**IFVG audits:** IFVG_AUDIT_RESULTS.md (V1) and IFVG_AUDIT_RESULTS_V2.md (V2) — code audit outputs for the IFVG implementation. See [[wiki/summaries/ifvg-full-year-verification-memo|IFVG Full Year Memo]] for the comprehensive version.

**Validation:** VALIDATION_SUMMARY.md and FINAL_SUMMARY.md — summary rollups of validation passes across strategies.

**Research logs:** RESEARCH_LOG_20260304.md — dated research session log. BOS_FVG_RESEARCH.md — BOS_FVG-specific research (invalidated, see [[wiki/summaries/bos-fvg-failure-consolidated]]).

**WickFade:** wickfade_5m_findings_20260303.md and WickFade_Strategy_Spec.md — earlier WickFade outputs. See [[wiki/summaries/wickfade-5m-research-findings|WickFade 5M]] for the comprehensive version.

**SF Portfolio:** BACKTEST_ISSUES_LOG.md and NT_AUDIT.md — bug tracking and NinjaTrader audit for the SF portfolio. AUDIT_RESULTS_20260329.md — March 29 audit pass.

## Status / caveats

Mixed trust levels. IFVG and WickFade results are generally trustworthy (tick-level). BOS_FVG research is **invalidated**. SF portfolio results should be cross-checked with [[wiki/summaries/sf-portfolio-cluster|SF Portfolio Cluster]].

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS_V2]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/IFVG_AUDIT_RESULTS]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/VALIDATION_SUMMARY]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/FINAL_SUMMARY]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/RESEARCH_LOG_20260304]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/BOS_FVG_RESEARCH]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/wickfade_5m_findings_20260303]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/tempo/scripts/AUDIT_RESULTS_20260329]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/BACKTEST_ISSUES_LOG]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/NT_AUDIT]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec]]
