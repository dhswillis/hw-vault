# V13 Trade Types — Visual Guide

> All concepts tested on 255 days of NQ tick data (Feb 2025 — Feb 2026)
> Commission: 1pt RT + 0.5pt/side slippage | Trail: 0.3R trigger, 30% trail

---

## 1. OR_FAIL — Opening Range Failure (15 min)

**THE WINNER. 90.8% WR on shorts. Production model.**

The concept: Breakout traders push price through the Opening Range, but it fails and snaps back inside. Those trapped breakout traders become fuel for the reversal.

### SHORT Setup

```
Price
  ^
  |
  |         xxxxxxxxx
  |        x    2    x
  |       x  BREAK!   x
  |------x--------------x----- OR High ─────────────────
  |     x                 x         3. CLOSES BACK
  |    x  1. Opening Range  x           BELOW OR High
  |   x    (9:30-9:45 ET)    x              |
  |--x-------------------------x--------xxxx v
  |  x                         xxxxxxxx    xxxxx
  |  x                                        xxx  4. ENTER SHORT
  |----------------------------------------------x---- next bar open
  |                                               xx
  |                                                 xx ← trail locks profit
  |                                                  x
  |- - - - - - - - - - - - - - - - - - - - - - - - -x- OR Mid (TARGET)
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      9:30     9:45     10:00    10:15    10:30
      |←─ OR ─→|
```

**Rules:**
- Mark OR High, Low, Mid from 9:30-9:45 ET
- Price breaks ABOVE OR High
- Wait at least 2 minutes
- Price CLOSES back BELOW OR High
- Enter SHORT at open of NEXT 1-minute bar
- Stop: OR High + 5 points
- Target: OR Midpoint
- Trail: activates at +0.3R, locks in 70% of max move

### LONG Setup (with delta filter)

```
Price
  ^
  |
  |- - - - - - - - - - - - - - - - - - - - - - - - -x- OR Mid (TARGET)
  |                                                  x
  |                                                 xx ← trail locks profit
  |                                               xx
  |----------------------------------------------x---- next bar open
  |  x                                        xxx  4. ENTER LONG
  |  x                         xxxxxxxx    xxxxx
  |--x-------------------------x--------xxxx v
  |   x    1. Opening Range    x              |
  |    x   (9:30-9:45 ET)    x          3. CLOSES BACK
  |     x                  x              ABOVE OR Low
  |------x--------------x----- OR Low ──────────────────
  |       x  BREAK!   x
  |        x    2    x
  |         xxxxxxxxx
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      9:30     9:45     10:00    10:15    10:30
```

**Extra filter for longs:** Only take if OR delta < 0 (more selling than buying during the OR period). A bearish OR that breaks low and fails = stronger reversal.

**Result:** 109 shorts at 90.8% WR + 45 longs at 82.2% WR = 154 trades, 88.3% WR, +64.0R

---

## 2. IB_FAIL — Initial Balance Failure (30 min)

**Same concept as OR_FAIL but with a 30-minute window instead of 15.**

```
Price
  ^
  |
  |                  xxxxx
  |                 x  2  x
  |                x BREAK x
  |──────────────x──────────x──── IB High ────────────
  |             x             x        3. CLOSES BACK
  |            x               x            BELOW
  |           x  1. Initial     x              |
  |          x   Balance         x             v
  |         x   (9:30-10:00)     xxxxxxxxxxxxxxx
  |        x                                    xxxx
  |       x                                        xx  → SHORT ENTRY
  |──────x──────────────────────────────────────────── IB Low
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      9:30        10:00       10:30       11:00
      |←──── IB (30 min) ────→|
```

**Why it works alongside OR_FAIL:**
- Different reference levels (30-min range is usually wider than 15-min)
- Fires on different days or at different prices
- 112 UNIQUE trades that don't overlap with OR_FAIL
- 87.5% WR, +40.5R on the unique trades alone

**Result:** 174 total trades, 86.8% WR, +54.2R (112 unique after dedup)

---

## 3. SESSION_EXTREME — PM Session Reversal

**Catches reversals at the day's high/low during the afternoon session.**

### SHORT Setup (new session high, weak close)

```
Price
  ^
  |
  |                              xxxxxx  ← NEW SESSION HIGH
  |                             x      x    (bar makes highest
  |                            x        x    price of the day)
  |                           x    but   x
  |                          x   closes   x
  |                         x   in LOWER   x
  |         AM Session     x    40% of bar  x  → SHORT ENTRY
  |     xxxx              x                  xxxxxxx
  |    x    xxxxx        x                         xxx
  |   x          xxx    x                             xx
  |  x              xxxx                                xx
  | x                                                     x
  |x                                                       x
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      9:30     10:00    10:30    11:00    11:30    12:00
                                 |←── PM Session (signals only after 11AM) ──→|
```

**Key difference from OR/IB_FAIL:**
- Fires in the AFTERNOON (after 11:00 AM ET) — different time window
- Uses SESSION high/low, not Opening Range
- Reversal signal: price makes new extreme but can't hold it
- Short: bar makes session high but closes in bottom 40%
- Long: bar makes session low but closes in top 60%
- Uncorrelated with OR_FAIL (correlation = -0.081)

**Result:** 259 trades, 81.1% WR, +32.1R. Longs actually better (83.5%) than shorts (79.2%).

---

## 4. VWAP_BOUNCE — VWAP Cross & Reversal

**Price dips below VWAP, then closes back above = long signal (and vice versa).**

```
Price
  ^
  |
  |     x                     x
  |    x x    VWAP           x x
  |   x   x  ──────────────x───x──────────  ← VWAP line
  |  x     x  ───────────x──    x
  |         x  ────────x──       x
  |          x  ─────x──    3. CLOSES      x  → LONG ENTRY
  |           x ───x──       BACK ABOVE      x
  |            xx──  2. DIPS    VWAP          x
  |             x     BELOW                    x
  |                   VWAP
  |
  |---x---x---x---x---x---x---x---x---x---x--> Time
      9:45     10:00    10:15    10:30
      |←── AM Session only (first 90 min) ──→|
```

**Rules:**
- Only fires in AM session (first 90 minutes)
- Price was above VWAP, dips below, then closes back above → LONG
- Price was below VWAP, pushes above, then closes back below → SHORT
- Stop: recent swing extreme + 5pt buffer
- Target: 1.5R from entry

**Result:** 430 trades, 76.5% WR, +33.6R. Walkforward PASS but lower WR than OR/IB concepts. Shorts (78.9%) better than longs (74.2%).

---

## 5. MICRO_SWEEP — 5-Minute Swing Sweep & Reversal

**A 5-minute candle sweeps a prior swing high/low, then closes back inside.**

```
Price
  ^
  |
  |        x           SWING HIGH
  |       x x         ─────────────────────────────
  |      x   x       |
  |     x     x     x|                     xxxxx
  |    x       x   x ||                   x  |  x
  |   x         x x  ||   SWEEP!         x   |   x
  |  x           x   |x  (wick goes     x    |    x
  |                   | x  above swing) x     |     x
  |                   |  x             x  CLOSES     x → SHORT
  |                   |   xxxx     xxxx   BACK        x   ENTRY
  |                   |       xxxxx       BELOW        x
  |                   |                   SWING         x
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      ← 5-min candles →
```

**LOOK-AHEAD BUG FOUND (V13o audit):** Original results used entry on a 1M bar WITHIN the 5M bar period (entering 3-4 minutes before the 5M bar closed). After fixing entry to wait for the 5M bar to close:

**Corrected results:**
- Raw: 9,227 trades, 78.9% WR, +541.4R (was 88.3% — inflated by look-ahead)
- First per dir: 510 trades, 78.6% WR, +51.0R (was 90.8%)
- Signal WR matches random entry + trail (~79%)

**Result:** DEAD CONCEPT. No signal edge beyond the trail mechanic. The 79% WR is exactly what random entries produce with the 0.3R/30% trail. The original 90% WR was entirely from look-ahead bias.

---

## 6. DRIVE_FADE — Opening Drive Fade

**Strong initial push (>20pts in first 15 min) that fades back.**

```
Price
  ^
  |
  |            x x x  ← Strong UP drive
  |           x      x   (+20pt in 15min)
  |          x        x
  |         x          x
  |        x            x
  |       x              x
  |      x  OPEN          x
  |     x                  x
  |    x                    x
  |                          x  FADE begins
  |                           x
  |                            x
  |                             x  Price drops below
  |                              x  OR midpoint
  |      OPEN price               x → SHORT ENTRY
  |     ─────────────────────────── (target: back to open)
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
      9:30     9:45     10:00    10:15
```

**Rules:**
- Opening drive must be >20 points in first 15 minutes
- UP drive: wait for price to close below OR midpoint → SHORT (target = opening price)
- DOWN drive: wait for price to close above OR midpoint → LONG (target = opening price)

**Result:** DEAD CONCEPT. Only 53 trades, 73.6% WR, -2.8R total. Shorts barely positive (+1.4R), longs negative (-4.2R). Not enough edge to overcome commissions.

---

## 7. OR Timeframe Variants (V13k)

**Same OR_FAIL concept tested with different Opening Range windows:**

```
                    OR Window Comparison

  9:30          9:40     9:45     9:50     9:55    10:00
  |──────────────|────────|────────|────────|────────|
  |← 10 min OR →|        |        |        |        |
  |←──── 15 min OR ──────→|        |        |        |
  |←──────── 20 min OR ───────────→|        |        |
  |←──────────── 25 min OR ────────────────→|        |
  |←──────────────── 30 min OR (IB) ────────────────→|
```

| Window | Trades | WR | R | Calmar |
|--------|--------|-----|---|--------|
| 10 min | 188 | 83.0% | +74.3R | **27.6** |
| **15 min** | **157** | **86.0%** | **+57.9R** | **21.5** |
| 20 min | 137 | 86.1% | +53.8R | 20.0 |
| 25 min | 121 | 85.1% | +33.9R | 10.6 |
| 30 min (IB) | 106 | 84.9% | +23.5R | 10.6 |

**Key finding:** Shorter ORs = more trades, slightly lower WR. 15 min is the sweet spot for WR. 10 min produces more R overall due to more signals.

---

## 8. RE-ENTRY — Multiple Failures Per Day (V13k/V13n)

**If price breaks the OR, fails, we trade... then breaks AGAIN and fails AGAIN.**

```
Price
  ^
  |
  |    1st BREAK        2nd BREAK         3rd BREAK
  |      & FAIL           & FAIL            & FAIL
  |       xxx              xxx               xxx
  |      x   x            x   x            x   x
  |─────x─────x──────────x─────x──────────x─────x──── OR High
  |    x       x        x       x        x       x
  |   x    →    xxxx   x    →    xxxx   x    →    xxxx
  |  SHORT #1       xxx SHORT #2     xxx SHORT #3     xxx
  |  86.0% WR           79.2% WR         85.2% WR
  |
  |──────────────────────────────────────────────────── OR Mid
  |
  |---x---x---x---x---x---x---x---x---x---x---x---x--> Time
```

**Surprising finding:** 3rd entries (85.2% WR) outperform 2nd entries (79.2%)! By the 3rd failure, trapped breakout traders are EXTREMELY committed and the reversal is stronger.

| Re-entry | Trades | WR | R/Year |
|----------|--------|-----|--------|
| 1x only | 157 | 86.0% | +57.9R |
| Up to 2x | 287 | 82.9% | +78.4R |
| Up to 3x | 395 | 83.5% | +109.9R |

---

## The Trail Mechanic — Universal Edge Engine

**Applied to ALL concepts. This is what makes everything work.**

```
Price (SHORT example)
  ^
  |
  |── STOP (OR High + 5pt) ────────────────────────────
  |
  |── ENTRY ──────────────x
  |                        x
  |                         x
  |   +0.3R reached ─ ─ ─ ─ x─ ─ ─  TRAIL ACTIVATES
  |                           x
  |         Trail stop moves    x     ← max favorable
  |         down with price      x       excursion (MFE)
  |    ┌─── trail stop ──────────-x
  |    │    (MFE × 70%)            x
  |    │                            x  ← price reverses
  |    │                           x
  |    └──── TRAIL STOP HIT ──── x  ← EXIT with profit
  |
  |── TARGET (OR Mid) ────────────────────────────────
  |
  |---x---x---x---x---x---x---x---x---x---x---x--> Time
```

**How it works:**
1. Trade enters. Stop is at original level.
2. Price moves in your favor. When it reaches +0.3R, trail ACTIVATES.
3. Trail stop = entry + (MFE × 70%). It only moves in your favor, never back.
4. If price reverses and hits the trail → exit with a partial win.
5. If price reaches the target → exit at target (full win).

**Why it's so powerful:**
- P(reaching +0.3R before -1R) ≈ 77% just from random walk math
- Average trail exit = +0.62R (locking in more than half the move)
- 88% of all trades exit via trail (only 12% are full losses)
- 0% of trades end between -0.5R and 0R — it's either profit or full stop

---

## Final Scoreboard

```
  CONCEPT             TRADES    WR      TOTAL R    STATUS
  ──────────────────────────────────────────────────────────
  OR_FAIL (15min)       157    86.0%    +57.9R     PRODUCTION  ★★★
  OR_FAIL 3x re-entry   395    83.5%   +109.9R     PRODUCTION  ★★★
  IB_FAIL (30min)       174    86.8%    +54.2R     PRODUCTION  ★★★
  SESSION_EXTREME       259    81.1%    +32.1R     PRODUCTION  ★★
  VWAP_BOUNCE           430    76.5%    +33.6R     VIABLE      ★
  OR_10min              188    83.0%    +74.3R     VALIDATED   ★★
  MICRO_SWEEP           510    78.6%    +51.0R     DEAD (=random+trail) ✗
  DRIVE_FADE             53    73.6%     -2.8R     DEAD        ✗
  ──────────────────────────────────────────────────────────

  NOTE: MICRO_SWEEP original results (90.6% WR) had look-ahead bias.
  Corrected entry (wait for 5M bar close) drops WR to 78.6% = random + trail.

  BEST PORTFOLIO (OR_FAIL 3x + IB_FAIL + SESSION_EXTREME):
  758 trades | 82.7% WR | +175.5R | 29.6 Calmar | 94.4% weekly WR
  ALL months positive | Walkforward PASS | Max 2 consecutive losses
```
