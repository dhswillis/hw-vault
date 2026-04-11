# MotiveWave BOS_FVG Strategy — Setup Guide

## Quick Start

### Step 1: Compile
```bash
cd ~/Documents/trading-system
./compile_mw_strategy.sh
```

If compilation fails with import errors, you need to find the correct SDK jar path. Run:
```bash
find /Applications/MotiveWave.app -name "*.jar" | head -20
```
Then compile manually:
```bash
javac -cp "/path/to/the/sdk.jar" -d "$HOME/MotiveWave Extensions/" BOSFVGStrategy.java
```

### Step 2: Restart MotiveWave
Close and reopen MotiveWave. The strategy needs to be loaded fresh.

### Step 3: Import Data (if needed)
Your 15-second NQ data is at: `~/Documents/trading-system/nq_15s_bars_3mo.csv`

In MotiveWave:
1. **File → Import → Historical Data** (or similar)
2. Select the CSV file
3. Map to NQ instrument
4. Set period to 15 seconds
5. Import

**Alternative**: Connect to Rithmic via your AMP Futures demo account for live + historical data.

### Step 4: Apply Strategy to Chart
1. Open an NQ chart (1-minute timeframe — the strategy works on 1M bars)
2. Right-click the chart → **Studies** (or **Indicators**)
3. Look under **Custom** for "BOS FVG Trailing" by com.willis
4. Add it to the chart
5. Right-click the study → **Settings** to adjust parameters

### Step 5: What You'll See
- **Green up-arrows**: Long entries
- **Red down-arrows**: Short entries
- **Red dashed lines**: Initial stop levels
- **Teal stepped lines**: Trailing stop path
- **Green squares**: Winning exits
- **Red squares**: Losing exits
- **Blue zones**: Bullish FVG areas
- **Orange zones**: Bearish FVG areas
- **Gray dashed lines**: BOS levels with "BOS+" / "BOS-" labels
- **P&L labels**: "+2.3R" or "-1.0R" near each exit

### Step 6: Adjust Parameters
Right-click → Settings. All parameters from clean_backtest.py are configurable:

| Parameter | Default | What it does |
|-----------|---------|-------------|
| Swing Lookback | 3 | Bars for swing detection |
| BOS Min Displacement | 2.0 | Min points for BOS to count |
| BOS→FVG Max Bars | 15 | Window for FVG after BOS |
| FVG Max Fill Bars | 50 | How long FVG stays valid |
| Risk Min/Max | 1.5 / 50.0 | Risk bounds in points |
| Trail Trigger | 0.5R | When trailing starts |
| Trail Buffer | 0.1R | Gap between trail and max |
| Session Start | 13:00 UTC | Trading window start |
| Session End | 20:00 UTC | Trading window end |
| Last Entry | 19:45 UTC | No new trades after this |
| Entry/Exit Slippage | 0.25 | Slippage in points |
| Entry Mode | fvg_edge | fvg_edge or fvg_midpoint |

**Optimized params from mining** (to test once baseline matches):
- Swing Lookback = 2
- BOS Min Displacement = 1.0
- BOS→FVG Max Bars = 20
- FVG Max Fill Bars = 30
- Risk Max = 20.0
- Trail Trigger = 0.75

## Troubleshooting

### "Strategy doesn't appear in MotiveWave"
- Check: `ls ~/MotiveWave\ Extensions/com/willis/BOSFVGStrategy.class`
- Make sure you restarted MotiveWave after compiling
- Check MotiveWave logs: `~/MotiveWave/logs/` or Help → View Log

### "Java version mismatch"
MotiveWave bundles its own JRE. Check what version it uses:
```bash
find /Applications/MotiveWave.app -name "java" -type f | head -5
# Then run: /path/to/motivewave/java -version
```
Compile with matching version: `javac --release 11` (or 8, 17, etc.)

### "Import errors during compilation"
The SDK API class names might differ from what's in the code. Common fixes:

1. **Find correct class names**:
```bash
SDK_JAR=$(find /Applications/MotiveWave.app -name "*sdk*" -name "*.jar" | head -1)
jar tf "$SDK_JAR" | grep -i study | head -20
jar tf "$SDK_JAR" | grep -i marker | head -20
jar tf "$SDK_JAR" | grep -i descriptor | head -20
jar tf "$SDK_JAR" | grep -i line | head -20
jar tf "$SDK_JAR" | grep -i label | head -20
jar tf "$SDK_JAR" | grep -i box | head -20
```

2. **Common renames needed**:
   - `Label` might be `Text` or `TextFigure`
   - `Box` might be `Rectangle` or `Rect`
   - `Line` might need `com.motivewave.platform.sdk.draw.Line`
   - `Marker` constructor might differ
   - `DiscreteDescriptor` might be `StringDescriptor`

3. **Check SDK docs bundled with MotiveWave**:
```bash
find /Applications/MotiveWave.app -name "*.pdf" -o -name "*.html" -o -name "doc*" | head -20
```

### "No trades showing on chart"
1. Make sure you're on a 1-minute NQ chart
2. Check the session times are correct for your timezone
3. Check MotiveWave console output (the strategy logs results via `debug()`)
4. Try reducing BOS Min Displacement to 0.5 to get more signals

### Cross-validation with Python
Run the Python backtest on the same date range and compare:
```bash
cd ~/Documents/trading-system
python3 clean_backtest.py --data-dir databento --max-files 70
```
Trade count, win rate, and total R should match within 5%.
