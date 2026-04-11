# QUICK REFERENCE GUIDE - Trading Rules at a Glance

## Decision Tree: Should You Trade This Signal?

```
START: You see a signal setup
│
├─> Is alignment CONFIRMING?
│   ├─ YES ──┐
│   │        └─> Continue to pattern check
│   │
│   └─ NO (neutral/mixed/contradicting)
│       ├─ Is it a FULL SIZE combo? (see list below)
│       │  ├─ YES → Trade but use HALF SIZE
│       │  └─ NO → SKIP or use QUARTER SIZE max
│       │
│       └─ If contradicting → NEVER trade (save 1.6R per signal)
│
└─> Check pattern direction match
    ├─ Pattern direction = Trade direction? (bull pattern + long, etc.)
    │  ├─ YES → Add 0.5 confidence point
    │  └─ NO → Reduce to quarter size or SKIP
    │
    └─> Check session
        ├─ NY AM / IB First / London AM?
        │  └─> Add 0.5 confidence
        │
        ├─ Power Hour / Overlap?
        │  └─> Add 0.25 confidence
        │
        └─> London Open / News Window?
            └─> Neutral, no bonus/penalty

        Final Confidence Score ≥ 2.0 → FULL SIZE
        Final Confidence Score 1.5-2.0 → HALF SIZE
        Final Confidence Score 1.0-1.5 → SMALL SIZE (25%)
        Final Confidence Score < 1.0 → SKIP
```

---

## Golden Rules (Copy to Your Trading Screen)

1. **CONFIRMING alignment adds +1.9R vs contradicting**
   - Confirming: 1.86R (BOS), 1.38R (sweep), 0.94R (fade)
   - Contradicting: -0.05R, -0.66R, -0.47R
   - Never trade contradicting alignment

2. **Pattern direction must match trade direction**
   - Bull patterns only on LONG trades
   - Bear patterns only on SHORT trades
   - Wrong direction = -1.0R average (deadly)

3. **Session + Signal combinations matter**
   - NY AM + BOS_FVG: 1.399 avg_R ✓
   - London AM + VA_fade + confirming: 1.361 avg_R ✓
   - London Open + BOS_FVG: 0.921 avg_R (weak)
   - News Window + signals: volatile but high avg_R

4. **Friday is weak for entries**
   - BOS_FVG on Friday: 0.643 avg_R (vs Mon 1.313 avg_R)
   - Only enter Friday if confirming + proven combo

5. **1-Minute patterns beat 5-Minute patterns**
   - doji_dragonfly (1M) + BOS_FVG: 67.4% WR, 2.687 avg_R
   - vs generic_bear (5M) + BOS_FVG: 32.5% WR, 0.885 avg_R

---

## The 31 FULL SIZE Combos (Trade Without Hesitation)

### Tier 1: Calmar >100
- strong_close_bear + SHORT direction (373 trades, 57%, 2.48 avg_R, **122 calmar**)
- strong_close_bull + LONG direction (396 trades, 57%, 2.23 avg_R, **101 calmar**)

### Tier 2: Calmar 60-100
- BOS_FVG + confirming (902 trades, 40%, 1.86 avg_R, **97 calmar**)
- sweep_reversal + confirming (587 trades, 58%, 1.38 avg_R, **74 calmar**)
- london_am + confirming (605 trades, 50%, 1.42 avg_R, **68 calmar**)

### Tier 3: Calmar 30-60
- sweep_reversal + Fri + confirming (117 trades, 60%, 1.60 avg_R, **51 calmar**)
- marubozu_bull + LONG (108 trades, 65%, 2.36 avg_R, **51 calmar**)
- sweep_reversal + Mon + confirming (106 trades, 62%, 1.47 avg_R, **43 calmar**)
- BOS_FVG + doji_gravestone (1M) (45 trades, 56%, 3.96 avg_R, **34 calmar**)
- VA_fade + london_am + confirming (322 trades, 51%, 1.36 avg_R, **34 calmar**)

Plus 21 more in calmar 10-30 range (see full document)

---

## The 193 TOXIC Combos (NEVER TRADE THESE)

### Category 1: Contradicting Alignment (DEADLY)
- ANY signal + contradicting alignment = -0.35R average, 14% WR
- VA_fade + contradicting: -0.465 avg_R
- sweep_reversal + contradicting: -0.663 avg_R
- Saves 1.6R per signal by avoiding these

### Category 2: Pattern-Direction Conflict
- Bull pattern + SHORT direction: -0.94R to -0.99R
- Bear pattern + LONG direction: -0.81R to -1.06R
- Example: strong_close_bull + short = 5.8% WR, -0.943 avg_R (n=207)

### Category 3: Session-Pattern Conflict
- Marubozu patterns + IB First = 0% win rate (kills 1.18R)
- VA_fade + IB First + contradicting = 2.9% WR (kills 1.07R)
- Long pattern + NY AM (specific combos) = -0.97R

### Category 4: Low Win Rate Killers (wr <25%)
- Inverted hammer + SHORT: 8% WR
- Long upper shadow + TUE: 7.1% WR
- Any pattern with confluence levels that don't work

---

## Signal-by-Signal Cheat Sheet

### BOS_FVG (Breakout of Structure)
**Base rate:** 31.9% WR, 1.066 avg_R
**With confirming:** 39.7% WR, **1.864 avg_R** ← Use confirming only

**Best combos:**
- ib_first + confirming: 2.639 avg_R ✓ FULL SIZE
- ny_am + confirming: 1.923 avg_R ✓ FULL SIZE
- doji_gravestone (1M): 3.955 avg_R ✓ FULL SIZE
- doji_dragonfly (1M): 2.687 avg_R ✓ FULL SIZE

**Worst combos:**
- contradicting alignment: -0.046 avg_R ✗ AVOID
- Friday: 0.643 avg_R (half baseline)
- London AM without confirming: 0.757 avg_R

---

### Sweep Reversal (Risk Management)
**Base rate:** 35.1% WR, 0.327 avg_R
**With confirming:** 58.4% WR, **1.379 avg_R** ← 3x better

**Best combos:**
- power_hour + confirming: 1.277 avg_R, 64% WR ✓ FULL SIZE
- london_open + confirming: 1.319 avg_R, 62% WR ✓ FULL SIZE
- sweeping reversal generally: 58.4% WR when confirming

**Worst combos:**
- contradicting: 12.9% WR, -0.663 avg_R ✗ DEADLY
- london_am + contradicting: 8.3% WR, -0.904 avg_R ✗ DEADLY
- Wednesday general: 32.7% WR, 0.209 avg_R

---

### VA Fade (Volume Profile)
**Base rate:** 31.7% WR, 0.398 avg_R
**With confirming:** 42.6% WR, **0.938 avg_R** ← Double the baseline

**Best combos:**
- london_am + confirming: 50.6% WR, 1.361 avg_R ✓ FULL SIZE
- overlap + confirming: 46.7% WR, 1.969 avg_R ✓ FULL SIZE
- thursday general: 36.7% WR, 0.725 avg_R

**Worst combos:**
- contradicting: 17% WR, -0.465 avg_R ✗ AVOID
- Monday: 31.7% WR, 0.121 avg_R (nearly worthless)

---

### Pre-Gap-Fill (Early Session Setup)
**Base rate:** 23.6% WR, 0.435 avg_R
**Recommendation:** Use only confirming, small sample size (81 trades total)
**Avoid:** contradicting alignment (5.4% WR, -0.232 avg_R)

---

## Session Rankings (By Average Signal Performance)

| Session | Avg WR | Avg avg_R | Best For | Notes |
|---------|--------|-----------|----------|-------|
| **NY AM** | 34.9% | 1.020 | All signals | Most consistent |
| **IB First** | 29.3% | 0.592 | BOS_FVG | Only with BOS_FVG + confirming |
| **Overlap** | 30.6% | 0.789 | VA_fade confirming | Best for overlap combos |
| **Power Hour** | 40.8% | 0.770 | Sweep reversal | High WR session |
| **News Window** | ? | 0.73 | All signals | High variance, small n |
| **London AM** | 26.0% | 0.323 | VA_fade confirming | Huge sample size (605) |
| **London Open** | 28.4% | 0.361 | General trading | Lower quality overall |
| **Silver Bullet** | ? | 0.57 | Pattern combos | Afternoon session |

---

## Day-of-Week Strength

| Day | BOS_FVG avg_R | Sweep avg_R | VA_fade avg_R | Overall |
|-----|---|---|---|---|
| **Monday** | 1.313 | 0.339 | 0.121 | ✓ Good for BOS_FVG |
| **Tuesday** | 0.898 | 0.383 | 0.312 | ⚠ Weak |
| **Wednesday** | 1.211 | **0.209** | 0.362 | ⚠ Avoid sweep |
| **Thursday** | 1.165 | **0.538** | **0.725** | ✓ Good overall |
| **Friday** | **0.643** | 0.522 | 0.415 | ⚠ Weak for BOS_FVG |

**Rule:** Avoid Friday entries unless confirming + known edge. Wednesday is bad for sweep.

---

## Position Sizing Decision Matrix

```
┌─────────────────────────────────────────────────────────────┐
│ POSITION SIZE DECISION                                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Confirming align + Pattern match + Good session + High WR? │
│ ────────────────────────────────────────────────────────── │
│   YES → FULL SIZE (100% normal risk)                       │
│   NO → Continue below                                      │
│                                                             │
│ Confirming align + Decent session + Positive avg_R?       │
│ ────────────────────────────────────────────────────────── │
│   YES → HALF SIZE (50% normal risk)                        │
│   NO → Continue below                                      │
│                                                             │
│ One positive factor + Positive avg_R > 0.3?               │
│ ────────────────────────────────────────────────────────── │
│   YES → SMALL SIZE (25% normal risk)                       │
│   NO → Continue below                                      │
│                                                             │
│ Any of the 193 toxic combos?                              │
│ ────────────────────────────────────────────────────────── │
│   YES → SKIP (Don't trade)                                │
│   NO → Evaluate unknown combo                             │
│                                                             │
│ Unknown combo with positive avg_R but low WR (<30%)?      │
│ ────────────────────────────────────────────────────────── │
│   YES → PAPER TRADE ONLY (validate before real money)     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Scoring System (Use This Before Every Trade)

Score each factor 0-1:

1. **Alignment** (most important)
   - Confirming: +1.0
   - Mixed/Neutral: +0.3
   - Contradicting: -1.0 (autofail)

2. **Pattern Quality**
   - 1M pattern (doji, hammer, etc.): +1.0
   - 5M strong pattern (strong_close, marubozu): +0.8
   - Generic/weak pattern: +0.3

3. **Pattern-Direction Match**
   - Perfect match (bull pattern + long): +1.0
   - No match or conflicting: -0.8 (reduce trade)

4. **Session**
   - NY AM / IB First / London AM: +0.5
   - Power Hour / Overlap: +0.3
   - London Open: 0.0
   - Known bad combo: -0.3

5. **Day of Week**
   - Mon-Thu: +0.2
   - Friday: -0.2 (penalty)
   - Weekend: SKIP

**TOTAL SCORE:**
- **5.0+** → FULL SIZE
- **4.0-4.9** → HALF SIZE
- **3.0-3.9** → SMALL SIZE (25%)
- **Below 3.0** → SKIP or paper trade
- **Any -1.0** component → Auto-reduce by 1 size category

---

## The Numbers That Matter (Memorize These)

| Metric | Gold Standard | Good | Acceptable | Avoid |
|--------|---|---|---|---|
| **Win Rate** | >60% | 50-60% | 40-50% | <25% |
| **avg_R** | >2.0 | 1.0-2.0 | 0.5-1.0 | <0 |
| **Calmar Ratio** | >50 | 20-50 | 10-20 | <5 |
| **Sample Size** | >100 | 50-100 | 20-50 | <10 |
| **Confirming WR boost** | +800% | +500% | +300% | → use anyway |
| **Alignment impact** | 1.9R swing | (standard) | (standard) | deadly |

---

## Common Mistakes (Don't Make These)

1. ❌ Trading contradicting alignment "just this once"
   - Costs you 1.6R per trade

2. ❌ Using bull pattern on short trades
   - Will hit -0.8R to -1.0R stops repeatedly

3. ❌ Forcing a Friday BOS_FVG without confirming
   - Friday base rate is 0.643 avg_R (nearly worthless)

4. ❌ Trading VA_fade on Monday without extra confirmation
   - Monday VA_fade is 0.121 avg_R (essentially flat)

5. ❌ Ignoring session quality
   - London Open costs you 50% of profitability vs NY AM

6. ❌ Taking full size on unknown combos
   - Test with quarter size first

7. ❌ Trading sweep_reversal on Wednesday + contradicting
   - That's a -0.768 avg_R trade (paper only)

8. ❌ Trading marubozu patterns in IB First
   - Historical 0% win rate (2 trades went 0-2)

---

## Pre-Market Checklist (Print This)

Before the market opens:

- [ ] Check calendar for major news (impacts confirming alignment)
- [ ] Identify today's day-of-week (Mon=good for BOS, Wed=bad for sweep)
- [ ] Plan which signals you'll trade (not all days are equal)
- [ ] Remember: Confirming alignment only (save energy for the best setups)
- [ ] Watch NY AM for BOS_FVG (highest avg_R in dataset)
- [ ] Watch London AM for VA_fade + confirming (huge sample, 1.36 avg_R)
- [ ] If Friday: Only trade known FULL SIZE combos
- [ ] Check your position sizing rules (print the matrix above)

---

## Emergency Exit Rules

If you're in a trade and one of these happens:

1. **Alignment flips to contradicting** → Exit immediately
   - You're now in a -1.6R swing disadvantage

2. **Pattern-direction match breaks** → Exit immediately
   - This was a core reason for the trade

3. **Session changes** (e.g., NY AM ends, London starts) → Reassess
   - Rules change per session; may need to reduce size

4. **Friday afternoon + no confirming** → Close for day
   - Weak session, weak confirmation = don't fight

5. **New conflicting pattern forms** → Consider exit
   - New information may invalidate original thesis

---

## Acknowledgments

Data source: V7aa_b_full_matrix.json
- 2,600+ individual trade combinations analyzed
- 17 cross-tabulated datasets
- 15,000+ total trades in backtest
- Covered 4 signal types, 8 sessions, 5 DOW, 4 alignment states

Key statistics:
- 31 FULL SIZE edges identified (calmar >10)
- 57 HALF SIZE edges identified
- 193 TOXIC combos identified and ruled out
- Single biggest factor: Confirming alignment (+1.9R swing)
- Most reliable edge: strong_close_bear directional trade (122 calmar, 57% WR)
