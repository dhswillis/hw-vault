---
created: 2026-04-14
updated: 2026-04-14
type: summary
sources:
  - raw-sources/trading/BRAIN_ARCHITECTURE.md
related:
  - wiki/summaries/unified-brain-architecture.md
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/nova-sable-brains.md
  - wiki/summaries/tempo-context-engine-spec.md
tags: [summary, architecture, trading, openclaw, ralph-loop]
---

# Brain Architecture — Summary

Engineering spec for the Tempo trading system's AI brain: Ralph Loop + OpenClaw + Berman Trifecta integration.

## Three-layer architecture

**Layer 1 — OpenClaw Gateway (Nervous System):** Self-hosted agent runtime providing 24/7 infrastructure: multi-channel messaging (WhatsApp/Telegram/Discord), tool execution, memory persistence, model routing. Runs on port 18789.

**Layer 2 — Ralph Loop (Reasoning Engine):** Autonomous loop that breaks complex trading tasks into atomic units, executes each in a fresh AI context, persists learnings via git + progress files, iterates until acceptance criteria pass. Handles strategy development, backtesting, system evolution. Uses PRD-driven task decomposition with quality gates.

**Layer 3 — Berman Trifecta (Model Stack):** Opus 4.6 for deep reasoning/strategy design, GPT-5.3 Codex for code generation/execution, OpenClaw as orchestrator routing to the right model.

## Agent workspaces

Four specialized agents: STRATEGIST (research, design), EXECUTOR (backtest, deploy), MONITOR (live tracking), RISK (drawdown, limits). Each operates in an isolated workspace with persistent state.

## Trading-specific extensions

Ralph Loop adapted for trading with: acceptance criteria tied to Calmar/WR/walk-forward metrics, backtesting as the test suite, git-committed progress files as persistent memory across sessions, and automatic iteration on failed quality gates.

## Relationship to Unified Brain

This doc is the engineering implementation spec. The [[unified-brain-architecture]] summary covers the conceptual architecture and the anti-stupidity rules. Together they define the full system design.

## Source documents

- [[raw-sources/trading/BRAIN_ARCHITECTURE]]
