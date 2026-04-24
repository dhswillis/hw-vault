---
date: 2026-04-15
status: decided
deciders: [harrison]
project: tempo-trading-system
tags: [decision, trading, tempo, wick-fade, demotion]
links:
  - wiki/summaries/tempo-portfolio-v26.md
  - wiki/summaries/ifvg-composite-audit-20260329.md
---

# Wick Fade: VALIDATED → UNPROVEN

## Context

Wick Fade had been tagged `VALIDATED` and was on the promotion shortlist based on a stack of favorable numbers:

- 15s wick fade: 43% WR, **$30.89/trade**, tick-validated on Databento.
- 5m wick fade: **+77 to +94 R/day**, Calmar 2.12, **10/10 walk-forward folds positive** — textbook anti-overfit evidence.

Then the flip variant — the second leg of the two-sided strategy, which the portfolio math assumed as equal contributor — was swept across **216 configurations** and returned **0 profitable**. Not "marginal," not "needs tuning" — zero. Re-decomposing the two-legged portfolio showed that **~68% of the claimed edge came from the flip leg** in the original single-config paper run. The fade-only leg alone is a thin +$8–12/trade strategy, not a headline number.

## Options considered

### Option 1: Keep `VALIDATED`, promote the fade leg alone to LIVE

**Pros:**
- Fade-only leg is still positive in backtest.
- Walk-forward on the 5m variant was genuinely 10/10 — that's not a statistical fluke.

**Cons:**
- The labeled "Wick Fade" number the org has been tracking is **not** the fade-leg-alone number. Promoting under the existing label is a silent downgrade of what the strategy means.
- Fade-alone edge is comparable to noise floor on NQ execution costs; promoting it commits review bandwidth to a low-information strategy.

### Option 2: Demote to `UNPROVEN`, rebuild from scratch

**Pros:**
- Forces re-characterization of what "Wick Fade" actually is (fade-only, no flip).
- Clears the promotion shortlist of a strategy whose top-line number is not reproducible.
- Preserves the real finding: 5m walk-forward held, which is still worth understanding, just not under the old label.

**Cons:**
- Discards ~4 weeks of cumulative framing work.
- Risk of over-correcting — the fade leg is positive; declaring the whole thing UNPROVEN may be too strong.

## Decision

Demote Wick Fade from `VALIDATED` to `UNPROVEN`. It never went live, so there's no live position to unwind — this is a label/roadmap change only. Re-open investigation of the **fade-only 5m variant** as a separate research item with its own name and its own honest numbers.

## Why

The headline number (+77 to +94 R/day, Calmar 2.12) was a two-leg result where one leg is now known to be dead. Keeping the `VALIDATED` tag while the top-line number is not reproducible is exactly the failure mode the vault's dedup/lint discipline is designed to prevent. The 10/10 walk-forward is real but applies to a two-leg strategy that no longer exists. The right move is to decouple: kill the composite label, preserve the valid sub-finding (fade-only positive), and re-characterize it on its own merits.

This is also a process win worth logging — the flip sweep (216 configs, 0 profitable) is the kind of result that would be easy to bury. The explicit demotion sets the precedent that **being wrong and saying so is a promotion-quality behavior**.

## Consequences

**Enables:**
- Honest portfolio accounting. The +19.8 pts/day V15 V2 number (see [[2026-04-14-lumi-promotion]]) is now the real promotion candidate, uncontaminated by a Wick-Fade number that wasn't going to materialize.
- Fade-only 5m variant gets investigated on its own terms — could still earn promotion later under a new label if walk-forward and tick-validation hold for the leg in isolation.

**Closes off:**
- Any automation or UI that keys off the `VALIDATED` tag for Wick Fade needs to be touched — check `composite_strategy.py`, any dashboards, and the portfolio v26 roster.
- Morally closes off the pattern of promoting two-legged strategies on composite numbers without sweeping each leg independently first. The flip sweep should be done **before** a composite is tagged, not after.

## Review trigger

- **Revisit** if a fade-only 5m sweep across ≥100 configs shows ≥70% profitable with Calmar ≥1.5 and median +$15/trade. At that point, the fade-only strategy earns its own `PAPER` tag under a new name.
- **Do not revisit** the two-leg Wick Fade under the old label regardless of new findings — the flip leg is considered definitively dead (0/216 is not a tuning problem).
