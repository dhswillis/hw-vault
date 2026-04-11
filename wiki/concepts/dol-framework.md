---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/bpr.md
  - wiki/concepts/setup-quality-grading.md
  - wiki/summaries/tempo-rules-v3.md
tags: [trading, tempo, structure, canonical]
---

# DOL Framework — Draw on Liquidity

**DOL** = "Draw on Liquidity." Where price is heading next — the structural magnet that it wants to reach. In the canonical [[tempo-methodology|Tempo]] framework, DOL is the answer to "where is this trade going," sitting opposite [[ifvg|IFVG]] (which answers "where do I enter") and [[smt]] (which answers "why is this reversing").

## The hierarchy

`tempo_rules_v3_complete.md` orders liquidity draws from strongest to weakest. An A+ trade is one where a single entry services multiple aligned draws:

1. **London session high/low** — the most powerful draw, especially when swept during NY
2. **Data wick high/low** — high-impact news wick (CPI, NFP, FOMC). Extremely strong draw, targets commonly 50–100pt
3. **Hourly high/low** (most recent 1H swing)
4. **50% of the daily range** (midline)
5. **Previous day's high/low** (PDH/PDL)
6. **Weekly high/low**
7. **Trend line liquidity** (diagonal structure)
8. **4H Fair Value Gap** (unfilled)
9. **[[bpr|BPR]]** (balanced price range)

## Relation to IFVG entries

The IFVG signal is an entry mechanism; DOL is the target. The strongest setups stack both:

> "Combine the fact that we're tapping into this gap AND tapping off the 50% level = play justified."
> — paraphrased from tempo_rules_v3, lines 349–369

When multiple draws align in the same direction (e.g. PDL below + London low below + hourly low below, all unswept), the DOL is unambiguous and the trade has structural "room to run." When draws conflict or are already swept, targets become uncertain and setup quality downgrades.

## The "all targets already swept" failure mode

One of Tempo's explicit avoid-patterns: **when overnight / pre-market sweeps have already taken all the obvious DOL levels before the NY session opens, there is no edge.** The liquidity has already been collected; any entry points into structural emptiness. This is specifically called out for all-time-high regimes — "At ATH, there is NO sell-side liquidity above. Shorting is extremely dangerous."

The practical rule: before entering, identify the DOL. If you can't name the liquidity level the trade is drawing toward, don't take the trade.

## How DOL interacts with session targets

DOL is about structure, not about the session-specific TP rules from [[ifvg]]. A London trade with TP rules "100% runner to +17pts B/E" still has to have a DOL — the runner is the vehicle for reaching the DOL, not a substitute for it. If the DOL is only +8pts away, the session rules force you to over-target and give back the move at B/E. This is part of why London underperforms on IFVG in backtests (see [[tempo-v14-corrections]]): the session rules don't adapt to how much room there actually is to the DOL.
