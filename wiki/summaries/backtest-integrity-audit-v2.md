---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/audits/BACKTEST_INTEGRITY_AUDIT_V2.docx
related:
  - wiki/maps/audit-history-moc
  - wiki/concepts/invalidation-rules
tags: [summary, audit, backtest, look-ahead-bias, trading]
---

# Backtest Integrity Audit V2 — Summary

> Willis Holdings NQ backtest contains 13 critical-to-low findings; 2 CRITICAL look-ahead biases inflate results 15-25%; Model 2 backtester is clean.

## Key findings

Comprehensive tick-level audit of main_v4.py (1,066 lines) against Databento tick data across 260 trading days. Main findings: **F1 & F2 (CRITICAL)** — Same-bar FVG detection and entry introduces 15-25% performance inflation via look-ahead bias. FVGs detected using current bar (bars[0]) immediately enter on same bar, but live trading would enter only on bar N+1. **F3-F5 (HIGH)** — Sweep detection includes current bar, entry uses bar.Close instead of realistic fill price, exit uses bar.Close instead of stop price. Together these create 5-10% additional inflation. **F6-F9 (MEDIUM)** — Tick-through check uses same-bar data, simultaneous entry guard missing, ATR from 1M only, double-counting in stats. **F10-F13 (LOW-INFO)** — Config loaded twice, EOD flatten, loss counter reset, Model 2 clean.

Slippage resolved to 0.75pt/side (conservative for RTH, accounts for illiquid fills). Round-trip cost now $34.50 vs $44.50 old (0.75pt vs 1.0pt slippage). All findings unresolved except those resolved by F1 fix when implemented.

## Status / caveats

**Critical blockers:** F1 & F2 require architectural fix (store signals in pending list, process on next bar). High-risk estimates: 15-25% combined inflation on backtests. Model 2 validation shows clean execution, suggesting the issue is backtest-only. Unclear if live [[wiki/concepts/invalidation-rules]] already guard against same-bar entry in production.
