---
created: 2026-04-11
updated: 2026-04-11
type: concept
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/dol-framework.md
  - wiki/concepts/bpr.md
  - wiki/summaries/tempo-rules-v3.md
tags: [trading, tempo, risk-management, canonical]
---

# Setup Quality Grading (A+/A/B+/B/C)

The canonical [[tempo-methodology|Tempo]] risk-management gate: before taking any trade, count the confluences present and assign a quality tier. Tier determines whether you trade, at what size, and whether the trade is eligible for funded accounts.

## The tiers

From `tempo_rules_v3_complete.md`:

| Tier | Confluences | Reported WR | Position action |
|---|---|---|---|
| **A+** | 7+ | 80%+ | Full size, funded eligible |
| **A** | 5–6 | 70–75% | Full size, funded eligible |
| **B+** | 3–4 | 50–70% | Half size, funded eligible (just above minimum) |
| **B** | 2 | ~50% | Half size, funded *not* recommended |
| **C / Skip** | 0–1 | Break-even or worse | **Do not trade** |

## The January 2026 rule change

Prior to January 2026, Tempo's minimum to trade was **2 confluences**. In January 2026 the minimum was raised to **3+ confluences on funded accounts**. `tempo_rules_v3_complete.md` frames this as a discipline tightening after scaling to 30+ funded accounts — a trade that fires once executes 30× simultaneously, so false positives at the B tier became too expensive at scale.

Practically: if you're trading a personal account, B (2 confluences) is still on the table. If you're trading funded / copy-trading, you only take B+ and above.

## What counts as a confluence

The confluence factors, tallied per trade:

1. **Major sweep** — liquidity taken at a significant DOL level (London H/L, PDH/PDL, session H/L)
2. **Sweep age ≥ 1 hour** — the swept level was formed at least an hour before the sweep (fresh sweeps less reliable)
3. **[[smt|SMT divergence]]** — NQ/ES intermarket disagreement on the sweep (primary since Nov 2025)
4. **Singular [[ifvg|IFVG]]** — the reversal closes back through a single FVG, not a stacked cluster
5. **IFVG size 8–15pt** — optimal zone (5–6pt too tight, >30pt too wide)
6. **Daily bias alignment** — trade direction matches the higher-TF bias (4H → 1H cascade)
7. **50% level rejection with wick** — price wicked through 50% of the daily range and rejected
8. **Clean momentum through the gap** — death candle (3+ points) OR 3+ consecutive directional candles
9. **[[bpr|BPR]] support** — entry sits at or near a balanced price range
10. **Session level alignment** — entry aligns with London H/L, NY open level, or other session anchor
11. **Multiple HTF confluences** — 4H and 1H both agree (e.g. both have unfilled FVGs in the trade direction)
12. **[[dol-framework|DOL]] ahead** — unswept structural liquidity in the trade direction = room to run

## The primary filter stack

Tempo's core "80% WR" formula is stated repeatedly in the rules doc:

**SMT + IFVG + liquidity sweep = 80%+ WR**

Everything else above is layered on top. Without all three of those present, you're at B tier or worse and either sizing down or skipping. Harrison's IFVG research (`TEMPO_IFVG_RESEARCH.md`) confirmed the pattern empirically — baseline 64.4% WR jumps to 83% WR only when SMT is paired with reaction-quality (death candle or 3+ consecutive candles), which maps to "sweep + IFVG + SMT + momentum" at the A tier.

## Why "raise the minimum" instead of "trade more setups"

An unusual discipline choice for a retail-style methodology: in Jan 2026 Tempo responded to growing the account count by *reducing* the number of eligible setups, not increasing. The logic is expected-value based — at 2 confluences (~50% WR), the payoff has to be asymmetric enough to cover losing trades, and at scale (30+ accounts per fire) the drawdowns from B-tier losses compound faster than the wins from marginal B+ tier trades.

This is also why [[be-trail-mechanism|break-even at +17pts]] is non-negotiable for Tempo even though it kills edge in Harrison's mining research: at 30× position count, a B/E save is a fleet save, not a trade save.

## Interaction with session rules

Setup grading is independent of session. An A+ London setup still takes the "100% runner to B/E at +17" treatment; an A+ NY AM setup still takes the 30%/70% trim. Grading determines *whether* you trade; session rules determine *how* you manage it. A B+ setup in a good session (NY AM) is preferable to an A setup in a bad session (NY PM) — tier and session are multiplicative, not additive.
