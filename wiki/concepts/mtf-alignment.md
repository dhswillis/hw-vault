---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
  - raw-sources/STRATEGY_MINING_REPORT.md
related:
  - wiki/concepts/bos-fvg.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/comprehensive-mining-report-v2.md
tags: [trading, filter, mtf, nq]
---

# MTF (Multi-Timeframe) Alignment

MTF alignment is the **most important filter** in the [[tempo-trading-system|Tempo]] signal stack. Both the V1-era [[strategy-mining-report-312d|312d report]] and the cleaned [[comprehensive-mining-report-v2|V2 report]] agree on this, even after the V2 cleanup removed the biggest source of look-ahead bias.

## What it measures

For each higher timeframe (1M, 3M, 5M, 10M, 15M, 1H, 4H), does the timeframe's directional context agree with the trade direction on the 1M signal bar? V2 uses a 3-bar HH/HL trend pattern on **completed bars only** — this is the clean formulation.

> **Do NOT use the V1 single-bar `close > open` formulation** — that's exactly what the [[v10i-look-ahead-bug|V10i look-ahead bug]] consulted, and it pulls in unclosed higher-TF bars' future close values. Any research that used it is contaminated.

## V2 numbers (clean, 7-timeframe stack)

Perfect monotonic WR improvement:

| TFs aligned | n | BE+Trail WR | R/T | Total R |
|---|---|---|---|---|
| 0 | 7,628 | 40.8% | -0.09 | -694 |
| 1 | 7,504 | 48.2% | +0.26 | +1,949 |
| 2 | 6,143 | 54.5% | +0.68 | +4,194 |
| 3 | 5,159 | 61.2% | +1.18 | +6,102 |
| 4 | 4,028 | 68.1% | +1.80 | +7,251 |
| 5 | 2,757 | 74.0% | +2.38 | +6,550 |
| 6 | 1,104 | 81.0% | +3.24 | +3,577 |

Every additional TF aligned improves WR and R/T. No inflection point.

## V1 MTF_5M gate claim (corroborated, magnitude suspect)

The [[strategy-mining-report-312d|V1 312d report]] found:
- Without `mtf_5m_aligned`: 46.5% WR, -0.28 R/T, -2,378R (catastrophic)
- With `mtf_5m_aligned`: 70.8% WR, +0.28 R/T, +446R

The direction is correct — the MTF_5M gate is load-bearing. But the specific magnitude may be inflated (the same "with MTF" numbers also hit 97.9% WR in the report's TIER_A — which V2 later showed was look-ahead).

## How to use it

- **As a hard filter:** require at least `al≥2` or `al≥3` for low-volume / high-conviction setups.
- **As a scaling lever:** increase position size as alignment count increases. 6-TF aligned is 3x the R/T of 2-TF aligned.
- **For combo filtering with [[bos-fvg|BOS_FVG]]:** V2 shows `al≥2 co≤2` (alignment ≥2, contra ≤2) as a common sweet spot for sweep-family signals at 5.0R fixed targets.

## Why it works (hypothesis)

Higher timeframes represent larger participants' directional commitment. When multiple TFs agree, the probability of running into a counter-flow wall in the next 1-5 minutes drops sharply. When TFs conflict, you're fading at least one larger player — and fat-tail runners (the source of [[be-trail-mechanism|BE+Trail]] edge) can't materialize.

## Clean vs dirty alignment

| Version | Formula | Status |
|---|---|---|
| V1 / V10i | Single bar: `tf_c.close > tf_c.open` on bar where `tf_c.index <= candles_1m.index[entry_idx]` | **BROKEN** — picks unclosed current HTF bar |
| V2 | 3-bar HH/HL trend pattern on completed bars only | CLEAN |

Fix: `mask = tf_c.index + TF_DURATION[tf_label] <= entry_time` — only use bars that have fully closed before the entry timestamp.

## Also referenced in

- [[raw-sources/imports/2026-04-11/Documents/README_EXCEL_ANALYSIS|README EXCEL ANALYSIS]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/01-wick-fade/docs/WickFade_Strategy_Spec|WickFade Strategy Spec]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/BACKTEST_ISSUES_LOG|BACKTEST ISSUES LOG]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/03-sf-portfolio/docs/CONTEXT|CONTEXT]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/CLAUDE|CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/MINING_SIGNAL_SPEC|MINING SIGNAL SPEC]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/08-keylevel-sweep/keylevel_sweep_fade/COWORK_PROMPT|COWORK PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/tempo/scripts/AUDIT_PROMPT|AUDIT PROMPT]]
