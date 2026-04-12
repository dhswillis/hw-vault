---
created: 2026-04-11
updated: 2026-04-11
type: summary
sources:
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
related:
  - wiki/summaries/tempo-rules-v3
  - wiki/entities/nova-sable-brains.md
  - wiki/entities/tempo-trading-system.md
  - wiki/concepts/bayesian-belief-engine.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/concepts/be-trail-mechanism.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/summaries/tempo-cluster.md
  - wiki/syntheses/tempo-three-layers.md
  - wiki/syntheses/research-arc-map.md
tags: [trading, tempo, architecture, unified-brain, canonical]
---

# Unified Brain Architecture — Summary

## What the source is

`~/HW/raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx` — **440 lines** (as converted via `textutil`), **Willis Holdings, April 2026, v2.0**. The most recent and authoritative planning document in the Tempo cluster. Replaces three earlier planning docs: *The Partnership*, *AI Trading Show Blueprint*, and *AIO System Blueprint*. This doc is the canonical project-side correction to the mining-era research and the origin of the position "nothing from V7–V11 is validated."

## The honesty disclosure (verbatim, front of doc)

> "NOTHING from the backtesting research (V7-V11) is validated for live trading. BOS_FVG does not work. V7/V8 was killed by look-ahead bias. The V10t clean results are approximately breakeven after commissions. The research journal documents what was TESTED, not what WORKS.
>
> v24 (IFVG MTF cascade) is the ONLY strategy currently on NinjaTrader sim. It has not yet proven itself. It is still collecting data.
>
> The brain's job is not to execute a proven strategy. There is no proven strategy. The brain's job is to FIND one."

This is the ground truth for the project's current state. Every vault page that tags something `suspect-results` flows from this position.

## The core thesis

The doc frames the project's year of mining research as a **lack of persistent intelligence**, not a lack of research:

> "The core issue is not a lack of research — it is a lack of persistent intelligence. Every Claude session starts from zero. Every correction gets lost. Every dead end gets revisited. You need AI that remembers what it learned, builds on previous work, and never makes the same mistake twice."

The solution: two local AI models with permanent memory, baked-in knowledge of every dead end, weekly self-improvement via LoRA fine-tuning, and a tournament system that creates competitive pressure.

## The two brains — [[nova-sable-brains|Nova and Sable]]

|  | **Nova (Partner A)** | **Sable (Partner B)** |
|---|---|---|
| **Role** | Methodical researcher. High WR, small trades, strong statistics | Aggressive researcher. Bold hypotheses, higher R-multiple |
| **Style** | Systematic, full parameter sweeps, demands walk-forward + conservative sim | Intuitive, structural patterns, willing to trade smaller samples |
| **Base model** | DeepSeek R1-Distill-Qwen-14B via Ollama | DeepSeek R1-Distill-Qwen-14B via Ollama |
| **LoRA weighting** | Conservative: 2× penalty on losses, 1× on wins | Aggressive: 2× bonus on big wins, 1× penalty on losses |
| **Memory** | Mem0 semantic + SQLite structured — independent stores | Same architecture, independent stores |
| **Show persona** | The analyst — viewers trust her for education | The wild card — viewers watch her for entertainment |
| **Voice** | ElevenLabs, clear professional female | ElevenLabs, energetic expressive female |

**Critically, memories are NOT shared between brains.** They learn different lessons from the same data. This is what makes the tournament meaningful.

## The four phases of brain progression

| Phase | Scored on | Capital at risk |
|---|---|---|
| **Research only** | Hypothesis quality, backtest rigor, did they avoid known dead ends, did walk-forward hold | None |
| **Paper trading** | Paper P&L (R earned), signal quality vs backtest, risk compliance, session adaptability | None (IBKR paper) |
| **Micro live** | Real P&L, WR, max DD, largest single loss, daily/weekly consistency | 1 MNQ ($2/point) |
| **Scale up** | Same as micro + Sharpe | NQ contracts ($20/point), size capped by tournament rank |

**Key difference from v1 of this doc:** the previous version assumed the brain would execute v24 as a known-good strategy. That was wrong — v24 is unproven. v2 treats the brain as a **researcher first**. It inherits the full research history (including what does NOT work) and its job is to find what does.

## Risk Officer (hard-coded Python, not AI)

Deterministic Python. No AI, no learning, no flexibility. If it says no, the trade does not happen.

| Rule | Threshold | Action | Override |
|---|---|---|---|
| Daily loss limit | −3R per brain per day | Brain shut down for session | None |
| Weekly drawdown | −10R per brain per week | Observation mode until Monday | None |
| Max position size | Kelly + tournament rank cap | Order rejected | None |
| Correlation block | Both brains same direction | Second order halved | None |
| Session gate | Outside US session / news | All orders blocked | None |
| Equity floor | Account below start −15% | System halted | Human review only |

## Tournament judging

Each week, Nova and Sable compete. Scoring adapts to current phase (research / paper / live). **External AI advisors** — Claude API, Grok API, ChatGPT API — provide independent scores. No single AI system judges its own work. The winner gets higher capital allocation for the next week.

## Anti-stupidity rules (hard-coded on Mem0 seed)

- **NEVER** test multi-timeframe candle alignment — known [[v10i-look-ahead-bug|look-ahead trap]]
- **NEVER** test break-even stops — dead across 11 configurations (confirmed full year)
- **NEVER** claim a strategy is validated based on backtesting alone
- **ALWAYS** use conservative simulation (worst-case OHLC ordering) for trailing strategies *(note: this rule is incomplete — see [[bar-sim-trailing-bug|bar-sim trailing bug]] discovered 2026-04-10)*
- **ALWAYS** run walk-forward validation (minimum 4 quarters, all positive OOS)
- **ALWAYS** apply realistic costs (0.5pt slippage + 0.5pt commission per side)
- **NEVER** re-test the 7 dead signals without a fundamentally new thesis
- **ALWAYS** show RT (R target) in every results table

## Dead strategies (baked into Mem0 seed)

Listed verbatim from the doc: `london_breakout`, `VP_POC_retest`, `fib_retracement`, `failed_breakout`, `sweep_reversal`, `double_BOS_momentum`, `VWAP_mean_reversion`. All tested, all negative.

## Dead filters (baked in)

- Multi-TF candle alignment (look-ahead bias) — all 5 clean alignment methods
- Structural filters: cluster, body ratio, volume, displacement, VWAP, FVG presence
- Stacked imbalances, exhaustion prints, absorption

## Dead management techniques (baked in)

- Break-even stops (kills edge)
- Inverse trades
- Re-entry after correct signal
- Partial profits
- Time stops
- V-reversals
- Runner management without alignment

## Look-ahead traps (explicitly listed)

- Multi-TF alignment using forming bar closes
- Intra-bar OHLC ordering assumptions
- Extended 5M without end-time cap
- Every bug from the V10o audit

**What's missing from this list (discovered after the doc was written):** the [[bar-sim-trailing-bug|bar-sim trailing path-reconstruction bug]] from 2026-04-10. The doc's "conservative simulation (worst-case OHLC ordering)" rule catches the strict intra-bar ordering variant (<1% of trades affected on 1m bars) but not the full path-reconstruction problem that affects every trailing-stop sim. The Unified Brain's Mem0 seed should be updated with the April finding before Phase 1 research begins.

## The research loop

The brain's job is to generate and evaluate hypotheses without human intervention:

1. **OBSERVE** — ingest latest market data, review recent trade outcomes
2. **RECALL** — query Mem0 for related patterns and past experiments
3. **HYPOTHESIZE** — propose a testable idea
4. **CHECK DEAD ENDS** — cross-reference against known failures, abort if redundant
5. **DESIGN** — specify backtest parameters, sample period, success criteria
6. **TEST** — run via `clean_backtest.py` on tick data, walk-forward validate
7. **EVALUATE** — conservative sim, slippage stress, commission drag
8. **RECORD** — log to Mem0 + SQLite, win or lose
9. **SHARE** — post findings to show dialogue + Discord
10. **REPEAT**

**Critical caveat:** step 6 references `clean_backtest.py`, which is the same file that produced the BOS_FVG +0.895 avgR bar-sim result. The Unified Brain cannot use `clean_backtest.py` for trailing-stop strategies without first fixing the bar-sim trailing bug documented in [[bar-sim-trailing-bug]]. This is an implementation blocker for Phase 1.

## Memory architecture

### Mem0 (semantic)
Vector-based memory that stores experiences as embeddings. When the brain encounters a market pattern, it searches Mem0 for similar past situations and their outcomes. Seed memories include the full dead-end list, research journal key findings, every correction Harrison has made, all execution parameters.

### SQLite (structured)
Every trade and research experiment logged: timestamp, entry, stop, target, fill price, R result (RT always captured), session type, context features, reasoning text, other brain's opinion. Research: hypothesis text, backtest parameters, result metrics, walk-forward outcome, pass/fail, related dead ends checked. Corrections: any time Harrison overrides a brain decision or corrects a belief, logged permanently.

### Weekly LoRA fine-tuning
Export the week's trades + research outcomes from SQLite. Nova trains with conservative weighting; Sable with aggressive. RunPod A100, 2–4 hours per brain, ~$5–15 per run. Merge adapter back, validate on last 20 trades, auto-rollback on failure.

## The show (YouTube Live)

The brain runs a daily YouTube Live stream — not scripted, not pre-recorded. Nova and Sable analyze markets in real time, genuinely disagree, and make real trade decisions.

**Technical pipeline:**
1. Brain outputs reasoning text (200–500 words, 2–4 sec inference)
2. Dialogue generator compresses to conversational exchange
3. Claude API refines for personality consistency
4. ElevenLabs Flash v2.5 → speech (75ms first-byte latency)
5. BocaLive renders two AI avatars with lip-sync
6. FFmpeg composites (avatars left+right, chart center, P&L overlay)
7. RTMP push to YouTube Live

Total latency: ~7–8 seconds. Fine for commentary, not meant as real-time execution feed.

**Daily schedule:** Pre-market 9:15–9:30 ET, open action 9:30–10:00, core session 10:00–11:30, midday wrap 11:30–12:00, post-stream auto-clips.

## Community & revenue channels

| Channel | Model | Target | Notes |
|---|---|---|---|
| Trading capital | Own capital via IBKR | 1 NQ = $400/RT | Primary, scale with results |
| Discord Premium | Stripe subscription | $49–99/mo | Real-time signals + AI chat |
| YouTube revenue | CPM + memberships | $2–8 CPM | Grows with viewership |
| Clip content | TikTok/Shorts/Reels | Ad rev + funnel | Auto-generated |
| Sponsorship | Weekly integration | $500–5K/week | After 10K+ audience |

Monthly cost estimate: **$220 (MVP) to $650 (full production)**. Breakdown: GPU VPS $80–150, RunPod LoRA $20–60, ElevenLabs $22–99, BocaLive $58–200, Claude API $20–50, market data $0–50, misc $20–40.

## Build phases

- **Phase 1 (Weeks 1–3)** — The Brain: Ollama + UnifiedBrain class + Mem0 + Risk Officer + research pipeline + tournament + LoRA. Exit criteria: both brains generating hypotheses, Mem0 refusing to re-test dead ends, at least one brain produced positive OOS walk-forward result.
- **Phase 2 (Weeks 4–7)** — Execution + Show: IBKR paper trading, ElevenLabs voices, BocaLive avatars, YouTube RTMP, first public stream.
- **Phase 3 (Weeks 8–12)** — Community + Scale: Discord bot, Stripe, 1 MNQ live, auto-clip highlights.

## Target file structure (partial, brains-specific)

```
trading-system/
├── brains/
│   ├── unified_brain.py       # NEW — Ollama + Mem0 + research loop
│   ├── nova.py                # NEW — Nova personality + research style
│   └── sable.py               # NEW
├── risk/
│   └── risk_officer.py        # NEW — Hard-coded limits
├── research/
│   ├── hypothesis_engine.py   # NEW
│   ├── experiment_runner.py   # NEW
│   ├── dead_end_checker.py    # NEW
│   └── walk_forward.py        # NEW
├── execution/
│   ├── ibkr_executor.py       # NEW — ib_insync order management
│   └── order_router.py        # NEW — Brain → Risk → IBKR
├── tournament/
│   ├── scorer.py, judges.py, weight_allocator.py
│   └── training/  # data exporter, lora trainer, model deployer
├── memory/
│   ├── mem0_store.py, trade_log.py
│   └── corrections.py         # NEW — Harrison's overrides, permanent
├── stream/  # dialogue generator, tts, avatar controller, compositor
└── orchestrator.py            # NEW — main loop
```

Notable: the `research/` directory and `memory/corrections.py` are what make the brain fundamentally different from a strategy executor.

## Immediate next steps (from the doc)

1. Keep v24 running on NinjaTrader sim — collect 2 more weeks of results
2. Set up GPU VPS with Ollama + DeepSeek R1-Distill-14B
3. Build `unified_brain.py` with Mem0 integration; seed Mem0 with dead-end list, research journal, corrections
4. Build `risk_officer.py`, unit test every rule
5. Build the research pipeline: `hypothesis_engine.py` + `experiment_runner.py` + `dead_end_checker.py`; connect to `clean_backtest.py`
6. Run the first research tournament — one week, review findings, course-correct

> "The brain's first job is to figure out if NQ futures can be traded profitably with the tools and data you have. If it can, everything else follows. If it cannot, you have saved yourself months of building infrastructure around an empty core."

## Cross-references

- [[nova-sable-brains]] — entity page for the two AI researchers
- [[tempo-trading-system]] — the project this architecture is being built into
- [[research-arc-map]] — Phase 5 context
- [[tempo-three-layers]] — why the pivot makes sense
- [[bar-sim-trailing-bug]] — a post-doc discovery that needs to be added to the Mem0 seed before Phase 1 begins
- [[bayesian-belief-engine]] — the Context Engine referenced as "already built and working"
- [[tempo-cluster]] — master cluster
- `~/Documents/trading-system/CLAUDE.md` — the project-side directive consistent with this doc
