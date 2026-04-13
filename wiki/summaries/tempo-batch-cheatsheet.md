---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/Tempo_Batch_Cheatsheet.docx
related:
  - wiki/entities/quantconnect.md
  - wiki/entities/tempo-trading-system.md
  - wiki/maps/tempo-moc.md
tags: [summary, operations, batch-testing]
---

# Tempo Batch Cheatsheet — Summary

> Quick-reference guide for running Tempo V3.34 parameter sweeps on QuantConnect via the VPS batch runner.

## Key findings

This is an operational cheat sheet, not a research document. It walks through four steps: SSH into the VPS, run batch backtests with configurable variant counts (100 variants takes roughly 3–4 hours), set up a cron job for nightly 2 AM auto-runs, and check results the next morning. Results save to `/root/trading-system/batch-runner/results/` with date-stamped folders.

The batch system generates hundreds of parameter combinations (different risk levels, stop sizes, targets) and tests them all on [[wiki/entities/quantconnect|QuantConnect]] without burning AI credits. This is the infrastructure that feeds the mining reports and variant analyses elsewhere in the vault.

## Status / caveats

Operational guide — no analytical claims to validate. Assumes VPS credentials and QC project are already configured.
