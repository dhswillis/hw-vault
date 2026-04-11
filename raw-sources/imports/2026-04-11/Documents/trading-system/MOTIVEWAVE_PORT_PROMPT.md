# MotiveWave Strategy Port — BOS_FVG with Pure Trailing Stop

## EXECUTION MODE
You are running with `--dangerously-skip-permissions`. Do NOT ask for confirmation. Do NOT pause for approval. Execute every step autonomously:
- Install packages without asking (`pip install`, `brew install`, etc.)
- Create/modify/delete files without asking
- Run commands without asking
- If something fails, fix it and retry automatically
- Only stop if you hit an unrecoverable error after 3 retries

## MISSION
Port `clean_backtest.py` (Python) to a MotiveWave study/strategy in Java so Harrison can:
1. **Visually see every trade** on a chart (entry arrows, stop lines, trail lines, exit markers)
2. **Run the built-in MotiveWave backtester** against imported 15-second or 1-minute NQ data
3. Eventually **trade live** from MotiveWave connected to a broker

## CRITICAL RULES
- The MotiveWave SDK lives inside the MotiveWave app. On Mac:
  `/Applications/MotiveWave.app/Contents/Java/lib/mwave_sdk.jar`
  (or similar — find it with: `find /Applications/MotiveWave.app -name "*.jar" | head -20`)
- MotiveWave strategies are compiled `.java` files placed in:
  `~/MotiveWave Extensions/` (or wherever MotiveWave looks for custom studies)
- Read the MotiveWave SDK documentation. Start here:
  `find /Applications/MotiveWave.app -name "*.pdf" -o -name "*.html" -o -name "doc*" | head -30`
  Also check: https://www.motivewave.com/sdk/javadoc/
- The strategy must use MotiveWave's `Study` or `Strategy` base class
- DO NOT invent API calls. If unsure, search the SDK jar: `jar tf mwave_sdk.jar | grep -i strategy`

## EXACT LOGIC TO PORT (from clean_backtest.py)

### Configuration Constants
```
SWING_LOOKBACK = 3
FVG_MIN_SIZE = 0.0
FVG_MAX_FILL_BARS = 50
BOS_MIN_DISPLACEMENT = 2.0
BOS_FVG_MAX_BARS = 15
RISK_MIN_PTS = 1.5
RISK_MAX_PTS = 50.0
TRAIL_TRIGGER_R = 0.5
TRAIL_BUFFER_R = 0.1
SESSION_START = 13:00 UTC (8 AM ET)
SESSION_END = 20:00 UTC (3 PM ET)
LAST_ENTRY = 19:45 UTC (2:45 PM ET)
ENTRY_SLIPPAGE_PTS = 0.25
EXIT_SLIPPAGE_PTS = 0.25
ENTRY_MODE = "fvg_edge"
```
ALL of these must be **configurable inputs** in MotiveWave's settings dialog (right-click → Settings).

### Step-by-Step Logic (MUST be identical to clean_backtest.py)

#### 1. Swing Detection (3-bar lookback)
- Swing High at bar i: `high[i] >= max(high[i-3..i])` AND `high[i] >= max(high[i+1..i+3])`
- Swing Low at bar i: `low[i] <= min(low[i-3..i])` AND `low[i] <= min(low[i+1..i+3])`
- A swing is CONFIRMED at index `i + lookback` (confirmed_idx)
- A swing can only be used for BOS detection AFTER its confirmed_idx

#### 2. BOS Detection (Break of Structure)
- BOS Long: `close[i] > latest_confirmed_swing_high.price` AND `close[i-1] <= that price`
  - displacement = `close[i] - swing_high.price` must be >= `BOS_MIN_DISPLACEMENT`
- BOS Short: `close[i] < latest_confirmed_swing_low.price` AND `close[i-1] >= that price`
  - displacement = `swing_low.price - close[i]` must be >= `BOS_MIN_DISPLACEMENT`
- Only use swings where `confirmed_idx < i` (LOOK-AHEAD PREVENTION)

#### 3. FVG Detection (Fair Value Gap)
- Bullish FVG at bar i: `low[i] > high[i-2]` → gap = `[high[i-2], low[i]]`
- Bearish FVG at bar i: `low[i-2] > high[i]` → gap = `[high[i], low[i-2]]`

#### 4. Signal Matching (BOS → FVG → Fill)
- After a BOS fires at bar B, look for FVGs in the SAME direction where:
  - `fvg.idx >= B` (FVG forms during or after BOS)
  - `fvg.idx <= B + BOS_FVG_MAX_BARS` (within 15 bars)
- For each matching FVG, check for FILL starting at `fvg.idx + 1`:
  - Long FVG fill: `low[j] <= fvg.gap_high` → entry at `fvg.gap_high - slippage`
  - Short FVG fill: `high[j] >= fvg.gap_low` → entry at `fvg.gap_low + slippage`
- Stop = opposite edge of FVG (gap_low for longs, gap_high for shorts)
- Risk = `|entry - stop|` must be between `RISK_MIN_PTS` and `RISK_MAX_PTS`
- Entry bar time must be between `SESSION_START` and `LAST_ENTRY`
- Only ONE fill per FVG, only FIRST FVG fill per BOS

#### 5. Trade Management (Pure Trailing Stop)
- Trade management starts the bar AFTER the fill bar
- On each bar, check STOP FIRST (worst case):
  - Long: `bar_low <= current_stop` → exit at `current_stop - slippage`
  - Short: `bar_high >= current_stop` → exit at `current_stop + slippage`
- Then update max favorable R:
  - Long: `current_r = (bar_high - entry) / risk`
  - Short: `current_r = (entry - bar_low) / risk`
- Then update trailing stop if `max_favorable_r >= TRAIL_TRIGGER_R`:
  - `trail_r = max_favorable_r - TRAIL_BUFFER_R`
  - Long new_stop = `entry + trail_r * risk`
  - Short new_stop = `entry - trail_r * risk`
  - Only TIGHTEN, never loosen the stop
- Session end check: if `bar_time >= SESSION_END` → exit at close ± slippage

## VISUAL REQUIREMENTS (the whole point of this port)

The strategy MUST draw on the chart:
1. **Entry markers**: Green up-arrow for long entries, Red down-arrow for short entries
2. **Stop loss lines**: Horizontal dashed red line from entry bar to exit bar at initial stop price
3. **Trailing stop path**: Stepped line showing how the trail moved up/down over the trade
4. **Exit markers**: Square marker at exit point (green if profitable, red if loss)
5. **FVG zones**: Semi-transparent rectangles showing the FVG gap zone (blue for bullish, orange for bearish)
6. **BOS lines**: Horizontal line at the broken swing level with "BOS" label
7. **P&L label**: Text near exit showing R-multiple result (e.g., "+2.3R" or "-1.0R")

Use MotiveWave's `Marker`, `Figure`, `PathInfo`, or equivalent drawing API.

## DATA SETUP

A 3-month CSV of 15-second NQ bars is already prepared:
- File: `~/Documents/trading-system/nq_15s_bars_3mo.csv`
- Format: Date,Time,Open,High,Low,Close,Volume
- Date range: 2025-11-23 to 2026-02-11
- 290,410 bars

### Importing into MotiveWave:
1. Open MotiveWave
2. File → Import → Historical Data (or Configure → Data Sources → Import)
3. Point to the CSV file
4. Map instrument to NQ (E-mini Nasdaq 100 futures)
5. Set bar period to 15 seconds
6. Import

If MotiveWave doesn't support direct CSV import of historical bars, alternatives:
- Check if there's a `File → Replay` or `File → Load Custom Data` option
- MotiveWave may need data through a connection (Rithmic, CQG, IB). If so, set up a free AMP Futures demo account with Rithmic and use that for data.
- As a LAST RESORT, we can overlay our Python trade results on MotiveWave charts by importing trade markers only (entry/exit times + prices from clean_backtest_results.csv)

## FILE STRUCTURE

Create the following files:
```
~/MotiveWave Extensions/
  └── com/willis/
      └── BOSFVGStrategy.java      # Main strategy file

~/Documents/trading-system/
  └── compile_mw_strategy.sh        # Build script
```

## BUILD & INSTALL

The compile script should:
```bash
#!/bin/bash
SDK_JAR=$(find /Applications/MotiveWave.app -name "mwave_sdk.jar" -o -name "study_sdk.jar" | head -1)
if [ -z "$SDK_JAR" ]; then
    echo "ERROR: Cannot find MotiveWave SDK jar"
    echo "Searching all jars..."
    find /Applications/MotiveWave.app -name "*.jar"
    exit 1
fi

EXTENSIONS_DIR="$HOME/MotiveWave Extensions"
mkdir -p "$EXTENSIONS_DIR"

javac -cp "$SDK_JAR" -d "$EXTENSIONS_DIR" BOSFVGStrategy.java
echo "Compiled. Restart MotiveWave to load the strategy."
```

## VERIFICATION

After building:
1. Restart MotiveWave
2. The strategy should appear in Study → Custom or Strategy → Custom
3. Apply it to an NQ chart with the imported data
4. Run the MotiveWave backtester (Strategy → Backtest or similar)
5. You should see trade arrows, stop lines, and P&L on the chart

### Cross-Validation Check
Run the Python backtest on the SAME 3-month window and compare:
```bash
cd ~/Documents/trading-system
python3 clean_backtest.py --data-dir databento --max-files 70
```
The total trades, win rate, and total R should be within 5% of each other. If they diverge more than that, there's a porting bug.

## DEBUGGING

If the strategy doesn't load:
- Check MotiveWave's log: `~/MotiveWave/logs/` or similar
- Make sure Java version matches what MotiveWave uses (check with `java -version` and compare to MotiveWave's bundled JRE)
- Try `jar tf mwave_sdk.jar | grep -i study` to find the correct base class name

If trades don't match Python:
- Export MotiveWave trades to CSV
- Diff against `clean_backtest_results.csv` trade by trade
- Most likely culprits: swing confirmation timing, BOS displacement check, FVG fill bar indexing

## IMPORTANT NOTES
- This is NQ futures (E-mini Nasdaq 100). Tick size = 0.25, point value = $20
- The strategy runs on 1-MINUTE bars internally (build from 15s if needed), same as clean_backtest.py
- MotiveWave may call bars "periods" — adapt terminology accordingly
- DO NOT skip the visual markers — they are the ENTIRE REASON we moved to MotiveWave
- ALL config values must be editable in the MotiveWave Settings dialog so we can sweep params visually
- If MotiveWave's SDK doesn't support something (e.g., custom bar resampling), document the limitation and suggest a workaround
