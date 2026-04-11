# Activity log

Append-only chronological record of vault operations. Entries use the format `## [YYYY-MM-DD HH:MM] <operation> | <description>` so they can be greppable:

```bash
grep "^## \[" log.md | tail -20
```

## [2026-04-10 15:27] setup | vault scaffolded

## [2026-04-10 17:12] ingest | COMPREHENSIVE_MINING_REPORT.md → wiki/summaries/comprehensive-mining-report-v2.md; concepts bos-fvg, mtf-alignment, v10i-look-ahead-bug, be-trail-mechanism

## [2026-04-10 17:12] ingest | STRATEGY_MINING_REPORT.md → wiki/summaries/strategy-mining-report-312d.md; flagged V1-era suspect-results (V10i contamination)

## [2026-04-10 17:12] ingest | V5_Strategy_Bug_Audit_2026-03-23.md → wiki/summaries/v5-strategy-bug-audit.md; concepts vwap-double-counting-bug; entity ninjatrader-v5

## [2026-04-10 17:12] ingest | Tempo_Context_Engine_Spec.docx → wiki/summaries/tempo-context-engine-spec.md; concepts volume-profile, session-type-taxonomy, bayesian-belief-engine

## [2026-04-10 17:12] ingest | Tempo_Quick_Start_Guide.docx → wiki/summaries/tempo-quick-start-guide.md; entity tempo-trading-system, quantconnect, databento

## [2026-04-10 17:12] synthesis | mining-reports-v1-v2-reconciliation.md — cross-source audit reconciling V1 97.9% WR vs V2 63.5% WR on BOS_FVG; three bias corrections documented
