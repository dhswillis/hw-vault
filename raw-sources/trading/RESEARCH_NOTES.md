# NQ Futures Trading System — Research Notes
## Data: Databento tick-level NQ futures, Oct 2025 - Jan 2026 (88 trading days)

---

## STRATEGY FINDINGS SUMMARY

### WHAT WORKS (Profitable over 88 days)

#### 1. Structural Level Breakouts (v24_levels.py, v24_portfolio_v3.py)
**Entry**: Break of prior bar's high/low on various timeframes.
**Best exits**: Trail 5/3 (activate at +5pts, trail 3 behind) for high-frequency strategies.

| Strategy | Trades/Day | Win Rate | Net Pts | PF | R/Day |
|----------|-----------|----------|---------|-----|-------|
| 1H Breakout (Tr5/3) | 4.6 | 84.8% | +319 | 1.26 | +0.19 |
| 2H Breakout (Tr5/3) | 1.8 | 87.5% | +217 | 1.53 | +0.14 |
| 4H Breakout (Tr5/3) | 0.5 | 91.7% | +125 | 2.52 | +0.13 |
| OR Breakout S10 (Tr5/3) | 1.9 | 70.8% | +17 | 1.03 | +0.02 |

**Best Portfolio (1H+2H+4H+OR)**: 8.9 T/D, 82.7% WR, +677pts ($13,540/NQ), PF 1.30, +0.39R/day, 67% positive days.

**Key notes**:
- Trades are breakouts of the PREVIOUS bar's range on each timeframe
- Only 1 trade at a time per strategy, unlimited re-entries per day
- Opening Range = first 15min of RTH (8:30-8:45 CT), max 2 trades
- ALL fades of these levels are losers — every single one tested

#### 2. 4H Fakeout with 1M Rejection Candle Entry ⭐ BEST STRATEGY
**Concept**: Price fakes out ≥10pts beyond the 4H candle's opening price. A 1M rejection candle forms (wick in fakeout direction > body, wick ≥2pts, body closes in reversal direction). Enter at rejection candle close, S15 fixed stop, timeout at 4H candle end.

| Config | Trades | T/D | WR | Net Pts | PF | R/Day |
|--------|--------|-----|-----|---------|-----|-------|
| REJ_1M S15 (timeout) | 168 | 1.9 | 16.7% | +2,120 | 1.98 | +1.64 |

**Key findings from entry optimization**:
- REJ_1M (1M rejection candle) dramatically outperforms 5M cross baseline
- Sweep+displacement 3M also strong at +1.04R/day
- FVGs barely trigger (7 trades) — useless
- Volume spikes marginal

**What was tested and rejected for this entry**:
- **Rejection candle stop** (stop at candle extreme): 94% stopped out on 1M, -859pts. Stop is too tight — market retests the wick before reversing.
- **Limit at rejection extreme** (buy at low for longs): Fill rate 88-97% but worse than market entry. Best limit config: 5M_S3 at +1.11R/day PF 1.57. Market at close with S15 (+1.41R/day, PF 1.84) significantly outperforms. Better entry price doesn't compensate for timing disadvantage.

#### 3. Daily Open Fakeout with 10M Confirmation
**Concept**: Same as 4H fakeout but on daily RTH open. Price fakes out ≥10pts, then a 10M bar closes back through the daily open. Enter at bar close, S30 fixed stop, timeout at session end.

| Config | Trades | T/D | WR | Net Pts | PF | R/Day |
|--------|--------|-----|-----|---------|-----|-------|
| 10M S30 (timeout) | 62 | 0.7 | 25.8% | +1,431 | 2.02 | +0.77 |

**Only Daily and 4H candle opens work for fakeout**. Tested and confirmed losers/flat: 30M (total loser), 1H (flat — see section below), 2H (marginal).

#### 4. 1H Open Fakeout (v24_1h_fakeout.py) — MARGINAL
**Concept**: Same fakeout setup but on 1H candle opens with lower TF entries.

**Best configs**:

| Signal | FD | Stop | Trades | T/D | Net Pts | PF | R/Day | MDD | Net/MDD |
|--------|-----|------|--------|-----|---------|-----|-------|-----|---------|
| 5M_CROSS | ≥10 | 10 | 420 | 4.8 | +890 | 1.25 | +1.04 | 479 | 1.9x |
| REJ_1M | ≥10 | 20 | 576 | 6.5 | +1,367 | 1.18 | +0.80 | 1,083 | 1.3x |
| REJ_3M | ≥10 | 10 | 558 | 6.3 | +353 | 1.07 | +0.41 | 390 | 0.9x |

**Verdict**: Profitable but far weaker than 4H fakeout. PFs barely above 1.0 (1.07-1.25), massive MDD relative to net, and inconsistent monthly performance (Oct was deeply negative for 5M_CROSS: -463pts). The 1H timeframe has too much noise — fakeouts are less meaningful on shorter candles. **Not recommended for live trading** as an addition to the portfolio unless further validated.

---

### ULTIMATE COMBINED PORTFOLIO (v24_ultimate_portfolio.py)

**Full Portfolio: Breakouts + 4H FK REJ_1M + Daily FK 10M**

| Metric | Value |
|--------|-------|
| **Net PnL** | **+4,227 pts ($84,540/NQ)** |
| **R/Day** | **+2.46R** |
| **Profit Factor** | **1.73** |
| **Trades** | 1,010 (11.5/day) |
| **Win Rate** | 68.2% |
| **Positive Days** | 49% |
| **Max Drawdown** | 391.5 pts ($7,830/NQ) |
| **Net/MDD** | **10.8x** |
| **Annualized** | ~$247,722/NQ |

**Portfolio Breakdown**:

| Component | Net Pts | R/Day | PF | T/D | Contribution |
|-----------|---------|-------|----|-----|-------------|
| BO_1H (Tr5/3) | +319 | +0.19 | 1.26 | 4.6 | 7.5% |
| BO_2H (Tr5/3) | +217 | +0.14 | 1.53 | 1.8 | 5.1% |
| BO_4H (Tr5/3) | +125 | +0.13 | 2.52 | 0.5 | 3.0% |
| BO_OR (S10/Tr5/3) | +17 | +0.02 | 1.03 | 1.9 | 0.4% |
| **FK_4H_REJ1M (S15)** | **+2,120** | **+1.64** | **1.98** | **1.9** | **50.1%** |
| **FK_DAILY_10M (S30)** | **+1,431** | **+0.77** | **2.02** | **0.7** | **33.9%** |

**Monthly Performance (Full Portfolio)**:

| Month | BO_1H | BO_2H | BO_4H | BO_OR | FK_4H | FK_Daily | TOTAL | $/NQ |
|-------|-------|-------|-------|-------|-------|----------|-------|------|
| Oct 25 | -6 | +61 | +5 | -59 | +1,587 | +1,046 | +2,633 | +52,665 |
| Nov 25 | +162 | +59 | +47 | +49 | +170 | +226 | +711 | +14,225 |
| Dec 25 | +47 | +5 | +14 | +47 | +229 | +72 | +414 | +8,270 |
| Jan 26 | +116 | +92 | +60 | -19 | +134 | +87 | +469 | +9,380 |

**Equity Curve Stats**:
- Avg win day: +153.2pts ($3,064/NQ)
- Avg loss day: -50.2pts ($-1,003/NQ)
- Best day: +1,675.5pts ($33,510/NQ)
- Worst day: -133.5pts ($-2,670/NQ)
- Max consecutive losses: 6 days
- Max consecutive wins: 5 days

**Correlation Matrix (Daily PnL)**:
- Fakeouts vs Breakouts: 0.05-0.19 (very low — genuine diversification)
- FK_4H vs FK_Daily: 0.43 (moderate — same concept, different TF)
- Breakouts inter-correlated: 0.36-0.51

---

### WHAT DOESN'T WORK (Tested and confirmed losers)

#### Fades of structural levels
- Prior Day H/L fades: LOSER
- Asia H/L fades: LOSER
- London H/L fades: LOSER
- Opening Range fades: LOSER
- 1H/4H bar level fades: ALL LOSERS

#### ICT Concepts (v24_ict2.py through v24_disp_fvg.py)
- **FVG entries**: Losers on all timeframes (1M, 5M, 15M), all killzones
- **IFVG entries**: Losers
- **Order Blocks**: Losers (also had implementation bugs)
- **Breaker Blocks**: Losers (also had implementation bugs)
- **Sweep + FVG**: Losers
- **Displacement FVG** (body/ATR filter): Less bad at D≥2.0 but still negative
- **Unicorn**: Marginally profitable only on 1M FULL session (+149pts, PF 1.21)

#### High-frequency strategies (>50 trades/day)
- ANY strategy at 1M bar frequency: destroyed by transaction costs
- 30M breakouts: marginal/negative
- 15M breakouts: negative

#### Opening Range strategies (v24_open_v3.py)
- Both fade and breakout at high frequency (228-446 T/D): ALL losers

#### Fakeout on smaller candle opens
- 30M open fakeout: total loser
- 1H open fakeout: marginal (see Section 4 above — weak PF, huge MDD)
- 2H open fakeout: marginal

---

## EXIT PARAMETER RESEARCH (v24_exit_grid.py)

### Key finding: Trail 5/3 chokes off profits
MFE analysis of structural breakout trades shows massive unrealized potential:
- 50th percentile MFE: +77.9pts
- 75th percentile: +162.8pts
- 90th percentile: +255.9pts

Trail 5/3 captures only ~2-5pts per winner when trades have 50-250pt potential.

### BE + Wide Trail
Moving to breakeven at +5-15pts, then trailing 30pts behind with 5-10pt trail distance captures 2-4x more R. But in live portfolio testing, the improvement was more modest than grid search predicted (grid overestimates due to evaluating exits in isolation).

### Best exit per strategy type:
- **High-frequency (1H, 4.5 T/D)**: Trail 5/3 works best — many small wins compound
- **Low-frequency (2H, 4H, 0.5-0.9 T/D)**: BE+trail or wider trail better
- **4H Fakeout**: Timeout (hold to end of 4H bar) captures the most
- **Daily Fakeout**: Timeout (hold to session end) with S30 stop

---

## FAKEOUT ENTRY OPTIMIZATION RESEARCH

### Confirmation Bar Timeframe (v24_fakeout_entry_tf2.py)
Tested TICK, 1M, 2M, 3M, 5M, 10M, 15M confirmation bars:
- **4H open**: 5M with S15 is sweet spot (+1.16R/day, PF 2.09). TICK highest R/day (1.28) but only 14 trades.
- **Daily open**: 10M with S30 best (+0.91R/day, PF 2.19)
- **Lower TFs (1M, 2M) actually hurt** — too much noise on standard cross confirmation

### ICT Entry Signals (v24_fakeout_ict_entry.py)
Tested during 4H fakeout: sweep+displacement, FVG, rejection candle, volume spike on 1M/3M/5M:
- **REJ_1M S15**: Winner at +1.41R/day, PF 1.84, 168 trades
- **SWEEP_DISP_3M S15**: +1.04R/day — solid second option
- **FVGs**: Barely triggered (7 trades) — useless
- **Volume spikes**: Marginal

### Rejection Candle Definition
- 1M bar where wick in fakeout direction > body
- Body closes in reversal direction (green for longs, red for shorts)
- Wick must be ≥2pts
- Enter at market at bar close + 0.25pt slippage

### Why Rejection Candle Stops Don't Work (v24_rej_candle_stop.py)
- 1M rejection candle stop: 94% stopped out, -859pts. The wick is too tight.
- 3M showed +1.01R/day but negative net PnL (-233pts) — misleading R from variable stop sizes
- Market retests the wick before reversing in most cases

### Why Limit Orders at Extreme Underperform (v24_rej_limit.py)
- Fill rate was high (88-97%)
- Best: 5M_S3 at +1.11R/day PF 1.57
- But market at close with S15: +1.41R/day, PF 1.84
- Better entry price doesn't compensate for worse timing

---

## TRANSACTION COSTS
- Slippage: 0.25 points per trade (0 for limit orders)
- Commission: 0.50 points per trade
- Total: 0.75 pts/trade ($15/NQ, $1.50/MNQ)

## SESSION TIMES (UTC)
- Asia: 0:00-8:00 (6PM-2AM CT)
- London: 8:00-14:30 (2AM-8:30AM CT)
- RTH: 14:30-21:00 (8:30AM-3PM CT)
- OR build: 14:30-14:45 (8:30-8:45AM CT)
- Flatten: 20:55 (2:55PM CT)
- Entry cutoff: 20:45 (2:45PM CT)

## DATA
- Location: ~/Documents/trading-system/databento/
- Format: glbx-mdp3-YYYYMMDD.trades.dbn.zst
- Front-month symbol: no dash in symbol name
- Available range: ~Feb 2025 - Feb 2026

---

## SCRIPTS INDEX
| File | Purpose | Key Results |
|------|---------|-------------|
| v24_levels.py | Multi-level breakout scanner | Found 1H/4H/OR breakouts profitable |
| v24_portfolio.py | Original portfolio (1H+4H+OR) | +422pts, +0.24R/day |
| v24_exit_grid.py | Exit parameter grid search | MFE analysis, BE+trail discovery |
| v24_portfolio_v2.py | Portfolio with BE+trail exits | ~Same as baseline surprisingly |
| v24_deep_mine.py | More TFs, TOD filters, re-entries | Found 2H breakout profitable |
| v24_portfolio_v3.py | Best portfolio (1H+2H+4H+OR) | +703pts, +0.41R/day |
| v24_4h_expansion.py | 4H fakeout strategy (5M cross) | +395-1078pts, +0.5-1.2R/day |
| v24_combined_final.py | Breakouts + 4H FK (5M cross exits) | +805-2077pts, +0.47-1.21R/day |
| v24_fakeout_opens.py | Multi-TF fakeout test (Daily/1H/2H/30M/4H) | Only Daily and 4H work |
| v24_fakeout_entry_tf2.py | Entry TF optimization (TICK-15M) | 5M sweet spot for 4H, 10M for Daily |
| v24_fakeout_ict_entry.py | ICT entry signals during fakeout | REJ_1M is the winner (+1.41R/day) |
| v24_rej_candle_stop.py | Stop at rejection candle extreme | 94% stopped out — terrible |
| v24_rej_limit.py | Limit order at rejection extreme | Underperforms market entry |
| v24_ultimate_portfolio.py | Full combined portfolio | **+4,227pts, +2.46R/day, PF 1.73** |
| v24_1h_fakeout.py | 1H open fakeout with lower TF entries | Marginal: +890pts best, PF 1.25 |
| v24_ict2.py | ICT concepts (FVG/IFVG/Unicorn) | All losers except 1M Unicorn |
| v24_disp_fvg.py | Displacement FVG test | Confirmed FVGs are losers |
| v24_scanner.py | Broad strategy scanner | Initial discovery phase |
| v24_validate2.py | Extended validation | Confirmed PD/OR breakouts |

---

## NEXT STEPS
1. Out-of-sample validation (Feb 2025 - Sep 2025 or Feb 2026)
2. NinjaTrader strategy implementation with proven parameters
3. Position sizing optimization
4. Walk-forward optimization
5. Consider adding 1H fakeout only if OOS validates

---

## KEY LESSONS LEARNED
1. **Transaction costs kill high-frequency edges** — anything >10 T/D with small R needs very high WR
2. **ICT concepts are mostly losers** in systematic backtesting on NQ — FVGs, order blocks, breaker blocks all negative
3. **Fakeout depth is the critical filter** — below 10pts, no edge exists
4. **Rejection candles work as entry signals** but NOT as stop placement
5. **Market entries beat limit entries** for fakeout trades — timing > price
6. **Grid search overestimates exit improvements** — always validate in full portfolio context
7. **Only large candle opens (4H, Daily) produce reliable fakeout edges** — 30M/1H/2H too noisy
8. **Low correlation between breakout and fakeout strategies** provides genuine diversification

*Last updated: March 5, 2026*
