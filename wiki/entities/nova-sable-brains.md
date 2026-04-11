---
created: 2026-04-11
updated: 2026-04-11
type: entity
sources:
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/tempo-cluster.md
  - wiki/concepts/bayesian-belief-engine.md
tags: [trading, tempo, ai, architecture, stub]
---

# Nova and Sable — the Unified Brain Researchers

> **Stub.** This page is a cross-link target for [[tempo-cluster]]. The authoritative source is `raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx`, which is a binary file and has not yet been fully ingested. Expand this page after the docx is converted.

## What they are (from the [[tempo-cluster|Tempo cluster summary]])

**Nova** and **Sable** are the two AI researcher personas in Harrison's "Unified Brain" architecture — the post-pivot project state documented in `The_Unified_Brain_Architecture.docx` (2026-04). They are local model instances running **DeepSeek R1-Distill-Qwen-14B via Ollama**.

## Architecture (high level)

The Unified Brain replaces three earlier planning docs (The Partnership, AI Trading Show Blueprint, AIO System Blueprint). Key components:

- **Two AI researchers (Nova, Sable)** — local DeepSeek R1-Distill-Qwen-14B instances
- **Persistent memory** — Mem0 (semantic) + SQLite (structured) + weekly LoRA fine-tuning
- **Risk Officer** — hard-coded Python, no AI, no flexibility, non-negotiable limits
- **Weekly tournament** — external AI advisor judging (Claude / Grok / ChatGPT APIs)
- **Phase progression** — research-only → paper → micro live (1 MNQ) → scale
- **Live YouTube show pipeline** — ElevenLabs + BocaLive avatars + FFmpeg compositor
- **Monthly cost target** — $220–650

## Why this replaces prior planning

The Unified Brain doc opens with a blunt honesty disclosure: "NOTHING from the backtesting research (V7–V11) is validated for live trading. BOS_FVG does not work. V7/V8 was killed by look-ahead bias. The V10t clean results are approximately breakeven after commissions. The research journal documents what was TESTED, not what WORKS. v24 (IFVG MTF cascade) is the ONLY strategy currently on NinjaTrader sim. It has not yet proven itself."

Nova and Sable exist because the prior approach — "keep mining, keep optimizing, trust the backtest" — has been explicitly abandoned as a dead-end. The researchers are meant to reason about *market state and methodology*, not re-run more backtests on suspect data.

## Anti-stupidity rules (baked into Mem0 seed)

- **NEVER** test multi-TF candle alignment (look-ahead trap — see [[v10i-look-ahead-bug]])
- **NEVER** test break-even stops (dead across 11 configs — see [[be-trail-mechanism]])
- **ALWAYS** walk-forward validate (4 quarters positive OOS)
- **ALWAYS** apply realistic costs (0.5pt slippage + 0.5pt commission per side)
- **Dead strategies baked in:** london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion

## Open questions (stub markers)

- Exact prompt templates for Nova vs Sable — what's the division of labor?
- How does the weekly tournament scoring actually work? What does "winning" mean?
- How does the Risk Officer interlock with Nova/Sable's proposed trades?
- Is there live-sim integration yet, or is Phase 1 still research-only?
- What's the relationship between Nova/Sable and the existing [[bayesian-belief-engine|Context Engine]]?

Expand this page after converting `The_Unified_Brain_Architecture.docx` to markdown via `textutil`.
