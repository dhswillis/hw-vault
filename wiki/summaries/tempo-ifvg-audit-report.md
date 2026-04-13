---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/TEMPO_IFVG_AUDIT_REPORT.docx
related:
  - wiki/concepts/ifvg
  - wiki/summaries/tempo-v14-corrections
  - wiki/maps/audit-history-moc
tags: [summary, audit, tick-level, live-validation]
---

# Tempo IFVG V14 Tick-Level Audit Report — Summary

> 30-day (Mar 3—Apr 4, 2025) post-emergency-fix validation: US M60 +24.5 pts/day, London Fade +10.9 pts/day, London Breakout dead, 4 EOD trades carry 139% of profit.

## Key findings

**Corrected critical bug:** Emergency stop placed on profit side when price moved between sweep and FVG formation (30 fade trades affected). Fix: max(wick±1, entry±3) for proper stop placement. Previous +16.0 pts/day London breakout entirely driven by this bug.

**US M60 Strategy (105 trades):** 70.5% WR, +24.5 pts/day, Calmar 5.7. Shorts dominate (+39.1 pts/day), longs -2.2 pts/day. NYAM session strongest (+24.8 pts/day, 9.3 Calmar). Exit breakdown: BE_STOP 73 trades (100%, +23.9 pts/day), EMERGENCY 23 (0%, -22.4), EOD 1 (444.8 pts = 86% of total), SOFT_STOP 8 (0%, -23.4). **Critical:** Non-EOD PnL = +2.4 pts/day; one EOD trade carries 86%.

**London Fade Strategy (58 trades):** 46.6% WR, +10.9 pts/day, Calmar 1.2. 1HSH source strongest (+22.4 pts/day). Exit breakdown: BE_STOP 24 (100%, +1.2 pts/day), EMERGENCY 18 (0%), EOD 3 (310.4 pts = 247% of total), SOFT_STOP 13 (0%, -19.4). **Critical:** Non-EOD PnL = -12.7 pts/day; three EOD flatten trades produce 247% of profit. 8:30 AM flatten critical—without it, becomes -16.0 pts/day.

**London Breakout:** Tested across all TP1 distances (8-25pt), BE thresholds, trailing stops. Uniformly negative (-2 to -5 pts/day). Previously +16 pts/day was emergency stop bug artifact.

**Combined portfolio:** 163 trades, 62% WR, +31.9 pts/day, Calmar 4.8, 4 EOD trades = +1,066 pts (139% of total +767 pts). Non-EOD PnL = -10.3 pts/day. **Conclusion:** Strategy highly dependent on end-of-day liquidity capture; vulnerable to EOD flatness or adverse session close.

## Status / caveats

Period: Mar 3—Apr 4, 2025 (30 days, 21 trading days). Databento tick data, raw hard stops. This is CORRECTED version superseding prior report with inflated headline numbers. M60 filter essential for US sessions; kills London performance. H4 swing point FVG timing critical—pre-fix report had wrong sequence. Soft stop close-based vs hard stop tick-based creates asymmetry. Strategy highly concentrated in 4 EOD trades—scalability risk if EOD conditions worsen.
