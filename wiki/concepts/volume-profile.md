---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/Tempo_Context_Engine_Spec.docx
related:
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/concepts/session-type-taxonomy.md
  - wiki/concepts/bayesian-belief-engine.md
tags: [trading, volume-profile, footprint, context-engine]
---

# Volume Profile

The volume profile is a core intelligence layer in the [[tempo-context-engine-spec|Tempo Context Engine]] — the Layer 2 / Layer 3 feature set that separates Tempo from pure price-action systems. It tells the engine **where** participation is concentrated and **who** is aggressive at each level.

## Construction

Divide the price range into bins (0.25pt for NQ = 1 tick) and sum volume at each level:

```python
def build_volume_profile(trades, tick_size=0.25):
    profile = defaultdict(lambda: {'total': 0, 'buy': 0, 'sell': 0})
    for trade in trades:
        level = round(trade.price / tick_size) * tick_size
        profile[level]['total'] += trade.size
        if trade.aggressor == 'buy':
            profile[level]['buy'] += trade.size
        else:
            profile[level]['sell'] += trade.size
    return profile
```

Requires trade-level data with aggressor side — from [[databento|Databento]] or an equivalent tick feed.

## Key metrics

### Point of Control (POC)
The price level with the highest total volume in the session. Represents fair value / acceptance. When POC migrates directionally through a session, it confirms trend. When POC stays flat, it confirms range.

### Value Area (VA)
The price range containing 70% of total volume. `VA High` and `VA Low` act as dynamic support/resistance. Narrow VA = compression; wide VA = acceptance of broad range.

### High Volume Nodes (HVN)
Price levels with significantly above-average volume. They act as **magnets** — price tends to return to HVNs. Trades near HVNs have higher fill probability but lower R potential.

### Low Volume Nodes (LVN)
Price levels with significantly below-average volume. They act as **air pockets** — price moves quickly through LVNs. Breakouts through LVNs tend to accelerate.

## POC migration tracking

The Context Engine tracks POC position at every checkpoint and computes migration direction and strength:

```python
def migration_direction(poc_history):
    if len(poc_history) < 2: return 'NEUTRAL'
    delta = poc_history[-1] - poc_history[-2]
    if delta > 1.0:  return 'UP'
    if delta < -1.0: return 'DOWN'
    return 'STABLE'
```

POC migration is the primary feature for distinguishing TREND from RANGE at the `OPEN_60MIN` checkpoint in the [[session-type-taxonomy|session type taxonomy]].

## Footprint analysis (Layer 3)

When volume is split by aggressor side you can see **who** is moving price.

- **Delta at level** — buy volume minus sell volume at a specific price. Positive delta at a support level = buyers defending. Negative delta at resistance = sellers pressing. Delta divergence from price = reversal signal.
- **Imbalance stacks** — 3+ consecutive price levels with a buy:sell ratio exceeding 3:1 (or the inverse). These mark footprint imbalance zones and often precede continuation moves.
- **Absorption** — high volume at a price level with minimal price movement. Large resting orders are absorbing aggressive flow. Absorption at highs = potential reversal. Absorption at lows = potential support.
- **Exhaustion** — volume spike combined with delta extreme combined with price stalling. The aggressive side has run out of fuel. High-probability reversal signal.

## Feature outputs consumed by the classifier

| Feature | Computation | Predicts |
|---|---|---|
| `session_poc` | Highest-volume level in current session | Fair value / acceptance zone |
| `poc_migration` | POC shift session-to-session, direction + magnitude | Trend strength confirmation |
| `poc_distance` | Current price - session POC, in ATR units | Mean reversion vs breakout |
| `value_area_width` | (VA High - VA Low) / day ATR | Acceptance range width |
| `hvn_proximity` | Distance to nearest HVN | Rejection vs acceptance |
| `lvn_proximity` | Distance to nearest LVN | Breakout acceleration zone |
| `relative_volume` | Current / average at this time of day | Participation level |
| `volume_at_extremes` | Volume within 2pt of session high/low / total | Edge conviction |
| `prior_day_poc_level` | Exact POC from yesterday | Key reference today |
| `weekly_poc_level` | Rolling 5-day POC | Macro fair value |

These feed into the [[bayesian-belief-engine|Bayesian belief update]] at each checkpoint.

## Build phase

Volume profile integration is Phase 2 of the Context Engine build (Week 2-3). Requires purchasing Databento tick data for a training window (~$50-150 for Jul-Sep 2025). Before shipping, the team validates that adding Layer 2 features improves classification accuracy over Layer 1 baseline. If it doesn't improve accuracy, don't ship it — measure first.
