# Excel Files Strategy Overview
**Generated:** 2026-02-18

## Summary of Key Trading Strategy Files

### 1. NQ_Condition_Matrix.xlsx
**Purpose:** Comprehensive condition matrix for Tempo Model 2 trading system
**Size:** 38 sheets | 135 KB
**Data Coverage:** 4,991 trades across 258 days (Jul-Sep 2025)
**Config:** tc200 rr2.0 no-kill

**Key Metrics (Overview Sheet):**
| Signal | Trades | WR% | R/Trade | R/Day | Total R |
|--------|--------|-----|---------|-------|---------|
| BOS_FVG | 1,745 | 31.2% | 1.037 | 7.24 | 1809.1 |
| sweep_reversal | 1,191 | 34.4% | 0.393 | 1.84 | 468.4 |
| VA_fade | 1,967 | 17.7% | 0.306 | 2.60 | 601.0 |
| pre_gap_fill | 88 | 21.6% | 0.447 | 0.54 | 39.4 |
| **ALL** | **4,991** | **26.5%** | **0.585** | **11.31** | **2,917.9** |

**Top Portfolio Candidates (Best R/day):**
1. **dow=Thu** → 15.33 R/day, 27.6% WR (943 trades)
2. **dow=Thu + conf≥4** → 12.56 R/day, 28.7% WR (659 trades)
3. **dow=Wed** → 12.2 R/day, 27% WR (1,076 trades)

**Content Sheets:**
- Overview, Portfolio Candidates, Sizing Rules
- Pattern × Signal matrices (5M, 1M, 2-Bar, 3-Bar patterns)
- Pattern × Session, Pattern × DOW analysis
- Signal × Session, Signal × DOW matrices
- Alignment analysis (Signal, Session, DOW)
- Counter-Trade Summary, BE+Target Matrix
- Management matrices, Portfolio overview
- PnL correlation, Clustering impact, Concurrent exposure
- Production Config v1, QC Dimensions

---

### 2. Tempo_Batch2_Winners.xlsx
**Purpose:** Analysis of top performers from Batch 2 testing
**Size:** 3 sheets | 47 KB
**Data Coverage:** 290 winners from 2,000 submitted variants (842 completed)
**Period:** Jul-Sep 2025 (3-month Stage A)

**Summary Statistics:**
- Total Winners: 290
- Win Rate (batch): 34.4%
- **Top Performer:** v4_sig_0142
  - Net Return: 106.2% ($53,087)
  - Sharpe: 12.584
  - Safe Rate: 60.3%

**Sheets:**
1. **Summary** - Batch overview & top performer
2. **Winners (All 290)** - Complete list of winning variants
3. **Top 50 Safe Rate** - Safest performers by risk metrics

---

### 3. Tempo_Inverse_Analysis.xlsx
**Purpose:** Analysis of counter-trade (inverse) signals as hedge mechanisms
**Size:** 6 sheets | 15 KB
**Data Coverage:** Same 4,991 trades analyzed for directional accuracy

**Key Discovery:**
- System entries are **57.7% directionally accurate at 1R**
- Headline 26.5% WR is at 2R+ targets
- **Inverse trades (42.3% WR) alone are NOT independently profitable**
- **BUT: Inverse trades reduce drawdown 28-34% when used as hedges**
- **Calmar ratio improves significantly with regime-switching approach**

**Analysis Breakdown:**
| Metric | Baseline (no B/E) | With B/E at 1R | Inverse (all) | Best Regime |
|--------|-------------------|----------------|---------------|-------------|
| Total Trades | 4,991 | 4,991 | 4,991 | 4,169 + 822 inv |
| Win Rate | 26.5% | 57.7% @1R | 42.3% | Mixed |
| R/day | +11.31 | +0.09 | -2.99 | +10.09 |

**Sheets:**
1. Overview - Key discovery summary
2. Walk-Forward - Temporal stability analysis
3. Regime Switch - Optimal switching strategies
4. Top Inverse Combos - Best counter-trade combinations
5. Regime Analysis - Detailed regime metrics
6. Recommendations - Implementation guidelines

---

### 4. Tempo_Batch3_Results.xlsx
**Purpose:** Quality control testing of 50 best variants from Batch 3
**Size:** 6 sheets | 25 KB
**Data Coverage:** Backtest Jul 1 - Sep 30, 2025 | 400 submitted → 50 passed QC

**Tier Breakdown:**

| Tier | Count | Avg Score | Avg Sharpe | Avg DD% | Avg Net Profit | Avg Trades | Avg WR% |
|------|-------|-----------|-----------|---------|-----------------|-----------|---------|
| **ELITE** | 3 | 583.6 | 1.871 | 20.0% | $52,596 | 631 | 15.0% |
| **STRONG** | 14 | 299.1 | 0.991 | 37.15% | $27,350 | 862 | 14.0% |
| **VIABLE** | 8 | 153.3 | 0.572 | 33.4% | $14,007 | 782 | 12.1% |
| **MARGINAL** | 7 | 66.2 | 0.275 | 29.8% | $5,953 | 743 | 14.7% |
| **LOSING** | 16 | -174.3 | -0.752 | 49.6% | -$14,297 | 853 | 12.3% |

**Sheets:**
1. Overview - Tier summary & key findings
2. All Variants - Individual variant metrics
3. Stability - Stability across timeframes
4. Dimensions - Dimensional analysis
5. Production - Production-ready config
6. Model 2 Cross-Ref - Reference to Model 2

---

### 5. Tempo_V8f_Strategy_A_Results.xlsx
**Purpose:** Latest V8f strategy rebuild with full out-of-sample validation
**Size:** 12 sheets | 28 KB
**Data Coverage:** 312 trading days | 218 train / 94 test days
**Date:** Feb 17, 2026 | Databento NQ E-mini tick data

**Headline Results (Out-of-Sample, 94 test days):**

| Candidate | Total R | Trade WR | Daily WR | Weekly WR | Trades/Day | Max DD | Calmar | Verdict |
|-----------|---------|----------|----------|-----------|-----------|--------|--------|---------|
| A1: sweep_both_pro | 35.33 | 84.6% | 90.0% | 44.4% | 1.3 | -1.16 | 30.5 | High WR, low volume signal |
| A2: sweep_conf≥3 | 176.46 | 37.4% | 61.3% | 77.8% | 4.5 | -10.73 | 16.4 | Confluence mismatch |
| **A3: sweep + BOS combo** | **425.11** | **42.1%** | **69.8%** | **83.3%** | **3.4** | **-8.53** | **49.8** | **Strong combo** |
| **A4: sweep_15m + BOS (RECOMMENDED)** | **448.87** | **41.5%** | **69.6%** | **94.4%** | **3.7** | **-8.52** | **52.7** | **BEST RISK-ADJ** |
| A5: Multi-signal hybrid | 455.12 | 41.7% | 68.6% | 94.4% | 3.8 | -8.52 | 53.4 | Marginal gain |

**Key Finding:** A4 achieves 94.4% weekly win rate with every month positive

**Sheets:**
1. Overview - Headline results
2. Strategy Descriptions - Signal definitions
3. Filter Scan - Filter impact analysis
4. Monthly Breakdown - Month-by-month results
5. Train vs Test - Validation performance
6. Glossary - Term definitions
7. Weekly Summary - Weekly-level metrics
8. V8g Overview - Next version preview
9. V8g Individual TF - Timeframe analysis
10. V8g Zero Contradicting - Contradiction analysis
11. V8g Best Combos - Best combinations
12. V8g TF Pairs - Timeframe pair analysis

---

## Historical Context: Evolution of Strategies

1. **Tempo Batch 2** (Jul-Sep 2025): Initial testing yielded 290 winners from 2,000 variants
2. **Tempo Batch 3** (Jul-Sep 2025): QC testing selected 50 best variants; identified ELITE tier
3. **Tempo Models** (Sep-Dec 2025): Comprehensive condition matrix analysis (4,991 trades)
4. **Inverse Analysis** (Recent): Discovery of counter-trade mechanics & hedging potential
5. **V8f Strategy A** (Feb 2026): Latest rebuild with walk-forward validation

---

## Key Strategic Insights

### Best Performing Signals (Tempo Model 2):
1. **BOS_FVG** - 31.2% WR, 7.24 R/day, 1.037 R/trade (most consistent)
2. **sweep_reversal** - 34.4% WR, 1.84 R/day, 0.393 R/trade
3. **VA_fade** - 17.7% WR, 2.6 R/day, high volume

### Best Time/Day Conditions:
- **Thursday** dominates with 15.33 R/day (27.6% WR)
- Adding **conf≥4** boost improves to 12.56 R/day with 28.7% WR
- **Wednesday** consistently strong: 12.2 R/day

### Sizing Rules (from Matrix):
- BOS_FVG: Base 1.5x-2.0x, increase after 2R profit
- sweep_reversal: Base 1.0x, up to 1.5x in specific sessions
- VA_fade: Base 0.5x (worst signal by R/trade)
- pre_gap_fill: Base 1.25x
- Reverse Martingale on BOS_FVG after wins

### Hedging Strategy (Inverse Analysis):
- Use inverse trades when system directional accuracy <55%
- Regime-switch between original and inverse trades
- Target: maintain 10+ R/day with 28-34% lower drawdown

### Production Ready (V8f Strategy A):
- **Recommended:** A4 (sweep_15m_pro OR BOS_both_pro_conf≥3)
- 448.87 total R over 94 test days
- 94.4% weekly win rate
- Every month positive returns
- Calmar ratio: 52.7

---

## Files Location
All files are stored in: `/sessions/friendly-hopeful-lamport/mnt/Documents/`
- `NQ_Condition_Matrix.xlsx` - Main reference (135 KB)
- `Tempo_Batch2_Winners.xlsx` - Historical winners (47 KB)
- `Tempo_Inverse_Analysis.xlsx` - Hedging strategies (15 KB)
- `Tempo_Batch3_Results.xlsx` - QC results (25 KB)
- `Tempo_V8f_Strategy_A_Results.xlsx` - Latest rebuild (28 KB)

Total: 250 KB of strategic trading analysis

