# MotiveWave BOS_FVG Strategy — Compile & Fix Session

You are running an autonomous session. DO NOT ASK QUESTIONS. DO NOT STOP. If something fails, log the error, fix it, and retry. Use `--dangerously-skip-permissions`.

## CRITICAL: Read Before Anything Else
1. Read this entire prompt top to bottom before doing anything
2. Read `CLAUDE.md` in this directory — it has ground truth and project context
3. Read `BOSFVGStrategy.java` top to bottom — this is your ONLY Java file. Do NOT create new Java files.
4. Read `compile_mw_strategy.sh` — this is the build script you'll update at the end

## YOUR SINGLE GOAL
`BOSFVGStrategy.java` in `~/Documents/trading-system/` is a port of `clean_backtest.py` to MotiveWave's Java SDK. It was written with GUESSED API calls. Your job:
- **Explore the real SDK** to discover the actual class names, method signatures, and constructors
- **Fix every SDK mismatch** in the Java file so it compiles cleanly
- **Compile it** and place the .class in `~/MotiveWave Extensions/`
- **Verify** the compiled class file exists and is valid
- **Update** `compile_mw_strategy.sh` with the real SDK jar path

**YOU MUST USE `BOSFVGStrategy.java`.** Fix it — do NOT rewrite it, do NOT create a new file, do NOT create abstractions. Just fix the SDK calls, compile it, done.

## Current State
- `BOSFVGStrategy.java` exists with complete trading logic (swings, BOS, FVGs, signals, trades, drawing)
- The trading logic is correct (exact port of `clean_backtest.py`) — DO NOT TOUCH IT
- The SDK API calls (imports, class names, method signatures, constructors) are guessed and probably wrong
- MotiveWave is installed at `/Applications/MotiveWave.app/`
- The compiled .class file needs to end up in `~/MotiveWave Extensions/com/willis/BOSFVGStrategy.class`

## PHASE 0: SDK DISCOVERY (do this FIRST — before editing ANY Java)

### 0a. Find the SDK jar
```bash
# Find all jars
find /Applications/MotiveWave.app -name "*.jar" 2>/dev/null | head -30

# Find the one with Study.class
for jar in $(find /Applications/MotiveWave.app -name "*.jar" 2>/dev/null); do
    if jar tf "$jar" 2>/dev/null | grep -q "Study.class"; then
        echo "SDK JAR: $jar"
        export SDK_JAR="$jar"
    fi
done
```

If you can't find the SDK jar in `/Applications/MotiveWave.app/`, also check:
- `~/MotiveWave/lib/`
- `~/MotiveWave Extensions/lib/`
- `/Applications/MotiveWave.app/Contents/Java/lib/`
- `/Applications/MotiveWave.app/Contents/Resources/lib/`

**If MotiveWave is NOT installed** (no `/Applications/MotiveWave.app/`):
1. Check: `ls /Applications/ | grep -i motive`
2. Check: `find / -name "MotiveWave*" -type d 2>/dev/null | head -10`
3. If truly not installed, log the error and stop — Harrison needs to install it first

### 0b. Dump ALL SDK classes
```bash
jar tf "$SDK_JAR" | sort > /tmp/mw_sdk_classes.txt
wc -l /tmp/mw_sdk_classes.txt
cat /tmp/mw_sdk_classes.txt
```

### 0c. Find the classes we need
```bash
echo "=== Study/Strategy ==="
jar tf "$SDK_JAR" | grep -i "study\|strategy" | grep ".class$"

echo "=== Drawing ==="
jar tf "$SDK_JAR" | grep -i "draw\|figure\|marker\|line\|box\|rect\|label\|text" | grep ".class$"

echo "=== Descriptors ==="
jar tf "$SDK_JAR" | grep -i "desc\|setting" | grep ".class$"

echo "=== Enums ==="
jar tf "$SDK_JAR" | grep -i "enum" | grep ".class$"

echo "=== DataSeries ==="
jar tf "$SDK_JAR" | grep -i "dataseries\|data.*series" | grep ".class$"

echo "=== Coordinate ==="
jar tf "$SDK_JAR" | grep -i "coordinate\|coord" | grep ".class$"

echo "=== Annotation ==="
jar tf "$SDK_JAR" | grep -i "studyheader\|annotation" | grep ".class$"

echo "=== Defaults ==="
jar tf "$SDK_JAR" | grep -i "defaults" | grep ".class$"
```

### 0d. Find example Java files (SDK often bundles these)
```bash
# SDK zip with examples
find /Applications/MotiveWave.app -name "*sdk*" -name "*.zip" 2>/dev/null
find /Applications/MotiveWave.app -name "*example*" -o -name "*sample*" 2>/dev/null | head -10
find ~/MotiveWave* -name "*.java" -o -name "*.pdf" -o -name "*.html" 2>/dev/null | head -20

# If there's an SDK zip, extract and READ the example .java files
SDK_ZIP=$(find /Applications/MotiveWave.app ~/MotiveWave* -name "*sdk*" -name "*.zip" 2>/dev/null | head -1)
if [ -n "$SDK_ZIP" ]; then
    mkdir -p /tmp/mw_sdk_docs
    unzip -o "$SDK_ZIP" -d /tmp/mw_sdk_docs
    find /tmp/mw_sdk_docs -name "*.java" | head -20
fi
```

**If you find example .java files, READ THEM ALL.** They are the ground truth for how the SDK works. Look for:
- `@StudyHeader` annotation fields
- How `initialize()` works (settings descriptors)
- How `calculate()` accesses bar data
- How figures are added to the chart
- Constructor signatures for Line, Marker, Label, Box, Coordinate
- Whether `var` keyword is used (Java version)

## PHASE 1: JAVAP INSPECTION

Use `javap` on EVERY class that `BOSFVGStrategy.java` uses. Save ALL output to a reference file.

```bash
# Create a comprehensive API reference
echo "" > /tmp/mw_api_reference.txt

# For each class we use, inspect it
for CLASS_PATH in "Study" "Marker" "Line" "Coordinate" "DataSeries" "Defaults" "Enums"; do
    MATCH=$(jar tf "$SDK_JAR" | grep "/${CLASS_PATH}.class" | head -1)
    if [ -n "$MATCH" ]; then
        FQN=$(echo "$MATCH" | sed 's/.class$//' | tr '/' '.')
        echo "=== $FQN ===" >> /tmp/mw_api_reference.txt
        javap -cp "$SDK_JAR" "$FQN" >> /tmp/mw_api_reference.txt 2>&1
        echo "" >> /tmp/mw_api_reference.txt
    fi
done

# Also inspect descriptor classes
for DESC in "IntegerDescriptor" "DoubleDescriptor" "BooleanDescriptor" "DiscreteDescriptor" "StringDescriptor" "SettingsDescriptor"; do
    MATCH=$(jar tf "$SDK_JAR" | grep "/${DESC}.class" | head -1)
    if [ -n "$MATCH" ]; then
        FQN=$(echo "$MATCH" | sed 's/.class$//' | tr '/' '.')
        echo "=== $FQN ===" >> /tmp/mw_api_reference.txt
        javap -cp "$SDK_JAR" "$FQN" >> /tmp/mw_api_reference.txt 2>&1
        echo "" >> /tmp/mw_api_reference.txt
    fi
done

# Inspect Label, Text, Box, Rect (we don't know which names are real)
for DRAW in "Label" "Text" "TextFigure" "Box" "Rect" "Rectangle" "Figure" "StudyHeader"; do
    MATCH=$(jar tf "$SDK_JAR" | grep "/${DRAW}.class" | head -1)
    if [ -n "$MATCH" ]; then
        FQN=$(echo "$MATCH" | sed 's/.class$//' | tr '/' '.')
        echo "=== $FQN ===" >> /tmp/mw_api_reference.txt
        javap -cp "$SDK_JAR" "$FQN" >> /tmp/mw_api_reference.txt 2>&1
        echo "" >> /tmp/mw_api_reference.txt
    fi
done

cat /tmp/mw_api_reference.txt
```

**READ ALL OF /tmp/mw_api_reference.txt** before proceeding. This is your ground truth.

## PHASE 2: CHECK JAVA VERSION

```bash
java -version
# If MotiveWave bundles its own JRE:
find /Applications/MotiveWave.app -name "java" -type f -exec {} -version \; 2>&1 | head -5
```

If MotiveWave uses Java 8-10, you MUST replace `var` with explicit types everywhere in the .java file. The `var` keyword was added in Java 10.

## PHASE 3: FIX BOSFVGStrategy.java

Now that you have the real API in /tmp/mw_api_reference.txt, go through `BOSFVGStrategy.java` and fix EVERY SDK API call.

### Known likely mismatches (check ALL of these):

| Line/Area | Current (guessed) | Might actually be |
|-----------|-------------------|-------------------|
| Imports | `com.motivewave.platform.sdk.common.*` | Check actual package paths from jar tf |
| Imports | `com.motivewave.platform.sdk.draw.*` | Might be different package |
| `@StudyHeader` fields | `namespace, id, name, desc, overlay, studyOverlay` | Fields may differ — check javap |
| `extends Study` | `Study` | Might need full path, might be different class |
| `createSD()` | Returns SettingsDescriptor | Method might not exist — check Study javap |
| `sd.addTab()` | Tab object | Check if this method exists on the real SD |
| `tab.addGroup()` | Group object | Check actual method name |
| `grp.addRow()` | Adding descriptors | Check actual method signature |
| `IntegerDescriptor` | Constructor: (key, label, default, min, max, step) | Constructor may differ |
| `DoubleDescriptor` | Same pattern | Constructor may differ |
| `DiscreteDescriptor` | Used for entry mode | Might not exist — use StringDescriptor or EnumDescriptor |
| `BooleanDescriptor` | Constructor: (key, label, default) | Check actual constructor |
| `setSettingsDescriptor(sd)` | Sets the SD | Method might not exist or differ |
| `calculate(int index, DataContext ctx)` | Override | Check actual signature on Study |
| `ctx.getDataSeries()` | Returns DataSeries | Method might differ |
| `series.getHigh(i)` | Direct accessor | Might be `series.getFloat(i, HIGH)` or `series.getDouble(i, HIGH)` |
| `series.getLow(i)`, `getClose(i)`, `getOpen(i)` | Same | Same potential issue |
| `series.getStartTime(i)` | Returns long (millis) | Method might differ |
| `series.size()` | Returns int | Might be `getCount()` or `getSize()` |
| `getSettings()` | Returns Settings | Check return type |
| `settings.getInteger()` | Named getter | Might be `get()` with cast, or different method |
| `settings.getDouble()` | Named getter | Same |
| `settings.getBoolean()` | Named getter | Same |
| `settings.getString()` | Named getter | Same |
| `new Coordinate(time, price)` | Constructor | Might be (price, time) or (series, index, price) |
| `new Line(coord1, coord2)` | Constructor | Might need setStart()/setEnd() |
| `line.setColor()` | Method | Check actual method |
| `line.setStroke()` | Sets dash pattern | Might not exist or differ |
| `new Marker(coord, type)` | Constructor | Check actual constructor |
| `marker.setFillColor()` | Method | Might not exist |
| `marker.setSize()` | Method | Might not exist |
| `Enums.MarkerType.TRIANGLE` | Enum | Check actual enum values |
| `Enums.MarkerType.SQUARE` | Enum | Check actual enum values |
| `Enums.Size.MEDIUM` | Enum | Check actual enum |
| `new Label(coord, text)` | Constructor | Might be `Text`, `TextFigure`, or different |
| `label.setFont()` | Method | Might not exist |
| `new Box(coord1, coord2)` | Constructor | Might be `Rect`, `Rectangle`, or not exist |
| `box.setFillColor()` | Method | Check actual method |
| `addFigure(figure)` | Method on Study | Might need `addFigure(Plot.PRICE, figure)` |
| `debug(text)` | Logging method | Might need `System.out.println()` or different |
| `var` keyword | Used throughout | Replace with explicit types if Java < 10 |

### What NOT to change (CRITICAL — these are correct):
- **Trading logic**: detectSwings(), detectBOS(), detectFVGs(), findSignals(), simulateTrade() — this is a precise port of clean_backtest.py. The MATH is correct. Only fix SDK API calls within these methods (like `series.getHigh(i)` → whatever the real method is).
- **Inner classes**: SwingPoint, BOSEvent, FVGEvent, Signal, TradeResult — pure Java, no SDK deps
- **getBarTimeMinUTC()** — standard Java Calendar API
- **logResults()** — standard Java (except possibly `debug()` call)
- **Algorithm flow in calculate()** — the order of operations is correct

### Approach:
1. Read `BOSFVGStrategy.java` top to bottom
2. For EVERY SDK call, check it against /tmp/mw_api_reference.txt
3. Fix each mismatch
4. If unsure about a class, run `javap -cp "$SDK_JAR" <fully.qualified.name>` to check
5. Pay special attention to constructors — wrong arg count/type = compile failure

## PHASE 4: COMPILE

```bash
cd ~/Documents/trading-system

# Build classpath with ALL jars in SDK lib directory
SDK_DIR=$(dirname "$SDK_JAR")
CP="$SDK_JAR"
for jar in "$SDK_DIR"/*.jar; do
    [ "$jar" != "$SDK_JAR" ] && CP="$CP:$jar"
done

# Create output directory
EXTENSIONS_DIR="$HOME/MotiveWave Extensions"
mkdir -p "$EXTENSIONS_DIR"

# Compile
javac -cp "$CP" -d "$EXTENSIONS_DIR" BOSFVGStrategy.java
```

**If javac fails:**
1. Read the EXACT error message — note the line number and error type
2. Look up the correct class/method using javap on the specific class
3. Fix the specific line in BOSFVGStrategy.java
4. Recompile
5. Repeat until 0 errors
6. Maximum 20 compile-fix cycles. If still failing after 20, log all remaining errors and stop.

**If javac says "source release X requires target release X":**
```bash
# Find what Java version MotiveWave uses
find /Applications/MotiveWave.app -name "java" -type f -exec {} -version \; 2>&1
# Compile with matching release:
javac --release 11 -cp "$CP" -d "$EXTENSIONS_DIR" BOSFVGStrategy.java
# Or: javac --release 8 ...
# Or: javac --release 17 ...
```

## PHASE 5: VERIFY

```bash
# Check class file exists
find "$HOME/MotiveWave Extensions" -name "*.class" -type f

# Verify class is valid
javap "$HOME/MotiveWave Extensions/com/willis/BOSFVGStrategy.class" | head -20

# Show class size
ls -lh "$HOME/MotiveWave Extensions/com/willis/BOSFVGStrategy.class"

echo ""
echo "SUCCESS — Restart MotiveWave and look for 'BOS FVG Trailing' under Study > Custom"
```

## PHASE 6: UPDATE BUILD SCRIPT

Update `~/Documents/trading-system/compile_mw_strategy.sh` with:
1. The ACTUAL SDK jar path you discovered (hardcode it as a fast path, keep the search as fallback)
2. The correct Java release flag if needed
3. Any classpath additions that were required

## ERROR PREVENTION RULES

### DO NOT:
- Create new Java files. Only modify BOSFVGStrategy.java.
- Change the trading logic (swing detection, BOS detection, FVG detection, signal matching, trade simulation).
- Invent SDK method names. EVERY method call must be verified with javap.
- Skip Phase 0-1. SDK discovery MUST happen before ANY Java editing.
- Use `var` keyword if Java version < 10.
- Ask questions. Just run.
- Give up after one compile failure. Fix and retry up to 20 times.

### DO:
- Log every step so progress is visible.
- If a compile error is about a missing class, search `jar tf "$SDK_JAR"` for similar names.
- If a compile error is about wrong method signature, run `javap` on that specific class.
- If an SDK class doesn't exist at all (e.g., `Box`, `Label`), search for alternatives and adapt.
- Save working javap output to /tmp/ for reference.
- After successful compile, verify the .class file with javap.

### SANITY CHECKS:
- Class file should be > 10KB (it's a big strategy with many methods)
- javap on the compiled class should show `BOSFVGStrategy extends <something>.Study`
- javap should show all methods: calculate, detectSwings, detectBOS, detectFVGs, findSignals, simulateTrade
- The package should be `com.willis`
- Inner classes should compile: BOSFVGStrategy$SwingPoint.class, etc.

## TIME ESTIMATE
- Phase 0 (SDK discovery): ~5 min
- Phase 1 (javap inspection): ~5 min
- Phase 2 (Java version): ~1 min
- Phase 3 (fix Java): ~15-30 min
- Phase 4 (compile-fix loop): ~10-30 min
- Phase 5-6 (verify + update script): ~5 min
- **Total: ~45-75 minutes**

## WHAT SUCCESS LOOKS LIKE

When you're done, Harrison should be able to:
1. Run `./compile_mw_strategy.sh` and get a clean build
2. See `~/MotiveWave Extensions/com/willis/BOSFVGStrategy.class` exists
3. Open MotiveWave → Study → Custom → "BOS FVG Trailing"
4. Apply it to an NQ 1-minute chart
5. See green entry arrows, red stops, teal trailing lines, P&L labels
6. Right-click → Settings to change all parameters (swing lookback, trail trigger, etc.)

START NOW. Phase 0 first. No questions.
