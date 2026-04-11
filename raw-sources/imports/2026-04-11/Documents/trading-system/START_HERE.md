# START HERE — Tempo Trading System

> **Every Claude session (Cowork, Claude Code, or any future tool) must read this file first.**
> This is the single source of truth for the entire project.
> Last updated: Mar 6, 2026

---

## Who You Are Working With

**Harrison** (@prop_profitable)
- Trader, not a developer. Speaks casually. Expects practical output, not lectures.
- Building an autonomous NQ futures trading intelligence platform called **Tempo**
- Email: dhswillis@gmail.com
- GitHub: dhswillis

---

## The 30-Second Summary

Tempo is a **portfolio of non-correlated NQ futures signal models** targeting **10R+ per day combined**. Each model has its own entry logic, execution assumptions, and edge profile. The models are intentionally different — they generate different trade populations so losses don't cluster.

**Model 1 (QC)**: Sweep at key levels → IFVG confirmation → market order entry. Multi-timeframe, liquidity-target exits. Runs on QuantConnect.
**Model 2 (Local backtester)**: BOS/sweep/VA/PGF signals → FVG midpoint limit entry. Confluence scoring, session timing. Runs on Databento tick data.

Model 2 is validated: **+11.31R/day over 258 days** (full year, 72% profitable days, 13/13 months). Model 2 ALONE exceeds the 10R target. Model 1 batch optimization in progress for additional non-correlated R.

**V8 Mining (Feb 17, 2026)**: Walk-forward validated Strategy A (65.3% daily WR, +4.31R/day, 94.4% weekly WR) and Strategy B (+11.05R/day, 68.4% daily WR). Combined portfolio: **+12.07R/day, 94.4% weekly WR, Calmar 31.4**. Full synthesis in `context-engine/V8_MINING_SYNTHESIS.md`.

We are currently in the **backtesting + optimization** phase. Not live trading yet.

---

## Reference Documents (Read These Next)

| Document | Location | What It Contains |
|----------|----------|-----------------|
| **CLAUDE_BOOTSTRAP.md** | Repo root | Credentials, project structure, technical setup, integration details |
| **CONTEXT.md** | Repo root | Strategy mechanics, signal types, execution modeling parameters, batch results |
| **Tempo_Context_Engine_Spec.docx** | `~/Documents/` | Full engineering spec for the context engine: 46 features, 9 checkpoints, volume profile engine, footprint analysis, self-improving loop, build phases |
| **Tempo_Batch2_Winners.xlsx** | `~/Documents/` | Batch 2 results (290 winners, pre-cost-modeling) |
| **main_v4.py** | `batch-runner/` | The QC strategy — read Initialize() for execution parameters |

---

## Current State of the System (Keep This Updated!)

### What's Running
- **Batch 3** (Model 1 — QC) on VPS with cost modeling (slippage, fees, tick-through, BE offset)
  - VPS: `root@172.93.181.209`
  - Log: `tail -20 ~/batch3_costmodel.log`
  - Results: `~/trading-system/batch-runner/results/batch3_costmodel/`
  - 2000 variants, Stage A (Jul-Sep 2025)
- **Auto-iterate + strategy matrix** (Model 2 — backtester) running in background on local data

### What's Built and Working

**Model 1 — QC Sweep + IFVG:**
- QC strategy (main_v4.py) with realistic execution costs
- Two-stage batch runner (Stage A screen → Stage B confirm)
- 2000 variant generator
- GitHub repo with auto-deploy

**Model 2 — Local Backtester (AUDITED — V10r/V10s/V10t):**
- **micro_structure.py**: Core pattern engine — FVG, BOS, sweeps, engulfing, volume spikes, fib
- **intraday_scanner.py**: 13 signal strategies, confluence scoring, session management. **FIXED**: look-ahead in extended_5m and target swing computation
- **⚠️ V7/V8 results INVALIDATED** — V10o audit discovered multi-TF alignment used forming bar closes (look-ahead bias). ALL high WR results (>55%) were from future data, not real edge.
- **V10t clean results (post-audit, walk-forward validated):**
  - **6 validated signals**: engulfing_key_level, BOS_FVG, VA_fade, pre_gap_fill, ORB_breakout, vol_spike_FVG
  - **Combined portfolio**: 2,423 trades, +1.44 R/day, Calmar 17.4, 56.4% DWR, 73.1% WWR
  - **Walk-forward**: IS +0.162 → OOS +0.243, ALL 4 quarters positive OOS
  - **7 dead signals**: london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion
  - **Commission note**: +1.44 R/day is approximately breakeven after commissions for 1 NQ contract
- **V10_STRATEGY_BIBLE.md**: Complete documentation of validated signals, dead ends, and methodology

**Shared Infrastructure:**
- **Context Engine V2 — FULLY BUILT** (see below)
- **Databento tick data** — 312 trading days (Feb 2025 - Feb 2026), processed into feature matrix
- **Bayesian session classifier** — 68.6% test accuracy on 5-class (TREND 100%, NEWS_SHOCK 100%)
- **Strategy bridge** — Per-signal gate matrix. Maps session type → signal permissions
- **Markov transition model** — Session type transitions with streak effects + day-of-week bias
- **Variant analyzer** — Context-tagged batch variant analyzer
- **Live tick processor** — Replays historical days through full pipeline
- OpenClaw/Telegram bot (configured, skill incomplete)
- Claude Code installed on Mac

### Context Engine V2 — Module Map
```
context-engine/
├── context_engine.py          ← V1 orchestrator (VIX, calendar, session, day type)
├── context_engine_v2.py       ← V2: 5-channel output (session probs, opp score, vol, SMT, confidence)
├── strategy_bridge.py         ← Maps V2 output → StrategyConfig overrides
├── live_processor.py          ← Real-time tick processing + checkpoint scheduling
├── run_v2.py                  ← CLI: replay / backtest / demo modes
├── backtest_report.py         ← Full 255-day performance report
├── process_full_year.py       ← Batch processor for Databento .dbn.zst files
├── train_classifier.py        ← V1 training (52.9% accuracy)
├── train_classifier_v2.py     ← V2 training with derived features (68.6% accuracy)
├── features/
│   ├── volume_profile.py      ← POC, Value Area, HVN/LVN from tick data
│   ├── footprint.py           ← Delta, imbalance stacks, absorption detection
│   ├── smt.py                 ← NQ vs ES swing divergence (SMT)
│   ├── bayesian_classifier.py ← Naive Bayes 5-class classifier
│   ├── vol_regime.py          ← AlgoFlows severity score + session×vol matrix
│   ├── markov_model.py        ← Session type transition model (Markov chain + streak + DOW)
│   └── session_labeler.py     ← Auto-labels days for training (rules-based)
├── variant_analyzer.py        ← Context-tagged batch variant analyzer (condition-specific winners)
├── models/
│   ├── bayesian_classifier_v1.json
│   ├── bayesian_classifier_v2.json
│   ├── markov_model.json      ← Trained Markov transition matrix
│   ├── V8a_overnight_mine.json ← MTF alignment, confluence, B/E, preceding
│   ├── V8b_new_triggers.json   ← Delta, imbalance, VP, entry delta composites
│   ├── V8c_gap_oi_analysis.json ← Gap/settlement/OI analysis
│   ├── V8d_strategy_builder.json ← Walk-forward Strategy A/B/Combined
│   └── V8e_session_day_deep.json ← Signal×session, DOW, preceding, momentum
├── run_v8a_overnight_mine.py  ← Multi-TF confluence mining
├── run_v8b_new_triggers.py    ← Footprint/VP new trigger mining
├── run_v8c_gap_oi_analysis.py ← Gap/OI/settlement analysis
├── run_v8d_strategy_builder.py ← Walk-forward strategy builder
├── run_v8e_session_day_deep.py ← Session/day/preceding deep dive
├── V8_MINING_SYNTHESIS.md     ← Full V8 findings synthesis
└── providers/                 ← V1 data providers (VIX, calendar, session, day type)
```

### V7r Deep Mining Results (Feb 15-16, 2026) — 312 days, all phases complete
**Session Analysis:**
- NY open 30min: 53.2% continuation rate — slight edge following opening direction
- London opens: ANTI-continuation (48.3% cont = reversal is more likely)
- Asia does NOT predict London/NY direction (41.8% cont = worse than coin flip)
- NY close: 49.2% bearish bias in last hour
- Overlap session: 223.8pt AAR, 14.1 BOS/day, 5.4 sweeps/day — most fertile

**Candle Pattern Edges (at trade entry):**
- **Doji at entry: 44.2% WR, 1.757 avg R** (vs baseline 31.6% WR, 1.037 avg R)
- **Pin bar bull at entry: 43.4% WR, 1.727 avg R** — strong confirmation
- **Shooting star at entry: 45.6% WR, 1.433 avg R** — strong confirmation
- **Hammer at entry: 41.1% WR, 1.106 avg R** — moderate confirmation
- Marubozu/strong close patterns HURT win rate at entry — avoid

**FVG Fill Quality:**
- Doji at fill: 40.8% WR, 1.483 avg R (BEST)
- Strong close no wick: 30.8% WR, 0.259 avg R (WORST — counterintuitive!)
- Thesis: Doji = absorption / indecision at FVG level = trapped traders = reversal works

**Composite GOLD (BOS_FVG + confirmation):**
- **BOS_FVG + strong_close: 83.9% WR, 6.15 avg R, Sharpe 0.912** (n=31, needs more data)
- BOS_FVG + pin_bar_bear: 60% WR, 2.24 avg R, Sharpe 0.565 (n=40)
- BOS_FVG + shooting_star: 62.7% WR, 2.98 avg R, Sharpe 0.554 (n=51)
- BOS_FVG + doji: 55.9% WR, 3.06 avg R, Sharpe 0.550 (n=111)

**Stop Limit Breakouts (liquidation catching):**
- 5pt offset, 2R target: 48.7% WR, 0.45 avg R, Sharpe 0.304, 10,431 trades
- 5pt offset scales well: 3R→39.8% WR, 5R→30.6% WR, 10R→23.4% WR, all profitable
- This is a NEW signal type: enter on stop limit after liquidity sweep

**AAR by Session (stop/target sizing reference):**
- Asia: 144.9pt mean, 116 median
- NY IB: 155.9pt mean, 123 median
- NY AM: 141.2pt mean, 114 median
- Overlap: 223.8pt mean, 198 median
- AAR sizing: 10% stop / 20% target best Sharpe (0.228), 54.2% WR

### V7r-b Sub-Minute Candle Confirmation (Feb 16, 2026)
**15S candle at BOS_FVG entry:**
- BOS_FVG + strong 15S confirm: 39.1% WR, 1.816 avg R (n=87) — BEST combo
- Counterintuitive: "neutral" 15S candle outperforms confirmation overall (34.1% vs 31.1% WR)
- LOW volume at 15S entry beats HIGH (0.659 vs 0.435 avg R)
- Waiting 3×15S for confirmation: WR 34.6% vs 32.4% immediate, avgR 0.696 vs 0.585 (filters 19%)
- 15S shooting star at entry: 35.4% WR, 0.958 avg R — best 15S pattern

### V7s Raw Stop Limit Deep Dive (Feb 16, 2026) — NEGATIVE
- Raw `find_liquidity_sweeps()` stop limits are NET NEGATIVE
- The V7r stop limit edge came from SCANNER-FILTERED sweeps (confluence ≥ 2)
- Raw sweeps are too noisy — scanner's confluence filtering IS the edge
- Only nuggets: tiny wicks (0-2pt) near breakeven, 03:00 UTC positive, high vol at sweep slightly better

### V7t Composite Confirmation Gates (Feb 16, 2026) — Walk-Forward Validated
**Gates tested on BOS_FVG trades, walk-forward split:**
- **gate_15s_strong**: 39.1% WR, 1.816 avg R, Sharpe 0.282, 5% filter. WF: STRONG (2.249→1.430)
- **gate_triple** (5M good + 15S confirm + low vol): 35.7% WR, 1.529 avg R, 5.6% filter. WF: STRONG
- **gate_5m_good_AND_no_contra**: 33.0% WR, 1.369 avg R, Sharpe 0.258, 32.9% filter. WF: STRONG ← **BEST PRACTICAL GATE**
- gate_low_vol_15s: 33.0% WR, 1.195 avg R, 60.7% filter. WF: STRONG ← lightest gate
- Baseline BOS_FVG: 31.6% WR, 1.038 avg R

### V7u Scanner-Filtered Stop Limit (Feb 16, 2026)
- Scanner sweep_reversal market baseline: 36.4% WR, 0.393 avg R, Sharpe 0.169
- Best stop limit (off3_r7.5_stop15): Sharpe 0.171 — barely beats market order
- **Stop limits don't add edge on scanner sweeps** — market entry already well-timed
- **15S contradiction at sweep OUTPERFORMS confirmation** (trapped traders = stronger reversal)
- Best sweep sessions: power_hour (41.3% WR) and overlap (32.3% WR)
- Walk-forward: ALL top 10 configs stable (both halves positive)

### V7v Streak-Based Sizing (Feb 16, 2026) — MAJOR FINDING
**VA_fade is EXTREMELY streak-dependent:**
- After a WIN: 60%+ WR, 1.6 avg R (hot hand is REAL for VA_fade)
- After 1 loss: drops to 20% WR, -0.27 avg R
- **"Aggressive" sizing (scale up after 2W, reduce after 3L): VA_fade goes 601R → 1319.8R (2.2x!). Calmar 11→41**
- **VA_fade pause_after_3L: 826.8R, Calmar 43.9 — simplest improvement**

**BOS_FVG is streak-resistant:**
- Flat vs aggressive similar. reduce_after_3L has best Calmar (70.6)
- After 2 consecutive losses: avg R INCREASES to 1.405 (mean reversion)

**sweep_reversal:**
- pause_after_3L: Calmar 20.6 (up from 13.2 flat)
- scale_up_after_2W: +10% total R

**Portfolio-level:** WR drops 43.9% → 19.6% over 7 loss streak. Mean reversion at 5+ losses.
**AM→PM correlation:** Zero (0.052). Morning results don't predict afternoon.

### V7w Combined Optimized Portfolio (Feb 16, 2026) — CALMAR DOUBLED
**Applied all V7r-V7v findings together:**
- **Baseline**: 2917.9R, 9.35R/day, MaxDD 47.4R, Calmar 61.56
- **Optimized**: 2222.5R, 7.12R/day, MaxDD 17.9R, **Calmar 123.99**
- **Max DD cut 62%** (47.4R → 17.9R) — much more investor-friendly
- Calmar doubled (+101%), worst day -13.7R (was -24.2R)
- Walk-forward: 7.73R/day train → 6.52R/day validate (STABLE)
- BOS_FVG: R/trade 1.037 → 1.370 (+32%)
- VA_fade: 601R → 1016.8R (+69%)
- Investor metrics (2 NQ, $400/R): est $718K/yr, $21K capital, 3341% return

### V7x Session-Specific Signal Allocation (Feb 16, 2026)
**VA_fade session filter (walk-forward confirmed):**
- **DISABLE VA_fade in ny_am, power_hour, silver_bullet** — all negative, saves +57R
- VA_fade GOLD sessions: overlap (1.181 avg R), news_window (1.281), london_am (0.636)
- BOS_FVG is GOLD in EVERY session — backbone signal
- sweep_reversal GOLD: ny_am (0.847), ib_first (0.652)

**Walk-forward stable gold pairs:**
- BOS_FVG_ny_am: 1.399 avg R (1.443→1.351 STRONG)
- BOS_FVG_ib_first: 1.378 avg R (1.592→1.195 STRONG)
- VA_fade_overlap: 1.181 avg R (1.807→0.668)
- sweep_reversal_ny_am: 0.847 avg R (0.684→1.130 STRONG)

### V7y Full Optimized Backtest (Feb 16, 2026) — PRODUCTION CONFIG
**All V7r-V7x findings combined (BOS gate + VA sizing + VA session filter + sweep pause):**
- **Baseline: 2917.9R, 9.35R/day, MaxDD 47.4R, Calmar 61.6**
- **OPTIMIZED: 2261.2R, 7.25R/day, MaxDD 16.4R, Calmar 137.8**
- MaxDD cut 65%: 47.4R → 16.4R
- Calmar up 124%: 61.6 → 137.8
- Worst day: -24.2R → -13.7R (43% better)
- Walk-forward: H1 DD 16.4R, H2 DD 16.0R — ROCK STABLE
- VA_fade: 601R → 1055.5R (+76%)
- Filters applied: 793 BOS 5M pattern, 379 BOS 15S contra, 334 sweep pause, 215 VA session

### V7z Advanced Filters (Feb 16, 2026) — WAIT-FOR-WIN IS REAL
**VA_fade wait-for-first-win:**
- **wait_win_then_only**: 50.9% WR, 1.268 avg R — 3.4x better per trade. 76.3% profitable days
- **aggressive_after_win**: Calmar 46.2 (vs 12.1 flat), MaxDD 25.9R — best risk-adjusted
- Thesis confirmed: VA_fade streaks cluster. First win signals "regime is on."
- ONLY trade VA_fade after it wins once that day. This is the production rule.

**BOS_FVG candle × streak interaction:**
- Gate only: MaxDD 27.8→20.6, Calmar 56.7
- Streak only: best Calmar 75.1
- Gate AND streak: lowest MaxDD 17.4R, Calmar 62.8

**Fading large wick sweeps: V7z had a BUG. V7z-fix corrects it.**
- V7z bug: same-candle stop-out (59.7% of huge wick fills were false stops)
- V7z also used fixed 10pt stop in 15+ pt wick environment — way too tight

### V7z-fix Large Wick Mechanics (Feb 16, 2026) — FOLLOW WORKS, FADE DEAD
**The V7z bug:** outcome checking started at the fill candle itself. On 1M bars, the fill
candle wicks through both entry AND stop → instant false stop-out. For huge 15+ pt wicks,
59.7% of trades were killed by this bug. Fixing it recovered 30.9% of trades.

**FOLLOW with dynamic stops is very profitable:**
- **Best config: stop=0.75×wick, 3R target, 0 offset → 48.0% WR, 0.899 avg R, 2154R total**
- stop=1.0×wick, 1.5R target → 69.0% WR, 0.723 avg R (highest WR)
- stop=1.0×wick, 2R target → 59.4% WR, 0.772 avg R (good balance)
- Key insight: stop MUST scale with wick size. Fixed 10pt stop = catastrophe.

**FADE is genuinely dead.** Even with fixed mechanics, dynamic stops, every config:
- Best fade config: 24.6% WR, -0.394 avg R (2.0× wick stop, 1.5R target, 0.5× offset)
- NEVER fade a sweep. The wick IS the rejection.

**15S granularity made it WORSE.** More noise, more false fills. 1M is the right timeframe.

**Waiting degrades performance.** Edge is strongest at wait=0. By wait=3min, follow goes negative.

**News doesn't matter.** Large wicks in news vs outside: 49.3% vs 51.5% WR — similar.

**Market order at sweep close (Study 6):**
- Large 10-15pt wicks: 10pt stop + 2R target → 57.1% WR, 0.710 avg R
- Huge 15+ pt wicks: need 15pt stop minimum → 50.3% WR, 0.508 avg R
- Medium 5-10pt wicks: 8pt stop + 2R target → 56.2% WR, 0.682 avg R

**Session distribution of large wicks (10+ pt):**
- ib_first: 19.2% — heaviest concentration
- news_window: 16.0%
- ny_am: 10.2%, power_hour: 8.8%, silver_bullet: 8.6%
- NOT clustered in one session — spread across the day

### V7aa Candle Pattern Effects at Entry (Feb 16, 2026) — SKIP CONTRA = +118% CALMAR
**The single biggest finding: SKIP trades where the 5M candle CONTRADICTS trade direction.**

**Pattern alignment is MASSIVE:**
- Confirming (bullish candle + long, or bearish + short):
  - BOS_FVG: 36.8% WR, 1.604 avg R | VA_fade: 40.4% WR, 0.852 avg R | sweep: 54.5% WR, 1.199 avg R
- Contradicting (bearish candle + long, or bullish + short):
  - BOS_FVG: 23.4% WR, 0.213 avg R | VA_fade: 18.7% WR, -0.338 avg R | sweep: 16.6% WR, -0.505 avg R

**Best sizing strategy: SKIP_CONTRA (don't take contradicting trades):**
- Portfolio Calmar: 54.6 → **119.3** (+118%). MaxDD 54.5 → 28.9R (-47%).
- sweep_reversal: Calmar 13.2 → **63.2** (4.8×!), WR 36.4% → 53.2%
- VA_fade: Calmar 12.1 → **30.4** (2.5×), MaxDD cut 47%
- BOS_FVG: Calmar 65.1 → **94.0**, WR 31.6% → 38.8%

**BOOST_AND_REDUCE (1.25x confirm, 0.5x contra) is the less aggressive option:**
- Portfolio Calmar 54.6 → 89.3 (+64%). Total R jumps 2975 → 4181 (+41%).

**5M single-candle patterns (best to worst, ALL signals):**
- GOLD: hanging_man (1.396 avg R), strong_close_bear (1.329), marubozu_bull (1.233), strong_close_bull (1.141)
- TOXIC: shooting_star (-0.295), long_upper_shadow (-0.193), pin_bar_bear (-0.006)

**1M candle at entry (counterintuitive):**
- Indecision candles (doji, pin bar) OUTPERFORM momentum candles (marubozu) at 1M
- Best: doji_gravestone (2.461 avg R), hanging_man (2.155), inverted_hammer (1.813)
- Worst: marubozu_bull (-0.210 avg R) — momentum at entry = chasing

**Two-candle patterns (5M):**
- Engulfing (bullish 0.786, bearish 0.655 avg R) = positive
- Bearish harami (-0.175 avg R) = negative. Tweezer top = weak.

**Three-candle patterns (5M):**
- All positive: three_black_crows (0.950), three_white_soldiers (0.866), evening_star (0.855), morning_star (0.768)

**BOS_FVG best entry patterns:** strong_close_bear (2.123 avg R, n=190, Calmar 43.4), hanging_man (1.823), doji_dragonfly (1.714)
**sweep_reversal best:** morning_star (1.785 avg R, 59.2% WR, n=49), evening_star (1.234, 53.6% WR)

### V8 Overnight Mining Results (Feb 17, 2026) — 312 days, 5 studies, ALL walk-forward validated

**Full synthesis: `context-engine/V8_MINING_SYNTHESIS.md`**

**Walk-Forward OOS Results (94 test days):**
| Strategy | Total R | R/Day | Daily WR | Weekly WR | Max DD | Calmar |
|---|---|---|---|---|---|---|
| Strategy A (High Conf) | +323.0 | +4.31 | **65.3%** | **94.4%** | -13.9R | 23.3 |
| Strategy B (Max R) | +839.7 | +11.05 | 68.4% | 94.4% | -27.6R | 30.4 |
| Combined | +917.2 | +12.07 | 68.4% | **94.4%** | -29.2R | 31.4 |

**V8a — Multi-TF Confluence:**
- sweep_reversal_both_pro: 66.1% WR, +2.07R avg (n=59) — best trade-level edge found
- BOS_FVG_both_pro: +1.39R avg (n=724) — workhorse
- Confluence scales perfectly: conf_7 = +3.63R avg (69.2% WR), conf_5 = +1.65R
- B/E confirmed dead: no_be = +2287R, be_1.5r = +1885R for BOS_FVG

**V8b — New Entry Triggers (Footprint/Volume Profile):**
- BOS_FVG + confirming delta: +2.26R avg
- BOS_FVG + strong imbalance stacks: +2.25 to +2.73R avg
- BOS_FVG + strong_sell_delta at entry: +2.24R avg (n=106)
- 1+ new trigger present: BOS_FVG avg R doubles (0.56 → 1.26), Calmar 10→75

**V8c — Gap/OI/Settlement:**
- BOS_FVG thrives on huge gaps: +1187R total (n=925)
- After extreme vol: BOS_FVG +4.82R per trade (n=34)
- Settlement zone: No strong filter needed — BOS_FVG robust across all zones

**V8e — Session/Day/Preceding:**
- BOS_FVG best in ny_am (+1.42R) and ib_first (+1.38R)
- VA_fade: Only overlap (+1.18R) and london_am (+0.64R) profitable — kill all other sessions
- After big win day → next day +9.96R avg (68.3% WR)
- London strong win → NY avg +4.84R (62.7% WR)

**V8f — Strategy A Rebuild (55%+ Trade WR Target):**
- V8d's "Strategy A" only had 28.4% trade WR — rebuilt with systematic filter scan
- Only sweep_reversal can achieve 55%+ trade WR; BOS_FVG maxes at ~44% (limit order mechanics)
- Best pure WR: sweep|both_pro → **84.6% trade WR OOS** but only 0.14 trades/day
- Best practical: A4 (sweep_15m_pro OR BOS_both_pro_conf>=3) → +449R in 94 days, 94.4% weekly WR, Calmar 52.7
- IMPORTANT: Live scanner confluence ≠ V8a pre-computed conf_score (different semantics)

**V8g — Multi-TF Candle Direction Alignment (1M/3M/5M/10M/15M/1H):**
- At each entry, classifies candle direction at 6 timeframes as confirming/contradicting/neutral
- **Monotonic relationship**: 0/6 confirming = 5.7% WR → 6/6 confirming = 40.2% WR
- sweep_reversal + 3M/5M/15M/1H all confirming = **79.8% WR** (n=109, +2.72R avg)
- BOS_FVG + 0 contradicting = **77.9% WR** (n=122, +4.86R avg)
- BOS_FVG >=3 confirming + 0 contradicting = **80.3% WR** (n=117, +5.10R avg)
- **1M is NOT useful** (drop it). 3M–1H all valuable filters.
- Full analysis in spreadsheet: `~/Documents/Tempo_V8f_Strategy_A_Results.xlsx` (V8g tabs)

**Scripts produced:**
- `run_v8a_overnight_mine.py` — MTF alignment, confluence scoring, B/E, preceding conditions
- `run_v8b_new_triggers.py` — Delta divergence, imbalance stacks, VP context, entry delta
- `run_v8c_gap_oi_analysis.py` — Gap/settlement/OI from statistics data
- `run_v8d_strategy_builder.py` — Walk-forward strategy builder (A/B/Combined)
- `run_v8e_session_day_deep.py` — Session/day/preceding deep analysis
- `run_v8f_strategy_a_rebuild.py` — Strategy A rebuild with systematic filter scan + walk-forward
- `run_v8g_multi_tf_candle_alignment.py` — 6-TF candle direction alignment analysis

### V9 Mining Results (Feb 18, 2026) — Production System Specification Complete
- **V9a**: Premium/discount zone filter: optimal zone = +16pp WR, 20x Calmar vs suboptimal
- **V9b**: Stress test: system survives 3pt extra slippage (Cal 108), 50% missed signals (Cal 111), extreme worst case (Cal 57)
- **V9c**: Stacked filters: V8g>=5 + optimal zone = **87.9% WR**, V8g>=4 + 0contra + optimal zone = **87.1% WR**
- **V9d**: Drawdown recovery: V8g>=4 recovers ALL drawdowns in ≤5 trades. After 3 losses: 71.4% WR, +6.25 avgR
- **V9e**: Target optimization: **LET WINNERS RUN is optimal**. No partials, no trailing, no caps.
- **V9f**: Production spec: **Tier 1+2 = 70% WR, +5.08 R/day, Calmar 235.7, $1M+/yr at 1% risk on $100K**

### V13 Opening Range Failure Mining (Feb 22, 2026) — PRODUCTION MODEL FOUND

**Problem solved:** V10-V12 all failed because 1M FVG-based stops are 2-4pts, making 1pt commission = 25-50% of risk. V13c solved this by moving to 5M timeframe entries with wider natural stops (6-15pts), reducing commission to 5-17% of risk.

**V13c-V13g progression:**
- V13c: 5 concepts tested (OR_FAIL, VWAP_REV, SWEEP_5M, ENG_5M, TREND_PULL). Only OR_FAIL survived full year.
- V13d: Deep OR_FAIL analysis. OR_15 (15-min opening range) + no FVG requirement = 212 trades, 77.8% WR, +25.2R.
- V13e: Direction analysis. **SHORT-ONLY = 84.4% WR, +29.3R, 10.4 Calmar. LONGS LOSE MONEY (-4.6R).**
- V13f: Trail optimization. **Trail 0.3R trigger / 30% trail = 90.8% WR, +51.8R, 21.5 Calmar (short-only).** Long rescue: delta filter (OR delta < 0) makes longs +4.0R.
- V13g: Production spec. Combined short + delta-filtered longs = **89.0% WR, +65.6R, +0.26 R/day, 22.9 Calmar**. Walkforward: H1 +46.7R, H2 +19.0R, BOTH HALVES POSITIVE. All 12 months positive.

**PRODUCTION SPEC (V13g `PROD_delta_only`):**
| Metric | Value |
|--------|-------|
| Trades/year | 164 |
| Win Rate | 89.0% |
| Total R | +65.6R |
| R/Day | +0.26 |
| Calmar | 22.9 |
| Max Drawdown | 2.9R |
| Max Consec Loss | 2 |
| Daily WR | 87.1% |
| Weekly WR | 90.2% |
| Short: 109 trades, 90.8% WR | Long: 55 trades, 85.5% WR |

**How it works:**
- Opening Range: 9:30-9:45 AM ET (first 15 minutes)
- SHORT: Price breaks above OR high, then closes back below → short at next bar open, target OR midpoint
- LONG: Price breaks below OR low, then closes back above → long at next bar open, target OR midpoint
  - FILTER: Only take longs when OR delta < 0 (bearish opening range)
- Stop: OR extreme + 5pt buffer
- Trail: Triggers at 0.3R, trails at 30% of max favorable (TIGHT — locks profits fast)
- 88.3% of trades exit via trail stop at avg +0.619R, only 11.7% are full losses
- Commission: 1pt RT + 0.5pt/side slippage (conservative)

**Scripts:** `run_v13c_wide_stop_mine.py` through `run_v13g_production_spec.py`
**Results:** `models/V13g_production.json`, `models/V13g_all_trades.json`

**V13h Bias Audit (Feb 22, 2026):**
- 8/10 checks passed. No look-ahead bias detected.
- Entry timing: 164/164 correct. Stop/target priority: correct. OR computation: correct.
- Time shift: Edge degrades with delay (82.4% at +1 bar vs 89.0%) — legitimate signal.
- Commission: Survives 3x commission (77.4% WR, +30.4R at 3pt).
- Random entries with same trail: 79% WR (mathematically expected from trail asymmetry). OR_FAIL signal adds +10pp WR AND 2.6x avg R per trade vs random.

**V13i Regime Stability (Feb 23, 2026):**
- All 4 quarters profitable (Q1 +7.1R, Q2 +24.9R, Q3 +29.5R, Q4 +2.4R)
- All 5 DOWs profitable (worst: Thursday 84.8% WR, best: Friday 93.3% WR)
- 100% of Monte Carlo subsamples profitable (1000 iterations, 80% sample)
- Max 2 consecutive losses. Profit factor: 4.2.
- 56% of trades in first 30 minutes (88.5% WR). Late signals (120min+): 100% WR on 11 trades.

**V13j Complementary Concepts (Feb 23, 2026):**
- IB_FAIL (30-min IB failure): 174 trades, 86.8% WR, +54.2R, Calmar 19.3, walkforward PASS
- SESSION_EXTREME (PM reversal): 259 trades, 81.1% WR, +32.1R, walkforward PASS
- VWAP_BOUNCE: 430 trades, 76.5% WR, +33.6R, walkforward PASS
- MICRO_SWEEP: 7301 trades, 88.3% WR — mostly trail mechanics (28/day, not practical for manual)

**V13k OR_FAIL Variants (Feb 23, 2026) — ALL 11 CONFIGS WALKFORWARD PASS:**
- RE-ENTRY 3x: 395 trades, 83.5% WR, +109.9R, 0.43 R/day, Calmar 21.6
  - 3rd entries at 85.2% WR — BETTER than 2nd entries (79.2%)
- OR_10min: 188 trades, 83.0% WR, +74.3R, best Calmar (27.6)
- Short re-entry 3x: 263 trades, 83.7% WR, +80.1R, Calmar 23.7

**V13l Overlap Analysis (Feb 23, 2026):**
- 112 UNIQUE IB_FAIL trades (non-overlapping with OR_FAIL): 87.5% WR, +40.5R

**V13m FINAL PORTFOLIO MODELS (Feb 23, 2026):**

| Portfolio | Trades | WR | R | R/Day | Calmar | Max DD | Weekly WR | All Months + |
|-----------|--------|-----|---|-------|--------|--------|-----------|-------------|
| **SNIPER** (OR_FAIL only) | 154 | 88.3% | +64.0R | 0.25 | 22.3 | 2.9R | 90.2% | YES |
| **SHARPSHOOTER** (OR+IB) | 266 | 88.0% | +104.5R | 0.41 | **30.0** | 3.5R | 90.9% | YES |
| **FULL** (OR+IB+EXTREME) | 525 | 84.6% | +136.6R | **0.54** | 20.6 | 6.6R | **96.3%** | YES |

SHARPSHOOTER is the best risk-adjusted model: 30.0 Calmar, 3.5R max DD, 90.9% weekly WR.
FULL PORTFOLIO trades 2.1/day, delivers +136.6R (0.54 R/day), 96.3% weekly WR.

**Scaling (SHARPSHOOTER, 1 NQ @ $20/pt, avg risk $212/trade):**
- 1 contract: $22K/yr, $742 max DD
- 5 contracts: $111K/yr, $3,710 max DD
- 10 contracts: $222K/yr, $7,420 max DD

**All scripts:** `run_v13h_bias_audit.py` through `run_v13m_final_synthesis.py`
**All results:** `models/V13h_audit.json` through `models/V13m_synthesis.json`

### V13n-V13t Extended Mining (Feb 23, 2026) — DUAL OR SYSTEM

**V13o MICRO_SWEEP Look-Ahead Bug Found:**
- 5M→1M entry had look-ahead: `c5m.index[i]` is bar START (pandas `label='left'`), entering at bar start = 3-4 min before bar close is known
- Fix: `bar_close_time = c5m.index[i] + pd.Timedelta(minutes=5)` then enter on first 1M bar after
- **After fix: 78.6% WR = random trail baseline. MICRO_SWEEP is DEAD.**
- All 1M→1M concepts (OR_FAIL, IB_FAIL, SESSION_EXTREME) verified clean — use `entry_idx = i + 1`

**V13q Production Indicator Overlays:**
- Tested EMA20/50, ATR14, Bollinger Bands, OR zone on OR_FAIL + IB_FAIL + SESSION_EXTREME
- 2017 trades, 82.0% WR, +446.6R, Calmar 70.8, 98.1% weekly WR (combined baseline)
- Best filter: inside OR zone = 83.8% WR (+1.8pp). No killer indicator filter found — concepts already well-filtered.

**V13r New Viable Concepts:**
- **OR_BREAK_CONTINUE**: 2+ consecutive 1M closes past OR = ride breakout. 290t, 80.7% WR, +57.5R, Calmar 6.1, **PASS WF**
- **PREV_CLOSE_BOUNCE**: Previous day close as S/D level. 124t, 83.9% WR, +27.6R, Calmar 4.8, **PASS WF**
- INTRADAY_SD: 45,556 trades = trail performance only (178/day, no signal edge)
- DRIVE_RIDER, CONSEC momentum: Dead

**V13s Marubozu at Open:** Dead. Strict marubozu = 75.6% WR (BELOW random baseline). Strong bar 1-3 = 80.4% (trail baseline).

**V13t DUAL OR SYSTEM — PRODUCTION PORTFOLIO UPDATE:**

| Portfolio | Trades | WR | Total R | R/Day | Calmar | Max DD | Weekly WR | WF |
|-----------|--------|-----|---------|-------|--------|--------|-----------|-----|
| OR_FAIL (3x re-entry) | 737 | 83.9% | +239.1R | 0.94 | 59.5 | 4.0R | 94.4% | PASS |
| OR_BREAK_2C | 311 | 81.4% | +65.7R | 0.26 | 7.5 | 8.7R | 72.2% | PASS |
| PREV_CLOSE_BOUNCE | 124 | 83.9% | +27.6R | 0.11 | 4.8 | 5.8R | 68.6% | PASS |
| **DUAL_OR** (Fail+Break) | 1,048 | 83.1% | +304.8R | 1.20 | 50.3 | 6.1R | 94.4% | PASS |
| **DUAL_OR + PCB** | 1,172 | 83.2% | +332.4R | 1.30 | 46.5 | 7.1R | **96.3%** | PASS |

Key findings:
- 2-close confirmation is OR_BREAK sweet spot (3C drops to 78.2%, 4C to 74.3%)
- Target R is irrelevant — trail exits everything (1.5/2.0/3.0 all identical)
- 100% day coverage — signals on every single trading day (255/255)
- Every month is green (worst: Aug +13.4R, best: Apr +37.1R)
- WF: H1 +180.8R, H2 +151.6R (both halves positive)

**Updated Production Portfolios (V13t):**

| Portfolio | Trades | WR | R/Day | Calmar | Weekly WR |
|-----------|--------|-----|-------|--------|-----------|
| **SNIPER** (OR_FAIL 1x) | 154 | 88.3% | 0.25 | 22.3 | 90.2% |
| **SHARPSHOOTER** (OR_FAIL 3x) | 737 | 83.9% | 0.94 | 59.5 | 94.4% |
| **DUAL OR** (OR_FAIL 3x + OR_BREAK_2C) | 1,048 | 83.1% | 1.20 | 50.3 | 94.4% |
| **FULL** (Dual OR + PREV_CLOSE + IB_FAIL + EXTREME) | ~1,700+ | ~83% | ~1.7+ | ~40+ | ~98% |

**All scripts:** `run_v13o_sweep_deep.py` through `run_v13t_or_dual_system.py`
**All results:** `models/V13o_*.json` through `models/V13t_dual_or.json`

### What's Next (Priority Order)
1. ~~All V7 studies through V7z~~ — **DONE**.
2. ~~V8 overnight mining~~ — **DONE**.
3. ~~V9 production specification~~ — **DONE**.
4. ~~V13 OR_FAIL mining~~ — **DONE**. Full portfolio validated (V13h-V13m).
5. **TRANSLATE OR_FAIL to MotiveWave or manual trading rules** — simple enough for manual execution
6. **IMPLEMENT V8g multi-TF alignment in intraday_scanner.py**:
   - Add 6-TF candle alignment (1M/3M/5M/10M/15M/1H) at each BOS_FVG entry
   - Count confirming/contradicting TFs. 10M MUST confirm.
   - Tier classification: Tier 1 (>=4conf, 0contra), Tier 2 (>=4conf, <=1contra)
   - Premium/discount zone detection (optional, +10pp WR)
7. **IMPLEMENT V7y+V7z+V7aa optimizations in scanner**:
   - Skip trades where 5M candle contradicts trade direction (V7aa)
   - VA_fade: wait_win_then_only + aggressive sizing after first win
   - BOS_FVG: 5M good pattern gate + no 15S contradiction
   - sweep_reversal: pause_after_3L
8. **Model 1 — Pull Batch 3 results from VPS** — need SSH access
9. **Correlation analysis**: Measure trade-level correlation between models
10. **Wire live Databento feed** for real-time processing on VPS

### Known Issues
- ~~**Inverse signal logic (backtester) is broken**~~ — **RESOLVED (V7k/V7l/V7m)**: inverse works as hedge, not standalone. Use streak-based regime detection to activate. NEVER invert BOS_FVG.
- **B/E kills Model 2 edge** — validated over full year. Don't add B/E to backtester signals.
- ATR for stops should use 1-minute bars (currently uses different timeframe)
- QC has 64K file size limit — main_v4.py was compressed to 47K but will need multi-file split
- Cowork sandbox blocks outbound HTTP — use Claude Code for API/SSH work
- Git index.lock on mounted filesystem can't be deleted — use fresh clone in `/sessions/` for git operations
- Vol regime uses static VIX=18.5 in backtest — needs live VIX feed for production
- RANGE classification accuracy (43.7%) is the weakest
- **Gap size does NOT equal risk** — position sizing normalizes to 1R. Don't filter on gap size for risk reasons.

---

## Infrastructure Map

```
YOUR MAC (Development)
├── ~/Documents/trading-system/     ← Source code, configs, docs
├── Claude Code (CLI)               ← Engineering tool: SSH, git, APIs
├── Cowork (Desktop)                ← Browser tasks, docs, visual work
└── OpenClaw                        ← Telegram bot interface

        │ git push/pull (GitHub PAT in .local/config.json)
        │ SSH (root@172.93.181.209)
        ▼

VPS — QuantVPS Pro (172.93.181.209)
├── ~/trading-system/batch-runner/  ← Strategy code + batch scripts
├── ~/trading-system/context-engine/ ← [TO BUILD] Context engine
├── /root/.env                      ← QC credentials
├── Cron: 10 PM CT nightly batch
└── nohup batch processes

        │ QC API (SHA256 hash auth)
        ▼

QUANTCONNECT (Backtesting Engine)
├── Project 28083727                ← main.py (uploaded from main_v4.py)
├── NQ futures historical data
└── Backtest execution + results

        │ [TO BUILD] Tick data
        ▼

DATABENTO (Market Data) — DATA PURCHASED & PROCESSED
├── trading_operator/data/     ← 27 files (Jan-Feb 2026), ~140MB
├── trading_operator/data2/    ← 312 files (Feb 2025-Feb 2026), ~1.6GB
├── NQ tick trades with aggressor side (B/A)
├── 255 trading days processed → features_full_year.json
└── Contracts: NQH5→NQM5→NQU5→NQZ5→NQH6 (auto front-month detection)
```

---

## Credentials Quick Reference

**DO NOT hardcode credentials. Always read from config files.**

| Service | Location | Fields |
|---------|----------|--------|
| QuantConnect API | `.local/config.json` | user_id, api_token, project_id |
| GitHub PAT | `.local/config.json` | token (expires May 15, 2026) |
| Databento | VPS `/root/.env` | DATABENTO_KEY |
| OpenClaw/Telegram | `~/Downloads/openclaw.json` | bot token, gateway port |
| VPS SSH | Direct | root@172.93.181.209 (password auth) |

---

## Strategy Quick Reference

**Instrument**: NQ E-mini NASDAQ-100 futures
**Point value**: $20/point
**Tick size**: 0.25 points ($5/tick)

**Execution modeling (main_v4.py Initialize):**
- Slippage: 0.25 pts/side (1 tick)
- Commission: $4.50 round-trip/contract
- Tick-through: 0.25 pts past FVG entry for fill
- BE offset: 0.50 pts above entry (covers $9.50 exit cost)

**Model 1 (QC)**: sweep_fvg (sweep + IFVG), htf_fvg_fill. BE modes: 0.25R-1.0R, liquidity-based. R targets: variable. Sessions: Asia, London, NY AM, NY Lunch, NY PM, Overlap.
**Model 2 (Backtester)**: BOS_FVG, sweep_reversal, VA_fade, pre_gap_fill. No B/E (kills edge). Limit order fills. Confluence ≥2. +3.98R/day validated.

---

## Context Engine Vision

The context engine predicts what kind of session is coming, then picks the best strategy config for that environment. It updates predictions in real-time at 9 checkpoints throughout the day.

**Session types**: Trend, Range, Expansion, Compression, News-Shock
**Feature layers**: Price structure → Volume profile → Order flow → News/calendar → Cross-asset
**Full spec**: See `Tempo_Context_Engine_Spec.docx`

The engine uses Bayesian belief updating — it forms a prior before the session, then updates probabilities as volume, delta, POC migration, and price structure evidence arrives. Every update is logged for post-hoc accuracy analysis.

### NinjaTrader Strategies — keylevel_sweep_fade/ (Mar 6, 2026)

**SweepBreak (v1-v17):**
- Pattern: previous bar (A) is reference candle (min 10pt range). Current bar sweeps one side → stop-market entry on the other side. Single-candle sweep+break.
- v15: rewrote to scan every tick (intra-bar sweep detection, entry on same bar)
- v16: B/E fix (1pt buffer + try/catch around ChangeOrder for race condition)
- v17: improved stats table (box sections, PF, avg win/loss, open R, bars held)
- Key bugs fixed across versions: market order timing, re-entry dedup, session close catch-all, B/E price validation, realtime transition reset
- **Rule: ALWAYS create new version file — never edit existing**

**DojiBreak (v1):**
- Pattern: doji candle (body/range ≤ 25%, min 5pt range). Place stop orders on BOTH sides (buy stop above high + 1pt, sell stop below low - 1pt). Whichever fills = entry, other side + 3pt = stop.
- B/E after 1 bar, TP at bar 3 close. Session: 09:30-15:55 ET.
- **Full strategy overview**: `keylevel_sweep_fade/DOJI_BREAK_STRATEGY.md`

### Doji Break Tick-Data Backtest (Mar 6-7, 2026) — VALIDATED ON 312 DAYS

**Full strategy doc**: `keylevel_sweep_fade/DOJI_BREAK_STRATEGY.md`

**Setup**: Doji candle (body ≤ 25% of range, range ≥ 5pts). Bracket stop orders on both sides. Tick-by-tick simulation — conservative (stop checked before TP on each tick). Without B/E: completely dead (~43% WR). B/E IS the edge.

**ASIA SESSION OUTPERFORMS US (Mar 7, 2026):**

| Session | Trades/Day | WR | Avg/Trade | R/Day | Calmar | DWR |
|---------|-----------|-----|-----------|-------|--------|-----|
| **Asia** (UTC 0-8) | 20.6 | **88.5%** | **+2.33** | +48.0 | **293.8** | **77%** |
| US (UTC 14:30-21) | 53.7 | 81.9% | +2.00 | +107.1 | 96.4 | 74% |

**Best Asia config**: BE=1 off=3pt, body<=0.35, rng>=5 → **89% WR, ~59 R/day, Cal 589**
- Walk-forward: H1 Cal 162 → H2 Cal 208 (OOS BETTER than IS)
- Widening body to 0.35 is free — same avg, +44% more trades
- Asia early (UTC 0-4) best sub-window (+2.57 avg/trade)
- Survives 1pt slippage (Cal 29.2)

**Why Asia works better (Leppyrd framework @LeppyrdTrading):**
- Asia/London creates the "manipulation wick" — the daily candle's wick = our doji
- The doji breakout IS the London reversal that discretionary traders trade
- Our B/E management mechanically captures what Leppyrd trades with HTF context
- Key concepts: daily bias from prev close → Asia manipulation → 5m breaker entry → PDL/PDH target
- "Don't mess with inside candles" — skip if doji is inside prev bar (unless SMT)

**US session results (BE=2pt offset, 16,741 trades):**
- TP winners: +25.4 pts (16.8%), B/E exits: +1.50 pts (65.1%), Stops: -18.0 pts (18.1%)
- Avg risk: 17.3 pts (median 14.5). Slippage: survives up to ~1pt.

**Scripts**: `/tmp/doji_break_v2.py` (US), `/tmp/doji_asia_mine.py` (Asia mining)
**Results**: `results/doji_break_312day.json`, `results/doji_asia_mine.json`
**NT Strategy**: `keylevel_sweep_fade/DojiBreakv2.cs` (improved table + B/E fix)

---

## How to Work With Harrison

1. **Be direct.** No preambles, no "great question." Just answer.
2. **Build, don't describe.** He wants code and results, not architecture docs about code and results.
3. **Ask once, then do.** Don't keep checking in. Make decisions and move.
4. **Update this file** after every significant session. Add what was built, what changed, what's next.
5. **Commit to git** after every session. This is the memory system.

---

## Session End Checklist

Before ending any session:
1. Update the "Current State" section above with what changed
2. Commit and push to GitHub: `git add -A && git commit -m "session update" && git push`
3. If credentials or integrations changed, update CLAUDE_BOOTSTRAP.md too
4. If new tools/bots/services were built, document them in CLAUDE_BOOTSTRAP.md "Built Integrations"

**The #1 rule: If it's not in this file or linked from this file, it doesn't exist next session.**
