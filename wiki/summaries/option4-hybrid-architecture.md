---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Downloads/Option4_Hybrid_Handoff_Document.docx
related:
  - wiki/summaries/unified-brain-architecture.md
  - wiki/summaries/tempo-project-state.md
  - wiki/summaries/tempo-context-engine-spec.md
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/quantconnect.md
  - wiki/entities/nova-sable-brains.md
  - wiki/concepts/ifvg.md
  - wiki/maps/tempo-moc.md
  - wiki/maps/context-engine-moc.md
tags: [trading, architecture, canonical, execution, infrastructure]
---

# Option 4: Hybrid Full Stack — Architecture Document

**Source:** `raw-sources/imports/2026-04-11/Downloads/Option4_Hybrid_Handoff_Document.docx`
**Dated:** February 2026
**Status:** **Canonical for the execution architecture.** Harrison has repeatedly said "read this for the correct system design." The [[unified-brain-architecture|Unified Brain doc]] references it explicitly as the correct architecture. The two docs are **complementary, not contradictory**: Option4 defines HOW the system executes; Unified Brain defines HOW the brains research and discover strategies.

## The four layers

| Layer | Name | Role | Where |
|---|---|---|---|
| 1 | **Claude Code** | The Builder — writes all code | MacBook Air M4 |
| 2 | **Custom API Wrapper** | The Genius Brain — 100K+ token knowledge base injected as system prompt via FastAPI + Claude API | VPS |
| 3 | **OpenClaw** | The Operator — autonomous 24/5 agent framework | VPS (QuantVPS Pro, Chicago, $70/mo) |
| 4 | **Claude Business Suite** | Harrison's daily tools — claude.ai, Chrome, Cowork | Mac/browser |

**Metaphor from the doc:** "Claude Code is the architect. The custom API wrapper is the genius brain that never forgets. OpenClaw is the hands and legs that operate 24/7."

## Why Option 4 (and not the other three)

| Option | Approach | Limitation |
|---|---|---|
| 1 | Claude Code + local .md files | Not autonomous — requires Harrison present |
| 2 | Custom Claude API Wrapper alone | No persistence, can't act on its own |
| 3 | OpenClaw + vector database alone | Quality depends on context provided |
| **4** | **All three combined** | Most complex, but institutional-grade |

## Execution flow (production signal chain)

```
Brain identifies setup
  → calls API Wrapper for validation (full 100K+ token context)
  → writes signal.json to shared folder
  → Quantower C# FileWatcher reads signal
  → Quantower executes order
  → Replikanto copies to prop firm accounts
  → Quantower writes fill.json
  → OpenClaw reads fill, logs trade with context, updates P&L
```

## The API Wrapper (Layer 2) — solving the memory problem

Claude's native memory = 30 edits at 200 chars each. The trading knowledge base exceeds 100K tokens. The API Wrapper is a FastAPI app that injects the **entire** knowledge base as the system prompt on every call. Prompt caching reduces cost ~90% (system prompt is the same every call). Cost: ~$50/mo API credits.

The system prompt contains: all brain definitions (entry/exit logic, filters), Context Engine variable reference, hard/soft filter thresholds, session ranges, DOL/reaction point frameworks, dynamic entry TF selection, day fingerprinting, 3R/day guardrails, performance benchmarks.

## 11 planned brains

### Core brains (from educators)

| # | Brain | Source | Status |
|---|---|---|---|
| 1 | **Tempo** | Tempo (Discord) — [[ifvg|IFVG]] + sweeps + [[smt|SMT]] | v24 on NT sim; [[tempo-rules-v3|rules extracted from 281 transcriptions]] |
| 2 | Lanto | Lanto (Discord) — supply/demand | Not yet extracted |
| 3 | Carmine | Carmine (YouTube) | Not yet extracted |
| 4 | Omega | Omega (Discord/YouTube) | Not yet extracted |
| 5 | Flipping Markets | Flipping Markets (PDF) | Recommended as second brain (easiest) |

### Advanced brains (institutional layer)

| # | Brain | Strategy source |
|---|---|---|
| 6 | SpotGamma Levels | GEX-derived S/R: Call Wall, Put Wall, Gamma Flip, Vol Trigger |
| 7 | Vanna/Charm Drift | Delta decay patterns near OPEX and last hour |
| 8 | Dark Pool Regime | DIX/GEX combo + dark pool accumulation zones |
| 9 | Order Flow Absorption | DOM reading: absorption, iceberg detection, delta confirmation |
| 10 | Sweep Momentum | FlowAlgo/CheddarFlow alerts + Context Engine alignment |
| 11 | Institutional Liquidity | Bookmap large lot clusters + CVD divergence |

## Context Engine (from Option4 — predates the [[tempo-context-engine-spec|v1.0 spec]])

Hard filters (binary: trade or don't):
- High-impact news within 15 min → no trades
- VIX > 35 (crisis) → no trades
- Before 8:30 AM or after 3:00 PM ET → no new positions
- GEX extreme bearish (DIX < 0.35) → no longs
- Price beyond SpotGamma 1-Day Expected Range → mean reversion only

Soft filters (position size multipliers):
- DIX bullish (>0.47) + long → 1.25×
- Price near SpotGamma Call/Put Wall → 1.15×
- Dark pool cluster aligns → 1.2×
- CVD divergence at zone → 1.15×
- GEX positive + inside day → 1.1×

Day fingerprinting: day of week + FOMC proximity + CPI/NFP proximity + OPEX proximity + VIX regime + GEX regime + gap size + IB range vs ATR.

## Tempo Brain — detailed rules (Section 4 of the doc)

Core sequence (matching the [[ifvg|canonical IFVG]] and the user's Claude-data-export codified rules):

1. FVG on dynamic entry TF (5s–5m per vol/tick regime)
2. Candle closes through → valid
3. Close simultaneously sweeps a HTF level: 15m fractal, session H/L, or 5m/15m FVG fill
4. Checklist passes: no news, not extreme momentum against
5. Entry: inverse (fade the sweep)

Dynamic entry TF:
- Low vol: 3m–5m, 1m FVGs relevant
- Medium vol: 1m–2m, 5m/15m FVGs + session H/L
- High vol: 5s–30s, only large levels matter

## Infrastructure costs

| Item | Monthly | Stage |
|---|---|---|
| Claude Max 5× | $100 | Now |
| Claude API credits | ~$50 | Stage 1 |
| QuantVPS Pro | $70 | Stage 1 |
| SpotGamma | $50–100 | Stage 2 |
| Tradytics | $69 | Stage 2 |
| Bookmap | $113–158 | Stage 3 |

Stage 1 total: ~$220/mo. Year 1 all-in (with MacBook): ~$4,800; ongoing ~$2,600/yr.

## Build phases

| Stage | Name | Timeline | Harrison's time |
|---|---|---|---|
| 1 | Foundation + First Brain | Weeks 1–8 | 5–10 hrs/week |
| 2 | Paper → Live | Weeks 8–16 | 3–5 hrs/week |
| 3 | Multi-Brain | Weeks 16–28 | 2–3 hrs/week |
| 4 | Full System | Months 6–12 | 1–2 hrs/week |
| 5 | Institutional Scale | Year 2+ | 1 hr/week |

## Operational cycle (how OpenClaw runs a trading day)

```
5:30 AM — Context Engine pulls data, writes context.json
6:00 AM — Telegram briefing sent to Harrison
8:30 AM — Brains activated
Continuous — scan for setups, call API Wrapper for validation
Signal detected — write signal.json → Quantower → Replikanto → prop firms
3:00 PM — Positions closed
3:30 PM — EOD report via Telegram
4:00 PM — Performance DB updated
Friday — Weekly learning loop analysis
Weekend — OpenClaw sends improvement suggestions, Harrison reviews
Next dev session — Claude Code implements approved changes
```

## Relationship to the Unified Brain

| Dimension | Option 4 | Unified Brain |
|---|---|---|
| Scope | **Execution architecture** — how signals become trades | **Research architecture** — how strategies are discovered |
| Who finds the strategy? | Harrison + Claude Code extract from educators | Nova + Sable AI researchers compete in tournaments |
| Who executes? | OpenClaw → Quantower → Replikanto | Same (inherited from Option 4) |
| Who provides context? | API Wrapper (100K+ token system prompt) | Mem0 + SQLite + weekly LoRA |
| When was it written? | February 2026 | April 2026 |
| Status | **Canonical for execution layer** | **Canonical for research layer** |

**Option 4 is the "what runs in production." Unified Brain is the "what figures out what to run."** Both are current.

## Standing corrections from Harrison (Section 1 of the doc and TEMPO_PROJECT_STATE)

1. **Do NOT invent rules.** Only use rules explicitly stated by Tempo.
2. **Follow the Option 4 architecture.** Not a monolithic QC algo.
3. **The real rules live in 324 trade recap videos.** The 23 text-extracted rules are a skeleton only.
4. **SMT = Smart Money Technique** (NQ vs ES divergence), NOT "Supply Manipulation Test."
5. **QC is for backtesting only.** Not the production execution layer.
6. **Risk per trade is 0.4–0.6%, NOT 1%.**
