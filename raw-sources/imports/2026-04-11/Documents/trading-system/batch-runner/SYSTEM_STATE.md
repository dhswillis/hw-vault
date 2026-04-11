# TEMPO — System State

> Single source of truth. Updated after every deploy and nightly run.
> Paste the top of this file into any AI session to restore full context.

## Architecture

```
Mac (author) → deploy_v4.sh → VPS (orchestrator) → QC API (compute)
                                    ↓
                              nightly_batch.sh (cron 10PM CT)
                                    ↓
                         generate variants (smart or random)
                                    ↓
                         run_batch.py → QC backtests (400/night)
                                    ↓
                         analyze_results.py → hot_zones → next night
```

## Credentials & Endpoints

- **VPS**: root@172.93.181.209 (QuantVPS, Ubuntu 24.04)
- **QC User**: 113205
- **QC Project**: 28083727
- **QC API**: https://www.quantconnect.com/api/v2/
- **QC Auth**: SHA256(token:timestamp) → Basic auth + Timestamp header
- **QC File Limit**: 64,000 bytes per file
- **Databento Key**: saved in /root/.env (not yet integrated)
- **Telegram Bot**: exists (not yet connected to results)
- **Cron**: `0 4 * * *` = 10 PM CT nightly

## File Inventory

### Strategy (runs on QC)
| File | Location | Purpose |
|------|----------|---------|
| main_v4.py | local + VPS + QC (as main.py) | The strategy. 930 lines, 47KB compressed. |

### Scripts (run on VPS)
| File | Purpose |
|------|---------|
| scripts/nightly_batch.sh | Cron entry point. Generates → runs → analyzes → extracts hot zones. |
| scripts/run_batch.py | Batch orchestrator. Compiles once, loops variants via QC API. |
| scripts/analyze_results.py | Post-batch analysis. Ranks variants, dimension breakdowns, hot zones. |
| scripts/smart_generate.py | Adaptive variant gen. 60% exploit hot zones, 40% explore random. |
| scripts/upload_to_qc.py | Pushes code files to QC via REST API. |
| scripts/test_single_backtest.py | Smoke test — one default backtest. |
| scripts/test_params_verify.py | 10 hand-picked variants to confirm params are applied. |

### Deployment (run on Mac)
| File | Purpose |
|------|---------|
| scripts/deploy_v4.sh | One-command deploy: scp all files to VPS + upload to QC. |
| generate_variants.py | V4 variant generator (400 variants, 4 tiers). |

## Strategy Architecture (V4)

### Signals Implemented
- **signal_sweep_fvg**: Liquidity sweep → inverse FVG entry (original V3.34 logic)
- **signal_htf_fvg_fill**: 5M/15M FVG fill with 15-second entries

### Signals Not Yet Built
- signal_15s_sweep, signal_15s_break, signal_news_straddle, signal_news_fvg, signal_bpr_retest

### Configurable Parameters Per Variant
- **enabled_signals**: which signals are active
- **r_targets**: TP per signal (0.33, 0.5, 0.75, 1.0, 1.5, 2.0, 2.5, 3.0, 5.0, 10.0, 20.0, 40.0R)
- **be_mode**: breakeven trigger (0.25R, 0.5R, liquidity, 1.0R)
- **session_filter**: all, ny_am, ny_pm, london, overlap
- **inside_day_mode**: both, inside_only, outside_only
- **risk_pct**: 0.003–0.007
- **max_contracts**: 1 or 2
- **gap params**: gap_min_optimal, gap_max_optimal, gap_min_small, gap_skip

### Stats Tracked Per Backtest
- Per signal type (win/loss/R by signal)
- Per entry timeframe
- Per session
- Per day of week
- Per news proximity (none/pre/during/post)
- Per inside/outside day
- Per exit reason (TP, BE_STOP, SOFT_STOP, HARD_STOP, EOD)
- BE effectiveness (triggered, saved, converted to win)

## Current Status

### Last Deploy
- **Date**: 2026-02-13
- **V4 uploaded to QC**: Yes (project 28083727, main.py, 47KB)
- **Single backtest test**: PASSED (compiled, produced trades)

### Verification Gate
- **10-variant param test**: NOT YET RUN
- **Status**: Must confirm different configs → different results before running 400/night

### Known Issues
- QC file limit is 64K — main_v4.py must stay compressed
- QC update API sometimes fails; upload_to_qc.py has delete+create fallback
- Previous sessions showed all backtests returning identical results (params not passed) — MUST verify this is fixed

### Nightly Pipeline Status
- Smart generation (explore/exploit): BUILT, not yet deployed
- Auto-analysis: BUILT, not yet deployed
- Two-stage screening: BUILT, not yet deployed
- Telegram alerts: NOT YET BUILT
- Git repo: NOT YET INITIALIZED

## Run History

| Date | Variants | Mode | Top Variant | Top Sharpe | Notes |
|------|----------|------|-------------|------------|-------|
| (no runs yet) | | | | | |

## Decision Log

| Date | Decision | Reasoning |
|------|----------|-----------|
| 2026-02-13 | Compress main_v4.py from 78K to 47K | QC 64K file limit |
| 2026-02-13 | Add 0.33R and 0.75R targets | Finer TP granularity |
| 2026-02-13 | 4 BE modes (0.25R, 0.5R, liquidity, 1.0R) | Test where to move stop to entry |
| 2026-02-13 | 400 variants/night with 4 tiers | Balance coverage vs compute |
| 2026-02-13 | Explore/exploit adaptive gen | Stop wasting compute on known bad space |
| 2026-02-13 | Two-stage screening | Stage A: 3mo (Jul-Sep 2025) → Stage B: 12mo (Oct 2024-Sep 2025) |
| 2026-02-13 | Configurable backtest dates | main_v4.py reads bt_start/bt_end params, defaults to Jul-Sep 2025 |
| 2026-02-17 | 400 | smart |   1. exploit_0042 |
