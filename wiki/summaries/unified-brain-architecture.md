---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
related:
  - wiki/entities/nova-sable-brains.md
  - wiki/entities/tempo-trading-system.md
  - wiki/concepts/bayesian-belief-engine.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/summaries/tempo-cluster.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, tempo, architecture, unified-brain, stub]
---

# Unified Brain Architecture — Summary

> **Stub.** The authoritative source is `~/HW/raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx`, a binary file not yet converted to markdown. The summary below is assembled from cross-references in [[tempo-cluster]] and Harrison's `~/Documents/trading-system/CLAUDE.md`. Expand after running `textutil -convert txt -output /tmp/unified-brain.txt` on the docx.

## What the source is

`The_Unified_Brain_Architecture.docx` (2026-04) — the most recent authoritative planning document in the Tempo cluster. It replaces three earlier planning docs: *The Partnership*, *AI Trading Show Blueprint*, and *AIO System Blueprint*. The doc captures a significant pivot from "keep mining, keep optimizing" to "accept that backtesting-as-truth has failed and build a different kind of system."

## The honesty disclosure (front of doc)

The doc opens with a blunt statement of the current project position:

> "NOTHING from the backtesting research (V7–V11) is validated for live trading. BOS_FVG does not work. V7/V8 was killed by look-ahead bias. The V10t clean results are approximately breakeven after commissions. The research journal documents what was TESTED, not what WORKS. v24 (IFVG MTF cascade) is the ONLY strategy currently on NinjaTrader sim. It has not yet proven itself."

This disclosure is the canonical project-side correction and is the reason [[bos-fvg]], [[nq-playbook]], and [[research-journal]] are all tagged suspect in the vault. If these pages ever disagree with the Unified Brain doc, the Unified Brain doc wins.

## Architecture components

- **Two AI researcher personas** — [[nova-sable-brains|Nova and Sable]], running on **DeepSeek R1-Distill-Qwen-14B via Ollama** (local, no cloud dependency)
- **Persistent memory** — **Mem0** (semantic layer) + **SQLite** (structured layer) + **weekly LoRA fine-tuning** on session data
- **Risk Officer** — hard-coded Python, no AI, no flexibility. Non-negotiable daily/weekly drawdown limits, account lock triggers, position sizing rules
- **Weekly tournament** — Nova and Sable submit research + recommendations weekly, judged by **external AI advisor APIs** (Claude, Grok, ChatGPT). Winner's approach gets increased weight in the next week's capital allocation
- **Phase progression** — strict gating: research-only → paper → micro live (1 MNQ) → scale. Cannot skip phases
- **Live YouTube show pipeline** — ElevenLabs voice synthesis + BocaLive avatars + FFmpeg compositor for a weekly/daily AI-driven market recap show
- **Monthly cost** — $220–650 depending on API usage tier

## Anti-stupidity rules (baked into Mem0 seed)

Rules hard-coded into the initial memory so Nova and Sable cannot re-discover already-known dead ends:

- **NEVER** test multi-TF candle alignment ([[v10i-look-ahead-bug|look-ahead trap]])
- **NEVER** test break-even stops ([[be-trail-mechanism|dead across 11 configs]])
- **ALWAYS** walk-forward validate (4 quarters positive OOS)
- **ALWAYS** apply realistic costs (0.5pt slippage + 0.5pt commission per side)

## Dead strategies (baked in)

Listed explicitly as "do not re-test":

- london_breakout
- VP_POC_retest
- fib_retracement
- failed_breakout
- sweep_reversal
- double_BOS_momentum
- VWAP_mean_reversion

These correspond to the strategies called out as dead across every parameter set in the [[nq-playbook|NQ Playbook]] and [[research-journal|Research Journal]] dead-ends lists. Baking them into the seed prevents Nova and Sable from wasting cycles re-deriving "these don't work."

## What's new vs the old approach

Old approach (V7 → V11):
- Mine the tick data for edges
- Find filter combinations that survive walk-forward
- Declare validated, ship to NinjaTrader, trade live
- **Outcome:** every wave of results was invalidated by a subsequent bug discovery (V10o alignment, phantom B/E, bar-sim trailing)

New approach (Unified Brain):
- Don't mine for edges — accept that tick-data mining on this dataset is exhausted
- Instead, have two AI researchers reason about *market regime and methodology*
- Use a disciplined phase progression with hard-coded risk guardrails that don't depend on the backtest being correct
- Use external AI judging to catch mistakes that local reasoning misses
- Learn from live paper data, not from historical backtests

The shift from "backtest then trade" to "reason, gate, and learn from forward data" is the core insight of this doc. It's a direct response to the pattern in [[tempo-three-layers]] — Layer 2 (mining) has produced successive waves of false positives, so the project is now betting on Layer 1 methodology (Tempo's canonical rules) + Layer 3 execution discipline (forward phase gating) + AI-driven synthesis rather than continued Layer 2 mining.

## Open questions (stub markers)

- Exact Nova vs Sable division of labor — is one researcher more risk-averse? More methodology-focused? More regime-focused?
- How the weekly tournament judging actually works — what's the rubric, how is "winning" measured?
- Is Phase 1 (research-only) currently active, or has it progressed to paper?
- Concrete relationship between Nova/Sable and the existing [[bayesian-belief-engine|Context Engine]]
- Is v24 IFVG MTF cascade part of the Unified Brain phase progression, or a separate track?

## Action needed

1. Convert the docx with `textutil -convert txt -output /tmp/unified-brain.txt ~/HW/raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx`
2. Re-read the text and expand this summary with direct quotes and architecture diagrams
3. Update [[nova-sable-brains]] with the divisional details

## Cross-references

- [[nova-sable-brains]] — entity page for the two AI researchers
- [[tempo-trading-system]] — the Mac-side directory where this architecture is being built
- [[bayesian-belief-engine]] — the Context Engine that may or may not integrate with Nova/Sable
- [[tempo-three-layers]] — the framework that explains *why* the Unified Brain pivot makes sense
- [[tempo-cluster]] — master cluster summary (references this doc extensively)
- `~/Documents/trading-system/CLAUDE.md` — the corresponding project-side directive
