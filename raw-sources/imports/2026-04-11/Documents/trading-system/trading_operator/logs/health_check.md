# Tempo Trading System — Health Check & Inventory
**Generated:** 2026-02-17
**Repo:** `/Users/harrisonwillis/Documents/trading-system/`
**Branch:** `main`

---

## System Overview

Portfolio of non-correlated NQ futures signal models targeting 10R+/day combined.
- **Model 1** (QC): Sweep + IFVG, market orders, multi-TF — batch testing on VPS
- **Model 2** (Local Backtester): BOS/sweep/VA/PGF, limit orders, 5M+1M — validated +11.31R/day

---

## Root-Level Documentation

| File | Size | Purpose |
|------|------|---------|
| `CLAUDE.md` | 2.0 KB | Claude Code instructions — read order, key rules, portfolio mental model |
| `START_HERE.md` | 23.7 KB | Master session doc — current state, infrastructure, credentials, what's next |
| `CLAUDE_BOOTSTRAP.md` | 6.1 KB | Technical bootstrap — credential locations, GitHub token, QC API setup |
| `CONTEXT.md` | 10.6 KB | Strategy mechanics — both models' entry/exit logic, execution assumptions |
| `SETUP.md` | 1.6 KB | Initial project setup instructions |
| `.env` / `.env.example` | ~1 KB | Environment variables (gitignored) |
| `.gitignore` | 0.5 KB | Git ignore rules |
| `requirements.txt` | 0.3 KB | Python dependencies |
| `NQ_Condition_Matrix.xlsx` | 110 KB | NQ condition matrix spreadsheet |

## Root-Level Python Scripts

| File | Size | Purpose |
|------|------|---------|
| `intraday_scanner.py` | 29.5 KB | Core signal generator for Model 2 — macro context + multi-TF analysis + micro-structure patterns. Uses Databento tick data from `trading_operator/data2/`. |
| `micro_structure.py` | 33.4 KB | Pattern detection engine — builds candles from ticks, detects FVG, BOS, engulfing, sweeps, volume spikes, Fibonacci retracements. Outputs TradeSignal objects. |
| `qc_connect.py` | 14.8 KB | Self-contained QuantConnect API tool — lists projects, compiles, runs backtests, uploads files. Reads creds from `~/.env`. |
| `qc_pull_data.py` | 13.2 KB | Downloads ALL backtest results from QC — outputs CSV summary, full JSON, per-backtest trades, equity curves to `qc_data/`. |
| `qc_download_backtests.py` | 7.1 KB | REST API backtest downloader with 10 concurrent threads, resumable. |

## Root-Level Spec Documents

| File | Size | Purpose |
|------|------|---------|
| `Tempo_Context_Engine_Spec.docx` | 25.0 KB | V1 engineering spec for context engine build |
| `Tempo_Context_Engine_Spec_V1.1_Addendum.docx` | 17.7 KB | V1.1 addendum updates |
| `Tempo_Context_Engine_Spec_V1.2_Addendum.docx` | 18.2 KB | V1.2 addendum updates |

---

## Directories

### 1. `api-wrapper/` — Claude API Wrapper (FastAPI)
Anthropic Claude API wrapper for intelligent trading signal validation using prompt caching + knowledge base.

| File | Purpose |
|------|---------|
| `main.py` | FastAPI app — `/ask` endpoint, Anthropic client with prompt caching, async lifecycle |
| `config.py` | Pydantic settings — API key, model selection (claude-sonnet-4-5), max_tokens, paths |
| `knowledge_loader.py` | KnowledgeLoader class — markdown knowledge base with auto-reload, token estimation |
| `example_client.py` | Test client showing how to call `/ask` endpoint |
| `test_example.py` | pytest tests for API wrapper |
| `docker-compose.yml` / `Dockerfile` | Docker deployment config |
| `requirements.txt` | FastAPI + Anthropic dependencies |

---

### 2. `batch-runner/` — QuantConnect Batch Test Harness
2000-variant strategy backtester for Model 1 (QC Sweep+IFVG). Two-stage pipeline: Stage A screens, Stage B confirms. Running Batch 3 on VPS with cost modeling.

#### Main Code
| File | Purpose |
|------|---------|
| `main.py` (59.9 KB) | Tempo V3.34 — sweep + IFVG detection, market order entry, multi-TF scanning, dynamic stops, session gating |
| `main_v334_original.py` | Original V3.34 backup |
| `main_v4.py` (55.6 KB) | Tempo V4 Modular Brain — extends V3.34 with modular signals, 15S consolidator, HTF FVG, comprehensive trade tagging |
| `strategy_config.py` | StrategyConfig dataclass — 50+ tunable params (risk, gap thresholds, stop/TP multipliers, session times) |
| `generate_variants.py` | Generates 2000-variant JSON from cartesian product of parameter ranges |

#### Scripts (`scripts/`)
| File | Purpose |
|------|---------|
| `run_batch.py` | **Main production script** — reads variants, authenticates to QC, compiles, runs backtests with polling, outputs JSONL |
| `test_single_backtest.py` | Single-backtest test for API connectivity |
| `analyze_results.py` | Post-run result analysis — filters, aggregations, metrics |
| `generate_report.py` | Markdown report generation from batch results |
| `summarize_results.py` | Quick JSONL summary |
| `smart_generate.py` | Intelligent variant generation |
| `run_twostage.py` | Two-stage (A→B) pipeline runner |
| `upload_to_qc.py` / `.sh` | Upload strategy files to QC |
| `nightly_batch.sh` / `nightly_run.sh` | Automated nightly cron runs |
| `deploy_to_vps.sh` / `deploy_v4.sh` | VPS deployment scripts |
| `diagnose_and_fix.sh` | Troubleshooting script |
| `find_working_project.py` | Auto-discover working QC project |
| `test_params_verify.py` | Parameter validation for variants |
| `write_manifest.py` | Generate batch run manifest |

#### Variants & Results
| File | Purpose |
|------|---------|
| `variants_batch1.json` | Batch 1 variant set (small test) |
| `variants_batch2_2000.json` (967 KB) | Batch 2 — full 2000 variants |
| `variants_v5_optimized.json` | V5 optimized subset (winners from batch results) |
| `results/` | Batch run outputs — JSONL, summaries, Stage A/B results, cron logs |
| `results/batch3_costmodel/` | Batch 3 with slippage/fees/tick-through/BE cost modeling |
| `research/` | Research notes and analysis |
| `logs/` | Execution logs |

#### Documentation
| File | Purpose |
|------|---------|
| `README.md` | Full feature list, setup, auth flow, usage examples |
| `QUICKSTART.md` | 1-minute setup guide |
| `CHECKLIST.md` | Deployment verification checklist |
| `DELIVERY_SUMMARY.md` | Complete delivery overview |
| `EXAMPLES.md` | Practical usage examples |
| `IMPLEMENTATION_NOTES.md` | Architecture and design docs |
| `INDEX.md` | File navigation guide |
| `SYSTEM_STATE.md` | Current batch status and known issues |

---

### 3. `brains/` — Strategy Brain Framework
Abstract base class + registry for trading strategy brains. Each brain is an independent signal generator.

| File | Purpose |
|------|---------|
| `base_brain.py` | `BaseBrain` ABC — `scan()`, `validate()`, `calculate_size()` interface, hard/soft filter checks |
| `brain_registry.py` | `BrainRegistry` — register, activate/deactivate, list brains. Factory: `create_default_registry()` |
| `tempo_brain.py` | `TempoBrain` skeleton (v0.1.0) — placeholder scan/validate, awaiting strategy extraction |

---

### 4. `context-engine/` — Macro Context Orchestrator
Real-time market context aggregation. Outputs 5-channel JSON used by all brains.

#### Core Engine
| File | Purpose |
|------|---------|
| `context_engine.py` | V1 orchestrator — VIX, calendar, session, day type providers → `context.json` |
| `context_engine_v2.py` | V2 enhanced — Bayesian classifier (68.6% accuracy), opportunity score, vol regime, SMT, confidence → `context_v2.json` |
| `config.py` | V1 config — VIX thresholds, session times, hard/soft filter definitions |
| `strategy_bridge.py` | Per-signal-type gates — NOT binary. Maps session type → signal permissions with adjusted risk/quality/direction |
| `strategy_matrix.py` | Strategy matrix analysis |
| `live_processor.py` | Real-time tick processing + checkpoint scheduling for backtesting |
| `backtest_report.py` | 255-day performance report generator |
| `intraday_scanner.py` | Context engine's copy of intraday scanner |
| `micro_structure.py` | Context engine's copy of micro structure detector |

#### Feature Modules (`features/`)
| File | Purpose |
|------|---------|
| `bayesian_classifier.py` | Naive Bayes 5-class session type classifier (TREND/RANGE/EXPANSION/COMPRESSION/NEWS_SHOCK) |
| `volume_profile.py` | POC, Value Area, HVN/LVN from tick data |
| `footprint.py` | Delta, imbalance stacks, absorption detection from bid/ask |
| `smt.py` | NQ vs ES swing divergence (Smart Money divergence) |
| `vol_regime.py` | AlgoFlows severity score + session×volume matrix |
| `markov_model.py` | Session type transition model — Markov chain + streaks + day-of-week bias |
| `session_labeler.py` | Labels historical sessions by type for training |

#### Data Providers (`providers/`)
| File | Purpose |
|------|---------|
| `vix_provider.py` | Current VIX level → regime mapping (low/medium/high/crisis) |
| `calendar_provider.py` | Economic calendar — high-impact events, news blackout |
| `session_provider.py` | Current session detector + time remaining |
| `day_type_provider.py` | Inside/outside day, gap type, trend bias classification |

#### Data Ingestion (`data_ingestion/`)
| File | Purpose |
|------|---------|
| `youtube_ingest.py` | YouTube transcript ingestion (educator videos) |
| `discord_ingest.py` | Discord export ingestion (strategy discussions) |

#### Analysis Processing
| File | Purpose |
|------|---------|
| `process_full_year.py` | Batch processor for 255 Databento .dbn.zst trading days |
| `process_session_features.py` | Feature extraction pipeline |
| `train_classifier.py` | V1 classifier training (52.9% accuracy) |
| `train_classifier_v2.py` | V2 classifier training (68.6% accuracy) |
| `derive_gates.py` | Derives per-signal-type gates from historical data |
| `deep_session_analysis.py` | Deep-dive session pattern analysis |
| `edge_deep_dive.py` | Edge analysis by session type |
| `variant_analyzer.py` | Variant performance analysis |
| `auto_iterate.py` / `auto_iterate_v2.py` | Automated optimization iteration loops |

#### V7 Analysis Scripts (Full-Year Optimization)
| File | Purpose |
|------|---------|
| `run_v7_test.py` | V7 test harness |
| `run_v7aa_b_full_matrix.py` | Full cross-tab matrix (17 tables, 4776 trades) |
| `run_v7aa_candle_patterns.py` | Candle pattern analysis |
| `run_v7ab_counter_trades.py` | Counter-trade toxic combo analysis |
| `run_v7ab_b_limit_counter.py` | Limit entry counter-trade |
| `run_v7ab_c_be_target.py` | Breakeven + target management |
| `run_v7ac_management_matrix.py` | Position management variations |
| `run_v7ad_overlap_correlation.py` | Signal overlap & correlation |
| `run_v7ae_portfolio_optimizer.py` | Portfolio construction optimizer |
| `run_v7c_fast.py` / `_fullyear.py` / `_remaining.py` | Fast screening, full year, remaining period |
| `run_v7d_filtered.py` | Filtered results |
| `run_v7e_params.py` / `_portfolio.py` / `_quick.py` / `_single.py` / `_tiered.py` | Parameter studies, portfolio, tiered sizing |
| `run_v7f_daily_wr.py` / `_one.py` / `_tpd.py` | Daily win rate, single runs, trades/day |
| `run_v7g_walkforward.py` | Walk-forward validation (PASSED — all periods positive) |
| `run_v7h_sizing_v2.py` / `_v3.py` / `_tiered.py` | Position sizing experiments |
| `run_v7i_win_rate.py` | Win rate deep-dive |
| `run_v7k_inverse.py` / `_inverse_deep.py` | Inverse signal analysis |
| `run_v7l_walkforward_inverse.py` | Walk-forward inverse signals |
| `run_v7m_regime_switch.py` | Regime-switched portfolio |
| `run_v7r_session_candle_deep.py` | Session candle deep analysis |
| `run_v7rb_subminute_confirm.py` | Sub-minute confirmation |
| `run_v7s_stoplimit_deep.py` | Stop-limit deep analysis |
| `run_v7t_composite_gates.py` | Composite gate construction |
| `run_v7u_scanner_stoplimit.py` | Scanner stop-limit |
| `run_v7v_streak_sizing.py` | Streak-based sizing |
| `run_v7w_combined_portfolio.py` | Combined portfolio optimization |
| `run_v7x_session_allocation.py` | Session allocation |
| `run_v7y_full_optimized.py` | Full optimized config |
| `run_v7z_advanced_filters.py` / `_fix_wick_mechanics.py` | Advanced filters, wick mechanics fix |

#### V8 Analysis Scripts (New Signal Exploration)
| File | Purpose |
|------|---------|
| `run_v8a_overnight_mine.py` | Overnight session signal discovery |
| `run_v8b_new_triggers.py` | New entry trigger exploration |
| `run_v8c_gap_oi_analysis.py` | Gap + open interest analysis |
| `run_v8d_strategy_builder.py` | Dynamic strategy construction |
| `run_v8e_session_day_deep.py` | Session x day type deep-dive |

#### Trained Models (`models/`)
| File | Purpose |
|------|---------|
| `bayesian_classifier_v1.json` | V1 classifier (52.9% accuracy) |
| `bayesian_classifier_v2.json` | V2 classifier (68.6% accuracy) |
| `markov_model.json` | Session transition Markov chain |
| `best_config.json` / `optimal_configs.json` | Optimal parameter sets |
| `NQ_Condition_v1_config.json` | NQ condition config |
| `tradeable_rules.json` / `data_driven_gates.json` / `deep_session_edges.json` | Derived trading rules |
| `*_full_year.json` | Full-year results by version (v4a, V4a_pruned, V4e, V5c, V6) |
| `V7ae_portfolio_optimizer.json` | Portfolio optimizer results |
| `edge_deep_dive.json` / `iteration_log.json` / `intraday_scanner_results.json` | Analysis outputs |

#### Documentation
| File | Purpose |
|------|---------|
| `MINING_ANALYSIS.md` | Pattern mining results documentation |
| `QUICK_REFERENCE.md` | Quick reference for context engine variables and outputs |

---

### 5. `knowledge-base/` — Master Knowledge Base
| File | Purpose |
|------|---------|
| `master.md` (~80 KB) | System prompt for API Wrapper — brain definitions (5 strategies), hard/soft filters, core variables (P0-P3 priority), known issues/TODOs |

---

### 6. `models/` — ML Models (Root Level)
63 JSON files — Bayesian classifiers, Markov models, neural network configs, ensemble models trained on historical session/signal data.

---

### 7. `signals/` — Context Output
| File | Purpose |
|------|---------|
| `context_v2.json` | Current 5-channel context snapshot |
| `context_v2_*.json` (5 dated) | Historical context snapshots |

---

### 8. `openclaw-skills/` — Telegram Bot Integration
**Empty.** Placeholder for OpenClaw/Telegram bot notification skills (not yet implemented).

---

### 9. `orderflow-transcripts/` — Educator Content (~11.5 MB)
YouTube transcript library organized by educator for knowledge extraction.

| Subfolder | Content |
|-----------|---------|
| `axia-futures/` | ~30 transcripts — footprint chart trading, scalping |
| `bookmap/` | ~18 transcripts — order book visualization |
| `carmine-rosato-orderflow/` | ~20 transcripts — order flow analysis |
| `carmine-rosato-supply-demand/` | ~72 transcripts — supply/demand zones |
| `orderflows-michael-valtos/` | ~666 transcripts — massive order flow library |
| `tradepro-academy/` | ~22 transcripts — trading education |
| `trader-dale/` | ~22 transcripts — volume profile trading |
| `ALL_TRANSCRIPTS_COMBINED.txt` | 11 MB master file of all transcripts |

---

### 10. `qc-backtests/` — QuantConnect Backtest Cache
Downloaded backtest results from QC API.

| Directory | Purpose |
|-----------|---------|
| `detail_cache/` (502 entries) | Individual backtest detail JSONs |
| `orders/` (2658 entries) | Per-backtest order lists |
| `backtests_summary.csv` | Summary CSV (currently 0 bytes — needs regeneration) |

---

### 11. `qc_data/` — QC API Output
| Directory | Purpose |
|-----------|---------|
| `project_28083727/` | Project-specific downloaded data |

---

### 12. `trading_operator/` — Live Trading Operator
Supervisor process for autonomous task execution via Claude Code.

| Item | Purpose |
|------|---------|
| `supervisor.py` | Main loop — watches `watchers/inbox/` for tasks, runs via Claude Code, logs results |
| `config/` | Configuration files |
| `data2/` | Live Databento tick data |
| `logs/` | Execution logs |
| `memory/` | Persistent memory (CEO_STATE.md, CEO_KNOWLEDGE.md, SYSTEM_STATE.md) |
| `watchers/inbox/` | Incoming task files (.txt/.md) |
| `watchers/done/` | Completed task files |
| `.venv/` | Python virtual environment |

---

### 13. `tests/` — Unit Tests
**Empty.** Reserved for pytest test suite.

### 14. `backtests/` — Historical Backtest Data
**Empty.** Reserved for local backtest output.

### 15. `quantower-bridge/` — Quantower Integration
**Empty.** Placeholder for Quantower platform bridge.

---

## System Health Summary

| Component | Status | Notes |
|-----------|--------|-------|
| Model 2 (Local Backtester) | **Validated** | V7g walk-forward passed, +11.31R/day, 72% profitable, 13/13 months positive |
| Model 1 (QC Batch) | **Active** | Batch 3 running on VPS with cost modeling |
| Context Engine V2 | **Operational** | 5-channel output, 68.6% session classifier accuracy |
| Strategy Bridge | **Operational** | Per-signal gates (not binary) |
| API Wrapper | **Ready** | FastAPI + Docker, prompt caching enabled |
| Brain Framework | **Skeleton** | Base class + registry done, TempoBrain awaiting extraction |
| Trading Operator | **Active** | Supervisor loop running, inbox/done workflow |
| Knowledge Base | **Populated** | ~80 KB master.md feeding API wrapper |
| QC Backtest Cache | **Partial** | 502 details + 2658 orders cached, summary CSV empty |
| Tests | **Empty** | No unit tests written yet |
| Telegram Bot | **Not Started** | openclaw-skills/ empty |
| Quantower Bridge | **Not Started** | quantower-bridge/ empty |

---

*Total files: 200+ code/config/data/docs. Total repo size: ~500 MB (mostly transcript/cache data).*
