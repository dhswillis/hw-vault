# OVERNIGHT MINING SESSION — Comprehensive Variable Sweep

## YOUR MANDATE

You are running an overnight mining session. The owner (Harrison) is asleep. You will run ALL NIGHT exploring every variable combination systematically. You have NO opportunity to ask questions — everything you need is in this prompt and the existing codebase.

**CRITICAL RULES:**
1. **ASSUME THERE ARE BUGS.** Before trusting ANY result, sanity-check it. If you see 100% loss rate, 0% WR on any subset, or >95% WR on a large sample — STOP and check for bugs first. The same-bar stop bug (checking stop on entry bar) has already bitten us once.
2. **Log everything to JSON files** in `context-engine/models/` with prefix `V11_overnight_`.
3. **Update `nautilus-bt/CLAUDE.md`** ground truth section whenever you confirm or overturn a finding.
4. **Small stops are OK.** We discovered 15S candle stops (very tight) produce 2x the R per trade. Don't filter them out.
5. **Pure trailing is the exit.** DO NOT use BE+Trail (breakeven at 1R). It's dead (21.8% WR, proven V10aa). Use: trigger=0.5R, trail_buffer=0.1R, no max hold, no max target.
6. **Every result must state which engine produced it** — mining framework or NautilusTrader.

---

## PHASE 0: SETUP & ERROR CHECKS (do this FIRST)

Before running any mines:

1. Read `nautilus-bt/CLAUDE.md` — especially the GROUND TRUTH section
2. Read `V8_MINING_SYNTHESIS.md` — understand what's already been tested
3. Read `context-engine/micro_structure.py` and `context-engine/intraday_scanner.py` — these are your core engines
4. Read `context-engine/run_v10aa_replicate_mining.py` — see how the latest mining scripts work
5. Verify data: `ls ~/trading_operator/data2/GLBX-20260213-YFP9CFN8HF/*.dbn.zst | wc -l` should be ~312 files
6. Run a quick 5-file sanity test on any new script before running full dataset
7. **Check for the same-bar stop bug**: any trade management that evaluates the entry bar's high/low against the stop is wrong. Stops activate NEXT bar.

---

## PHASE 1: THE 7 SIGNAL TYPES (with pure trailing)

These are the 7 signal types from the original mining. ALL were tested with BE+Trail (now dead). Re-mine ALL of them with pure trailing exit (trigger=0.5R, buffer=0.1R).

### Signal 1: BOS_FVG (our main signal)
- BOS on 1M (3-bar swing), FVG same direction within 15 bars
- Entry at FVG edge (fill price), stop at opposite FVG edge
- Already partially done in V10aa — but needs pure trailing version

### Signal 2: sweep_wick_sweep
- Liquidity sweep below swing low (or above swing high)
- Wick shows rejection, closes back inside range
- Entry on the sweep candle close, stop beyond the wick

### Signal 3: sweep_close_through
- Sweep of liquidity + close beyond the level
- Momentum continuation after sweep
- Entry at close, stop at swing extreme

### Signal 4: swing_failure
- Failed swing breakout pattern
- Price breaks structure then immediately reverses
- Entry on reversal bar close, stop beyond the failed swing

### Signal 5: fvg_exit_fvg_stop
- FVG-based entry with stop at FVG edge
- Variant: entry at FVG zone, stop at FVG boundary

### Signal 6: fvg_exit_body_stop
- FVG entry with stop at body of the displacement candle
- Tighter stop than fvg_exit_fvg_stop

### Signal 7: sweep_fvg_combo
- Liquidity sweep + FVG confluence
- Sweep happens, then FVG forms in the reversal direction
- Entry at FVG fill, stop beyond sweep low/high

**For each signal, output:**
- n trades, WR%, avg R, total R, max DD, Calmar
- Monthly breakdown
- Best/worst 20 trades
- Save to `V11_overnight_signal_{name}_pure_trail.json`

---

## PHASE 2: ENTRY VARIATIONS (for BOS_FVG specifically)

Test these entry methods, all with pure trailing:

### Entry A: FVG Edge (baseline)
Fill price when price first touches FVG zone.

### Entry B: FVG Midpoint
Limit order at midpoint of FVG. Tighter risk.

### Entry C: 1M Rejection Candle
Bar dips into FVG, closes back outside. Entry at bar close.
Stop at bar extreme OR FVG edge (test both).
V8h showed 58.8% WR with this.

### Entry D: 15S Rejection Candle
Same as C but on 15-second bars. V8j showed this is the R-maximizer (2x avg R due to tight stops).

### Entry E: Rejection + Momentum
Bar dips into FVG, closes back outside, AND close is in direction of trade (body > 50% of range).

### Entry F: Full FVG Fill
Price touches far edge of FVG (full fill), then reverses. Entry on reversal bar close.

### Entry G: Midpoint Away
Price touches FVG midpoint, closes away from it in trade direction.

**For each entry, test with:**
- FVG edge stop
- Candle extreme stop (bar high/low)
- FVG midpoint stop
- Save to `V11_overnight_entry_{method}_pure_trail.json`

---

## PHASE 3: TIMEFRAME VARIATIONS

### Base timeframes for BOS detection:
Test BOS on: 15S, 1M, 3M, 5M, 15M

### FVG timeframes:
Test FVG on: 15S, 1M, 3M, 5M

### Cross-timeframe combos:
Test every combination where BOS_TF >= FVG_TF:
- 5M BOS + 1M FVG (V8p showed 87.7% WR at V8g>=5)
- 15M BOS + 1M FVG
- 15M BOS + 5M FVG
- 1M BOS + 15S FVG
- 3M BOS + 1M FVG
- 5M BOS + 3M FVG

For each combo, run with pure trailing and V8g alignment filter at thresholds: >=0, >=2, >=3, >=4, >=5.

Save to `V11_overnight_tf_cross_{bos_tf}_{fvg_tf}.json`

---

## PHASE 4: ALIGNMENT & FILTER SWEEP

### V8g Multi-TF Candle Direction (already proven — re-validate with pure trail)
- 6 timeframes: 1M, 3M, 5M, 10M, 15M, 1H
- Test confirming TF thresholds: 0, 1, 2, 3, 4, 5, 6
- Test contradicting TF max: 0, 1, 2, 3 (any)
- V8g>=4 + 0 contra was the gold standard (80%+ WR)

### V8u Full TF Stack (5M/15M/1H/4H/Daily)
- Count how many of 5M, 15M, 1H, 4H, Daily candle direction confirms
- Test TFs aligned: 0/5 through 5/5
- Cross with V8g filter

### Higher-TF BOS Alignment
- Does a 5M or 15M BOS in the same direction exist?
- double_bos_aligned vs single_bos_aligned vs no_bos_context

### Session Filters
Sessions to test individually and in combination:
- london_open (08:00-09:30 UTC)
- london_am (09:30-13:00 UTC)
- ny_am (13:30-16:00 UTC)
- ib_first (13:30-14:30 UTC — initial balance)
- silver_bullet (14:00-15:00 UTC)
- overlap (13:00-16:30 UTC — London/NY overlap)
- news_window (13:30-14:30 UTC on FOMC/NFP days)
- power_hour (19:00-20:00 UTC)
- asian (00:00-08:00 UTC)

### Day of Week
- Test Mon through Fri independently
- V8 showed BOS_FVG degrades on Fridays

### NY/London Session High/Low Reversals (NEW — not yet tested)
- Track Asian session high/low (00:00-08:00 UTC)
- Track London session high/low (08:00-13:00 UTC)
- Track NY AM high/low (13:30-16:00 UTC)
- When price sweeps the London high or low during NY session → reversal signal
- When price sweeps Asian high/low during London → reversal signal
- Test as standalone signal AND as filter on BOS_FVG

### FVG Fill Percentage
- How much of the FVG was filled before entry?
- Test buckets: 0-25%, 25-50%, 50-75%, 75-100%
- Do partial fills (just touching the edge) perform differently from deep fills?

Save all to `V11_overnight_filters_{name}.json`

---

## PHASE 5: ORDERFLOW & FOOTPRINT PATTERNS

### Delta Divergence at Entry
- Calculate net delta on the entry bar
- strong_buy_delta, moderate_buy, neutral, moderate_sell, strong_sell
- Does delta CONFIRMING the trade direction help? Or does DIVERGENCE (exhaustion) help?
- V8k showed rej_delta_aligned at 83.8% WR with V8g>=5

### Imbalance Stacks
- Scan for stacked buy/sell imbalances in the 5 bars before entry
- V8b showed strong_buy_imbalance at +2.73R avg

### Volume Profile Context
- Is entry near a high volume node (HVN) or low volume node (LVN)?
- Value area high/low proximity
- V8b showed VP adds value in combination with other triggers

### Rejection Candle Delta Alignment (V8k confirmed)
- On rejection candle: does delta confirm the rejection?
- Delta aligned = delta opposes the wick direction (confirming the rejection)

Save to `V11_overnight_orderflow_{name}.json`

---

## PHASE 6: TRAIL PARAMETER OPTIMIZATION

Test pure trailing with these parameter grids:

### Trail Trigger (when to start trailing):
- 0.25R, 0.5R, 0.75R, 1.0R, 1.5R, 2.0R

### Trail Buffer (how much slack to give):
- 0.05R, 0.1R, 0.15R, 0.2R, 0.3R, 0.5R, 0.75R, 1.0R

### Combined: test ALL trigger × buffer combos = 48 combinations
- Run on the full BOS_FVG dataset with V8g>=3 filter
- Find the Pareto frontier (max WR vs max total R vs min DD)

### Trail Ratcheting:
- Test a ratcheting trail: buffer decreases as R increases
  - At 0.5R: buffer=0.3R
  - At 1.0R: buffer=0.2R
  - At 2.0R: buffer=0.1R
  - At 3.0R: buffer=0.05R

### Time-Decay Trail:
- Buffer shrinks over time (bars held)
- At 5 bars: full buffer
- At 30 bars: buffer × 0.5
- At 60 bars: buffer × 0.25

Save to `V11_overnight_trail_optimization.json`

---

## PHASE 7: COMPOSITE SIGNAL SCORING

Build a composite score that combines the best individual findings:

### Score components (weight each by V8 findings):
1. V8g confirming TF count (3M through 1H): +1 per confirming, -1 per contradicting
2. V8u full TF stack (5M through Daily): +1 per aligned
3. Session quality: +2 for ny_am/ib_first, +1 for overlap/silver_bullet, 0 for others, -1 for london_open
4. HTF BOS alignment: +2 if double_bos_aligned, +1 if single, 0 if none
5. 1M rejection quality: +2 if body > 60% of range, +1 if 30-60%, 0 if < 30%
6. Delta alignment: +1 if confirming at entry
7. Day of week: -1 for Friday BOS_FVG, +1 for Thursday
8. London/Asian high-low sweep: +2 if sweeping a session extreme

### For each composite score threshold (0 through 12):
- Calculate n, WR, avgR, totalR, maxDD, Calmar

Save to `V11_overnight_composite_score.json`

---

## PHASE 8: PORTFOLIO CONSTRUCTION

Using the best findings from Phases 1-7:

1. Select the top 5-8 non-correlated signal×filter combos
2. Calculate pairwise correlation between each pair
3. Build a combined equity curve
4. Run Monte Carlo (1000 simulations, random trade ordering)
5. Walk-forward validate (70/30 chronological split)

Save to `V11_overnight_portfolio.json`

---

## OUTPUT FORMAT

Every JSON output file should include:

```json
{
  "metadata": {
    "version": "V11_overnight",
    "test_name": "descriptive name",
    "date_run": "2026-02-19",
    "engine": "mining_framework",
    "data_files_used": 312,
    "data_date_range": "2025-02-XX to 2026-02-XX",
    "exit_method": "pure_trail_0.5R_trigger_0.1R_buffer",
    "known_caveats": ["mining uses theoretical fills", "no slippage model"]
  },
  "sanity_checks": {
    "total_bars_processed": 0,
    "total_signals_detected": 0,
    "avg_signals_per_day": 0,
    "min_signals_any_day": 0,
    "max_signals_any_day": 0,
    "pct_days_with_zero_signals": 0,
    "any_100pct_loss_rate_subsets": false,
    "any_95plus_wr_subsets_over_50n": false
  },
  "results": {}
}
```

---

## EXECUTION ORDER

Run the phases in this order (each builds on the previous):
1. Phase 0 (setup + error checks) — 10 min
2. Phase 1 (7 signal types) — 2 hours
3. Phase 6 (trail optimization) — 1 hour (run on BOS_FVG while Phase 1 runs the others)
4. Phase 2 (entry variations) — 1 hour
5. Phase 3 (timeframe cross) — 2 hours
6. Phase 4 (filters) — 2 hours
7. Phase 5 (orderflow) — 1 hour
8. Phase 7 (composite score) — 30 min
9. Phase 8 (portfolio) — 30 min

**Total estimated: ~10 hours. You have all night.**

If any phase is taking too long (>3 hours), skip to the next and come back. Breadth first, depth second.

---

## WHAT SUCCESS LOOKS LIKE

By morning, Harrison should wake up to:
1. A `V11_overnight_summary.json` with the top 20 findings ranked by Calmar ratio
2. Updated `nautilus-bt/CLAUDE.md` ground truth with any new validated findings
3. A clear answer to: **"Which signal × entry × filter × trail combo is the best, and how confident are we?"**
4. Trade-level detail for the top 3 combos so we can visually verify them
5. Any bugs discovered documented in the ground truth "mistakes" section
