---
created: 2026-04-11
updated: 2026-04-11
type: entity
sources:
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/unified-brain-architecture.md
  - wiki/summaries/tempo-cluster.md
  - wiki/concepts/bayesian-belief-engine.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
tags: [trading, tempo, ai, architecture, canonical]
---

# Nova and Sable — the Unified Brain Researchers

**Nova** and **Sable** are the two AI researcher personas in [[unified-brain-architecture|The Unified Brain Architecture]], the post-pivot project state for Harrison's NQ trading system. They are local model instances running **DeepSeek R1-Distill-Qwen-14B via Ollama**. They do not yet exist as running software — this entity page documents the spec from the 2026-04 architecture doc.

## Role (not execution, but research)

The previous version of the project plan assumed the brain would execute `v24` as a known-good strategy. The v2 architecture (April 2026) explicitly rejects that framing:

> "The brain's job is not to execute a proven strategy. There is no proven strategy. The brain's job is to FIND one."

Nova and Sable are **researchers first**, executors second. They inherit the full research history (every dead end, every lesson, every correction) and their job is to propose, test, and evaluate new hypotheses without repeating known failures.

## Personality split

Nova and Sable are intentionally asymmetric. They use the same base model but train on different reward weightings, creating a diversified "portfolio of research styles":

| Dimension | **Nova (Partner A)** | **Sable (Partner B)** |
|---|---|---|
| **Archetype** | Methodical researcher | Aggressive researcher |
| **Preference** | High WR, small trades, strong statistical backing | High R-multiple, willing to accept lower WR for bigger payoffs |
| **Research style** | Systematic, full parameter sweeps, demands walk-forward validation + conservative simulation | Intuitive, structural patterns, willing to trade smaller samples if logic is sound |
| **LoRA reward** | **Conservative:** 2× penalty on losses, 1× on wins | **Aggressive:** 2× bonus on big wins, 1× penalty on losses |
| **Show persona** | The analyst — viewers trust her for education | The wild card — viewers watch her for entertainment |
| **Voice (ElevenLabs)** | Clear, professional female | Energetic, expressive female |

**Memories are NOT shared between them.** Each maintains independent Mem0 + SQLite stores. They learn different lessons from the same data, which is what makes the weekly tournament meaningful rather than a rubber-stamp of one model's output.

## Base model and infrastructure

- **Model:** DeepSeek R1-Distill-Qwen-14B
- **Runtime:** Ollama on GPU VPS (RTX 3090, ~$80–150/month)
- **Memory (semantic):** [Mem0](https://mem0.ai/) — vector-based memory that stores experiences as embeddings and supports similarity search
- **Memory (structured):** SQLite — trade logs, research logs, corrections
- **Fine-tuning:** Weekly LoRA runs on [RunPod](https://runpod.io/) A100, 2–4 hours per brain, ~$5–15 per run
- **Voice synthesis:** ElevenLabs Flash v2.5 (75ms first-byte latency)
- **Avatars:** BocaLive (trained face models with lip-sync)
- **Video compositor:** FFmpeg

## The research loop (both brains run this same loop)

1. **OBSERVE** — ingest latest market data, review recent trade outcomes
2. **RECALL** — query Mem0 for related patterns and past experiments
3. **HYPOTHESIZE** — propose a testable idea (new signal, filter, parameter)
4. **CHECK DEAD ENDS** — cross-reference against known failures; abort if redundant
5. **DESIGN** — specify backtest parameters, sample period, success criteria
6. **TEST** — run via `clean_backtest.py` on tick data, walk-forward validate
7. **EVALUATE** — conservative sim, slippage stress, commission drag
8. **RECORD** — log to Mem0 + SQLite (win or lose)
9. **SHARE** — post findings to show dialogue + Discord, explain what was learned
10. **REPEAT**

**Implementation blocker (2026-04-10 discovery):** the research loop's step 6 references `clean_backtest.py`. That file has the [[bar-sim-trailing-bug|bar-sim trailing path-reconstruction bug]] — any trailing-stop hypothesis tested through it will produce inflated results. Before Phase 1 research begins, either:
- `clean_backtest.py` must be replaced with a tick-level simulator, OR
- The brains' anti-stupidity rules must be extended to disallow trailing stops as a hypothesis form, OR
- All trailing-stop hypotheses must pass through a tick-level cross-validation step

The Unified Brain doc was written before this bug was discovered. The Mem0 seed should be updated accordingly.

## Anti-stupidity rules (hard constraints, can't be overridden)

- **NEVER** test multi-TF candle alignment (known [[v10i-look-ahead-bug|look-ahead trap]])
- **NEVER** test break-even stops (dead across 11 configs)
- **NEVER** claim a strategy validated based on backtesting alone
- **NEVER** re-test the 7 dead signals without a fundamentally new thesis
- **ALWAYS** use conservative simulation for trailing strategies *(insufficient — see [[bar-sim-trailing-bug]] caveat above)*
- **ALWAYS** walk-forward validate (4 quarters positive OOS)
- **ALWAYS** apply realistic costs (0.5pt slippage + 0.5pt commission per side)
- **ALWAYS** show RT (R target) in every results table

## The tournament

Each week, Nova and Sable compete. Scoring adapts to the current phase:

- **Research-only phase:** hypothesis quality, backtest rigor, did they avoid known dead ends, did walk-forward hold
- **Paper trading phase:** paper P&L, signal quality vs backtest expectations, risk compliance
- **Live trading phase:** real P&L, WR, max DD, consistency

Tournament judging uses **external AI advisor APIs** — Claude, Grok, ChatGPT — to provide independent scores. No single AI system judges its own work. The winner gets higher capital allocation for the next week.

## Phase progression (strict gating)

Both brains start in research-only mode. Advancement is earned, not assumed:

1. **Research only** — no orders, hypothesis-generation only. Exit: at least one brain produces a positive walk-forward OOS result.
2. **Paper trading** — IBKR paper account, real-time decisions with simulated fills. Exit: paper P&L justifies real capital.
3. **Micro live** — 1 MNQ ($2/point). Minimal real-money exposure. Exit: realized Sharpe and risk compliance.
4. **Scale up** — NQ contracts ($20/point), size capped by tournament rank.

## Relationship to the existing Context Engine

The Unified Brain doc references the [[bayesian-belief-engine|Context Engine V2]] as "already built and working" with:
- 46-feature market state vector
- Bayesian session classifier at 68.6% accuracy
- Strategy bridge
- Volume profile + footprint analysis

Nova and Sable should be able to query the Context Engine as a tool — i.e. the Engine computes the feature vector for a given moment, and the brain reasons about what to do with it. The relationship between the Engine's deterministic classifier and the brains' Mem0-based pattern-matching is **not fully specified** in the architecture doc — it's an open implementation question.

## Status as of 2026-04-11

**Spec only.** Nova and Sable do not exist as running software. Phase 1 (Weeks 1–3: brain + memory + research pipeline) has not begun. Harrison's `~/Documents/trading-system/CLAUDE.md` confirms that `v24 IFVG MTF cascade` is the only strategy currently on NinjaTrader sim, which is the reference point the brains will be benchmarked against once they exist.

## See also

- [[unified-brain-architecture]] — full summary of the architecture doc
- [[tempo-trading-system]] — the Mac-side directory where the brain will be built
- [[research-arc-map]] — Phase 5 "Unified Brain pivot" context
- [[bar-sim-trailing-bug]] — an implementation blocker for the research pipeline
- [[v10i-look-ahead-bug]] — hardcoded anti-pattern #1 in the Mem0 seed
- [[bayesian-belief-engine]] — the existing Context Engine the brains will query
- `~/Documents/trading-system/CLAUDE.md` — project-side directive
