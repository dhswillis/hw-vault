# Automated Futures Trading System - Master Knowledge Base

**Version:** 0.1.0
**Last Updated:** 2026-02-08
**Purpose:** This is the system prompt for the API Wrapper. Every Claude API call gets this entire document as context. Claude Code maintains it — it grows as strategies are extracted.

---

## System Overview

This system trades ES/NQ futures on prop firm accounts. Goal: 3R daily returns through 6-10 smaller trades (0.25-0.5% risk each). Anti-fragile by design — no single trade can blow up the day.

---

## Brain Definitions

### Brain #1: Tempo Strategy

**Source:** Tempo (Discord)
**Status:** AWAITING EXTRACTION — no strategy documents loaded yet.

**What we know so far (from handoff doc):**
- FVG on dynamic entry TF (5s-5m per vol/tick regime)
- Candle closes through FVG = "valid"
- Close sweeps 15m fractal, session H/L, or 5m/15m FVG fill
- Inverse entry (fade the sweep)
- Has a checklist + indicator from the educator

**What we still need to extract:**
- [ ] FVG Rules: minimum size, body or wick, "clean" criteria
- [ ] Validation Rules: close through = any part or fully through?
- [ ] Sweep Targets: 15m fractal definition (how many bars), which sessions
- [ ] Entry: exact placement (FVG retest, immediate), limit or market
- [ ] Stop: below sweep, below FVG, fixed ticks, ATR-based?
- [ ] Target: fixed R, opposing FVG, opposing liquidity?
- [ ] The Indicator: what platform, is source code visible, does he sell it?
- [ ] Checklist details: what are the "no trade" conditions beyond news?

**Next step:** Load Tempo educator materials (videos, Discord exports, PDFs) and extract rules.

### Brain #2: Lanto Strategy
**Status:** NOT STARTED — to be extracted

### Brain #3: Carmine Strategy
**Status:** NOT STARTED — to be extracted

### Brain #4: Omega Strategy
**Status:** NOT STARTED — to be extracted

### Brain #5: Flipping Markets Strategy
**Status:** NOT STARTED — to be extracted

---

## Context Engine Variables

### Hard Filters (Binary: Trade or Don't)

| Filter | Threshold | Action |
|--------|-----------|--------|
| High-impact news | Within 15 minutes | No trades |
| VIX | > 35 (crisis) | No trades |
| Time | Before 8:30 AM or after 3:00 PM ET | No new positions |
| GEX extreme bearish | DIX < 0.35 | No longs |
| Price beyond expected range | Beyond SpotGamma 1-Day Expected Range | Mean reversion only |

### Soft Filters (Position Size Multipliers)

| Condition | Multiplier |
|-----------|------------|
| DIX bullish (>0.47) + long setup | 1.25x |
| Price near SpotGamma Call/Put Wall | 1.15x |
| Dark pool cluster aligns with S/D zone | 1.2x |
| CVD divergence at zone | 1.15x |
| GEX positive + inside day | 1.1x |

### Core Variables (P0 - Must Have)

| Variable | Type | Source |
|----------|------|--------|
| VIX Level + Regime | Volatility | CBOE/Yahoo |
| Economic Calendar | Event | ForexFactory |
| SPX GEX Level | Dealer Flow | SpotGamma |
| Gamma Flip Point | Dealer Flow | SpotGamma |
| Session Ranges (Globex/Asia/London/IB) | Structure | Quantower/Calc |

### P1 Variables (Should Have)

| Variable | Type | Source |
|----------|------|--------|
| TICK/ADD/VOLD | Internals | Quantower |
| News Sentiment | Sentiment | Benzinga/NewsAPI |
| Day Classification | Structure | Calculated |
| DIX (Dark Index) | Institutional | SqueezeMetrics |

### P2 Variables (Nice to Have)

| Variable | Type | Source |
|----------|------|--------|
| Vanna/Charm Direction | Greeks Flow | Calculated |
| CVD / Delta Divergence | Order Flow | Quantower |

---

## Dynamic Entry Timeframe Selection

| Regime | Entry TF | Which Levels Matter | FVG Size |
|--------|----------|---------------------|----------|
| Low Vol / Low Tick | 3m - 5m | 1m FVGs, 1m fractals, small session ranges | Smaller gaps valid |
| Medium Vol | 1m - 2m | 5m FVGs, 15m fractals, session H/L | Standard gaps |
| High Vol / High Tick | 5s - 30s | 15m FVGs, session H/L, major liquidity pools | Only large gaps |

**Note:** Volume/tick thresholds for regime classification TBD during extraction.

---

## Day Fingerprinting

Each trading day gets a fingerprint based on: day of week, FOMC proximity, CPI/NFP proximity, OPEX proximity, VIX regime, GEX regime, gap size, IB range vs ATR, historical distribution for that day type. System adjusts brain parameters per fingerprint.

---

## Risk Management

- **Risk per trade:** 0.25-0.5% of account equity
- **Daily target:** 3R through 6-10 trades
- **Daily loss limit:** 5% equity = hard stop
- **Equity drawdown limit:** 20% from peak = all trading stops
- **Consecutive losses > 3:** reduce position size 50%

---

## Signal Format

All brains output signals as JSON:

```json
{
  "brain": "TempoStrategy",
  "instrument": "ES",
  "direction": "long",
  "entry": 5002.75,
  "stop": 4998.00,
  "target": 5010.50,
  "size": 2,
  "risk_r": 2.15,
  "timestamp": "2026-02-08T14:30:22Z",
  "context_snapshot": {},
  "confidence": 0.82
}
```

---

## Signal Flow

Brain identifies setup → calls API Wrapper for validation → writes signal.json → Quantower C# FileWatcher picks up signal → Quantower executes order → Replikanto copies to prop accounts → fill.json written → OpenClaw logs trade.

---

*This document is the single source of truth. It will grow as strategies are extracted from educator materials.*
