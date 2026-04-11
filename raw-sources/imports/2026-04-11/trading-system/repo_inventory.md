# Tempo Trading System — Repository Inventory

**Repository:** dhswillis/trading-system (Private)  
**Owner:** Harrison (@prop_profitable)  
**Last Updated:** Feb 16, 2026  
**Primary Language:** Python (98.1%) | Shell (1.8%) | Dockerfile (0.1%)  

## Repository Overview

Tempo is an autonomous NQ futures backtesting and optimization platform built on QuantConnect. It uses a portfolio of non-correlated signal models targeting 10R+ per day combined. The system includes batch runners, context engines, and various analysis tools.

## Directory Structure & Contents

### Root Level Files

**Documentation:**
- `CLAUDE.md` - Claude Code instruction guide
- `CLAUDE_BOOTSTRAP.md` - Technical setup, credentials, integration configs
- `CONTEXT.md` - Strategy mechanics, signal types, execution parameters, batch results
- `START_HERE.md` - Main entry point with full project context (35KB+)
- `SETUP.md` - MacBook Air setup instructions

**Configuration:**
- `.env.example` - Environment template
- `.gitignore` - Git ignore rules
- `requirements.txt` - Python dependencies

**Strategy Specifications:**
- `Tempo_Context_Engine_Spec.docx` - Full engineering spec (46 features, 9 checkpoints)
- `Tempo_Context_Engine_Spec_V1.1_Addendum.docx` - V1.1 improvements
- `Tempo_Context_Engine_Spec_V1.2_Addendum.docx` - V1.2 addendum (vol regime, SMT)
- `NQ_Condition_Matrix.xlsx` - Trading condition matrix

**Root-Level Backtester Scripts:**
- `intraday_scanner.py` - 13 signal strategies, confluence scoring, session management
- `micro_structure.py` - Core pattern engine (FVG, BOS, sweeps, engulfing, volume spikes)
- `qc_connect.py` - Self-contained QC API tool for local execution (Feb 16)
- `qc_pull_data.py` - Download all backtest results locally (Feb 16)

### batch-runner/

**Main Strategy Files:**
- `main.py` - V3.34 strategy (minimal filters, sweep + IFVG)
- `main_v4.py` - V4 modular strategy (QuantConnect format)
- `main_v334_original.py` - Original V3.34 implementation
- `generate_variants.py` - 433 lines, variant generator with 4 modes:
  - `random` - Random sampling from sweep ranges
  - `grid` - Exhaustive grid generation
  - `manual` - Single variant from JSON
  - `v4` - Signal-aware variants (40% single-signal, 20% combos, 20% session-filtered, 20% full parameter sweeps)

**Configuration & Data:**
- `strategy_config.py` - Default config template
- `variants.json.example` - Example variants file
- `variants_batch1.json` - Batch 1 variants
- `variants_batch2_2000.json` - Batch 2 (2000 variants, 290 winners)
- `variants_v5_optimized.json` - Optimized variant set

**Documentation:**
- `README.md` - Comprehensive batch runner documentation
- `CHECKLIST.md` - Pre-flight checklist
- `DELIVERY_SUMMARY.md` - System delivery status
- `EXAMPLES.md` - Usage examples
- `IMPLEMENTATION_NOTES.md` - Technical implementation notes
- `INDEX.md` - Index of files
- `QUICKSTART.md` - Quick start guide
- `SYSTEM_STATE.md` - Current system state
- `.gitignore` - Local ignores

### batch-runner/scripts/ (18 files)

**Python Scripts:**
- `run_batch.py` - Main batch runner for multiple variants
- `run_twostage.py` - Two-stage batch runner (Stage A screen → Stage B confirm)
- `upload_to_qc.py` - Upload strategy to QuantConnect
- `generate_report.py` - Generate results report
- `analyze_results.py` - Analyze batch results
- `summarize_results.py` - Summarize results
- `test_single_backtest.py` - Test single backtest for API connectivity
- `test_params_verify.py` - Verify parameter passing
- `find_working_project.py` - Find working QC project
- `smart_generate.py` - Intelligent variant generation
- `write_manifest.py` - Write manifest for batch runs

**Shell Scripts:**
- `nightly_batch.sh` - Nightly batch runner (cron)
- `nightly_run.sh` - Nightly execution script
- `deploy_to_vps.sh` - Deploy to VPS
- `deploy_v4.sh` - Deploy V4 strategy
- `diagnose_and_fix.sh` - Diagnostic and fix script
- `run_batch.sh` - Shell-based batch runner
- `upload_to_qc.sh` - Shell-based QC upload

### context-engine/

**Status:** V2 Fully Built (Feb 2026)

**Core Orchestration:**
- `context_engine.py` - V1 orchestrator (VIX, calendar, session, day type)
- `context_engine_v2.py` - V2: 5-channel output (session probs, opp score, vol, SMT, confidence)
- `strategy_bridge.py` - Maps V2 output → StrategyConfig overrides
- `live_processor.py` - Real-time tick processing + checkpoint scheduling
- `run_v2.py` - CLI: replay/backtest/demo modes
- `backtest_report.py` - Full 255-day performance report
- `process_full_year.py` - Batch processor for Databento .dbn.zst files
- `variant_analyzer.py` - Context-tagged batch variant analyzer

**Training & Classification:**
- `train_classifier.py` - V1 training (52.9% accuracy)
- `train_classifier_v2.py` - V2 training (68.6% accuracy, 5-class)

**Features Module (context-engine/features/):**
- `volume_profile.py` - POC, Value Area, HVN/LVN from tick data
- `footprint.py` - Delta, imbalance stacks, absorption detection
- `smt.py` - NQ vs ES swing divergence (SMT)
- `bayesian_classifier.py` - Naive Bayes 5-class classifier
- `vol_regime.py` - AlgoFlows severity score + session×vol matrix
- `markov_model.py` - Session type transition model (Markov chain + streak + DOW)
- `session_labeler.py` - Auto-labels days for training (rules-based)

**Trained Models (context-engine/models/):**
- `bayesian_classifier_v1.json` - V1 classifier model
- `bayesian_classifier_v2.json` - V2 classifier model (68.6% accuracy)
- `markov_model.json` - Trained Markov transition matrix

**Data Providers (context-engine/providers/):**
- VIX, calendar, session, day type providers

### api-wrapper/

Initial scaffold directory for API wrapper integration.

### brains/

Initial scaffold directory for brain framework components.

### knowledge-base/

Initial scaffold directory for knowledge base system.

### models/

Directory for trained models and model artifacts.

## Key Files Extracted

### Markdown Documentation (5 files)
1. `/sessions/nice-brave-gates/mnt/trading-system/START_HERE.md` (35KB)
   - Main entry point with full project context, infrastructure map, credentials
   
2. `/sessions/nice-brave-gates/mnt/trading-system/CONTEXT.md`
   - Strategy mechanics, two-model architecture, execution parameters
   
3. `/sessions/nice-brave-gates/mnt/trading-system/CLAUDE_BOOTSTRAP.md`
   - Technical setup, credential locations, integration configs
   
4. `/sessions/nice-brave-gates/mnt/trading-system/CLAUDE.md`
   - Claude Code instructions, key rules, document hierarchy
   
5. `/sessions/nice-brave-gates/mnt/trading-system/SETUP.md`
   - MacBook Air setup guide, initial installation steps

### Python Files

#### generate_variants.py (433 lines)
- **Location:** `/sessions/nice-brave-gates/mnt/trading-system/batch-runner/generate_variants.py`
- **Purpose:** Create variant parameter sets for Tempo V4 batch backtesting
- **Modes:**
  - Random: sampling from sweep ranges
  - Grid: exhaustive grid generation
  - Manual: single variant from JSON
  - V4: signal-aware variants with 4 tiers (40% single-signal, 20% combos, 20% session-filtered, 20% full sweeps)
- **Key Features:**
  - V3.34 sweep ranges (tp_multiplier, be_multiplier, risk_pct, stop parameters, gap sizes, quality thresholds)
  - R-targets weighted toward 5-20R hot zone
  - Session filters and inside-day modes
  - Validation functions for parameter coherence

#### main.py (Large file)
- **Location:** `/sessions/nice-brave-gates/mnt/trading-system/batch-runner/main.py`
- **Status:** Unable to extract in full (file too large for browser extraction)
- **Description:** V3.34 strategy with minimal filters, sweep + IFVG only
- **Key Classes:**
  - SweepEvent: Liquidity sweep tracking
  - TempoTradeContext: Trade context management

## Statistics & Data

**Backtester Results (V7f - Full Year Validated, Optimized):**
- Best Config: tc200 rr2.0 no-kill → +11.31R/day
- 258 days, 4,991 trades, 26.5% WR, +2,918R total
- 72% profitable days, 13/13 months positive
- Max drawdown: 47R, Median daily R: +8.7
- 19.3 trades/day average, 5.1 wins/day

**Walk-Forward Validation (V7g - PASSED):**
- First half: +12.58R/day | Second half: +10.04R/day
- 0/13 months negative (all months profitable)
- Q3 worst quarter: +7.01R/day

**Batch 2 Results (QC - Pre-Cost-Modeling):**
- 2000 variants generated
- 290 winners found (Batch 2 Stage A)
- Top performer: v4_sig_0142 at 106.2% / $53K

**Context Engine Accuracy:**
- V1: 52.9% test accuracy
- V2: 68.6% test accuracy on 5-class
- TREND: 100%, NEWS_SHOCK: 100%

## Infrastructure Map

**Development:**
- Mac: Claude Code (CLI), Cowork (Desktop), OpenClaw (Telegram bot)

**VPS:** QuantVPS Pro (172.93.181.209)
- Batch runner code: ~/trading-system/batch-runner/
- Context engine: ~/trading-system/context-engine/
- Credentials: /root/.env
- Nightly cron: 10 PM CT (nightly_batch.sh)

**Cloud Services:**
- QuantConnect: Project 28083727 (NQ futures backtesting)
- Databento: 255 trading days (Feb 2025 - Feb 2026), 312 files (~1.6GB)
- GitHub: dhswillis/trading-system (source of truth)

## Known Issues & Status

**Resolved:**
- B/E kills edge (validated full year) - NO B/E on backtester signals
- Inverse signal logic (V7k/V7l/V7m resolved) - use as hedge, not replacement
- Walk-forward validation (V7g PASSED)
- Batch 3 running with cost modeling on VPS

**In Progress:**
- Batch 3 (Model 1 - QC) with cost modeling (slippage, fees, tick-through, BE offset)
- Context engine V1 build (day-type classification + volume features)
- OpenClaw tempo-trading skill for Telegram monitoring

**Known Limitations:**
- QC has 64K file size limit (main_v4.py compressed to 47K)
- Cowork sandbox blocks outbound HTTP (use Claude Code for API/SSH work)
- Git index.lock on mounted filesystem (use fresh clone in /sessions/)
- Vol regime uses static VIX=18.5 (needs live VIX feed)
- RANGE classification accuracy is weakest (43.7%)

## Session Checklist

Before ending sessions:
1. Update START_HERE.md "Current State" section
2. Commit and push to GitHub
3. Update CLAUDE_BOOTSTRAP.md if credentials changed
4. Document new tools/bots in "Built Integrations"

## Git Information

- **Current Commit:** 076639ecaf7d0d90a337d5c86d9e2b7557dc4d96
- **Latest Message:** "V7ae: Portfolio optimizer — 20R/day target construction"
- **Total Commits:** 44
- **Last Updated:** Feb 16, 2026

---

**Summary:** The Tempo trading system is a comprehensive NQ futures backtesting and optimization platform with two models: QC-based sweep+IFVG (Model 1) and local backtester with BOS/sweep/VA signals (Model 2). The system includes full context engine (V2), batch runners with variant generation, and extensive documentation. Code is actively maintained with daily git commits and validated through walk-forward testing.
