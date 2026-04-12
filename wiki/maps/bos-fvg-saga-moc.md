---
created: 2026-04-11
updated: 2026-04-11
type: moc
topic: BOS_FVG Saga
tags: [moc, bos-fvg, trading, dead-strategy, case-study]
related:
  - wiki/maps/tempo-moc.md
  - wiki/maps/audit-history-moc.md
  - wiki/concepts/invalidation-rules.md
---

# BOS_FVG Saga — Map of Content

> The full case study of how a signal went from "the best in the dataset" to DEAD — and the three bugs that inflated it at different stages. A cautionary tale for future strategy research.

## Orientation

BOS_FVG (Break of Structure + Fair Value Gap) was Harrison's mining-era interpretation of Tempo's IFVG. At its peak, it was reported at **97.9% WR** (V1 era), then **63.5% WR / +1.354 R/T** (V2 "corrected"), then **+1.141 avgR / Calmar 263.6** (V10z "conservative"). As of 2026-04-10, the tick-level ground truth is **+0.001 avgR** — essentially zero. It was never a real signal. **Three different bugs** inflated it at different stages, each discovered by a different audit.

Read this MOC as a case study in how backtest contamination compounds. Each audit caught one bug and appeared to "clean" the result — but one is not enough. The real lesson is [[invalidation-rules|Rule 12]]: if a tick-level audit shows zero, no bar-level result can rehabilitate it.

## The three contamination layers

### Layer 1 — V10i Look-Ahead Alignment (Feb 2026)

- [[wiki/concepts/v10i-look-ahead-bug]] — The `get_alignment()` function used unclosed HTF bars. A 5M bar's `close` field at 10:02 is actually the 10:05 close — 3 minutes of future data.
- **Effect:** Inflated alignment counts. A 91.1% WR config on 2,035 trades collapsed to 37.6% on 340 trades after the fix. 0/13 green months.
- **Caught by:** [[wiki/summaries/look-ahead-audit|LOOK_AHEAD_AUDIT_2026-02-18]]
- **V2 mining "fixed" this.** But it didn't fix the other two layers.

### Layer 2 — Minimum Risk Floor + idx Offset (Feb 2026)

- V1 had no minimum risk floor. 53% of `body_stop` trades had < 3pt risk → artificial 10R+ winners.
- V1 simulation started checking targets from the fill bar (`idx`), not the next bar (`idx+1`). The fill bar's H/L already happened before the close-entry.
- **Effect:** `fvg_exit_body_stop` went from +15,623R (V1) to +502R (V2) — 1/30th.
- **Caught by:** [[wiki/summaries/comprehensive-mining-report-v2|V2 mining rerun]]
- **V2 fixed both.** But BOS_FVG's headline (+1.354 R/T) was still inflated by Layer 3.

### Layer 3 — Bar-Sim Trailing Bug (Apr 2026)

- [[wiki/concepts/bar-sim-trailing-bug]] — Trailing stops on OHLC bars systematically inflate performance because the path reconstruction guesses "low-then-high" when the real tick sequence is often "high-then-low."
- **Effect:** +0.29 R/trade inflation on BOS_FVG. Bar-sim: +0.359 avgR. **Tick: +0.001 avgR.**
- **Caught by:** [[wiki/summaries/bos-fvg-failure-consolidated|BOS_FVG_FAILURE_CONSOLIDATED (2026-04-10)]]
- **This is the one that killed it.** Survived both earlier fixes because it's not a bug in the *code* — it's a structural limitation of simulating a trail on OHLC.

## The timeline

```
Feb 2025    V1 mining begins (312d dataset, 8,358 trades)
Feb 2026    V1 report: BOS_FVG 97.9% WR, "Tier S sniper"
Feb 2026    V10i look-ahead audit → alignment was fake → V2 mining begins
Feb 2026    V2 report: BOS_FVG 63.5% WR, +1.354 R/T, "core signal of dataset"
Mar 2026    V10z "conservative" sim → BOS_FVG +1.141 avgR, Cal 263.6
Mar 2026    NQ Playbook written around BOS_FVG as the flagship
Mar 2026    BOS_FVG 10pt tick audit → static T3R = +0.015 avgR (FIRST WARNING)
Mar 2026    WickFade findings doc FLAGS bar-sim trailing as a known problem
Apr 2026    Paired bar-vs-tick trailing replay → +0.001 avgR (GROUND TRUTH)
Apr 2026    CLAUDE.md updated: "BOS_FVG DOES NOT WORK" (retroactive to March)
Apr 2026    HW vault wiki: bos-fvg.md rewritten as dead-strategy
```

## The key documents (reading order)

1. **[[wiki/summaries/strategy-mining-report-312d]]** — The V1 "97.9% WR sniper tier" claim. Start here to see what was originally claimed. Tagged `suspect-results`.
2. **[[wiki/summaries/comprehensive-mining-report-v2]]** — The V2 "correction" that fixed look-ahead but still ran bar-sim. The "63.5% WR is the honest number" claim — which turned out to also be dishonest. Tagged `suspect-results`.
3. **[[wiki/summaries/nq-playbook]]** — The V10z flagship that claimed +1.141 avgR as the "validated" number. Written in March, dead by April. Tagged `suspect-results, superseded`.
4. **[[wiki/summaries/bos-fvg-10pt-audit]]** — The March tick-level audit that **should have been** the end. +0.015 avgR baseline, no filter surviving Bonferroni. This was the first warning that was not propagated urgently enough.
5. **[[wiki/summaries/bos-fvg-failure-consolidated]]** — The April canonical failure report. The paired bar-vs-tick experiment that produced the definitive +0.001 avgR ground truth. **Read this as the authoritative BOS_FVG position.**

## The concepts that made the illusion possible

- [[wiki/concepts/be-trail-mechanism]] — The exit mechanism whose "edge" was the bar-sim illusion. BE+Trail converts losers to breakevens and lets winners run — which looks great on bar-sim because the sim over-credits the runners.
- [[wiki/concepts/mtf-alignment]] — The filter that V10i inflated. 81% WR at 6 TFs aligned looked like monotonic edge — it was monotonic look-ahead.
- [[wiki/concepts/bar-sim-trailing-bug]] — The structural problem. Not a code bug (the intra-bar re-check only catches 0.5% of trades). The real issue is path reconstruction itself.

## The syntheses that connected the dots

- [[wiki/syntheses/bos-fvg-claim-vs-reality]] — Why "BOS_FVG is the core signal of Tempo" was a two-layer mistake: (1) BOS_FVG is not the same signal as Tempo's IFVG, and (2) its backtest numbers were bar-sim artifacts.
- [[wiki/syntheses/mining-reports-v1-v2-reconciliation]] — The V1 → V2 correction. V2 fixed alignment look-ahead but not bar-sim. Now known to be insufficient — both V1 and V2 numbers are dead.
- [[wiki/syntheses/tempo-three-layers]] — The framework that explains why Tempo's real 88% WR and Harrison's mining 63.5% WR are not contradictions — they're measuring different things.

## What BOS_FVG is NOT

BOS_FVG is NOT Tempo's [[ifvg|IFVG]]. The two signals share DNA (FVGs + liquidity structure), but:

| | BOS_FVG | IFVG |
|---|---|---|
| FVG timing | After BOS confirmation | Before sweep (momentum gap) |
| Entry | Limit at FVG far edge | Close through pre-existing FVG at FVG edge |
| Stop | Fixed R with BE+Trail | Opposite FVG edge (soft stop, close-through) |
| Bar-sim safe? | **No** (trail) | **Yes** (close-through soft stop is bar-safe) |
| Status | **DEAD** | Active — v24 on sim |

This distinction matters because IFVG's close-through soft stop is bar-sim-safe by construction ([[invalidation-rules|Rule 1 exemption]]), while BOS_FVG's trailing stop is not.

## The rule(s) it produced

- [[invalidation-rules|Rule 1]] — Do not trail on bars
- [[invalidation-rules|Rule 6]] — WF pass ≠ edge
- [[invalidation-rules|Rule 12]] — Tick supersedes bar
- [[invalidation-rules|Rule 14]] — NautilusTrader cross-check warning sign

## Related MOCs

- [[maps/tempo-moc]] — the broader Tempo research arc
- [[maps/audit-history-moc]] — every audit sorted by outcome
