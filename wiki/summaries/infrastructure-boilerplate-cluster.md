---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Documents/trading-system/CLAUDE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/operator/CLAUDE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/nautilus-bt/CLAUDE.md
  - raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/CLAUDE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/AGENTS.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/HEARTBEAT.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/USER.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/SOUL.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/MW_CLAUDE_CODE_PROMPT.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/MOTIVEWAVE_PORT_PROMPT.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/OVERNIGHT_MINE_PROMPT.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/MINE_V2_PROMPT.md
  - raw-sources/imports/2026-04-11/Documents/V17_AUDIT_PROMPT.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/trading-operator/SKILL.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/ceo-briefing/SKILL.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/claude-code-manager/SKILL.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/shell-runner/SKILL.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/HEARTBEAT.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/USER.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/SOUL.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/skills/genius-brain/SKILL.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/KNOWLEDGE_BASE.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/docs/brains/TEMPO.md
  - raw-sources/imports/2026-04-11/Downloads/trading-system/docs/ARCHITECTURE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/knowledge-base/master.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/trading_operator/logs/health_check.md
  - raw-sources/imports/2026-04-11/Documents/README_EXCEL_ANALYSIS.md
  - raw-sources/imports/2026-04-11/Documents/EXCEL_STRATEGY_OVERVIEW.md
  - raw-sources/imports/2026-04-11/Documents/START_HERE.md
  - raw-sources/imports/2026-04-11/Documents/trading-system/results/archive/bar-sim-pre-2026-04-10/README.md
  - raw-sources/imports/2026-04-11/trading-system/CLAUDE.md
  - raw-sources/imports/2026-04-11/trading-system/LOCAL_SETUP.md
  - raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/MINING_SIGNAL_SPEC.md
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/openclaw-vps-setup-guide.md
  - wiki/maps/automation-moc.md
tags: [summary, cluster, infrastructure, boilerplate, prompts, skills]
---

# Infrastructure & Boilerplate — Cluster Summary

> 33 operational documents: CLAUDE.md context files (×5), OpenClaw skills (×5), mining/audit prompts (×5), knowledge bases, architecture docs, setup guides, health checks, and data analysis READMEs.

## Contents

These are the operational scaffolding that makes the trading system work — not research or strategy documents, but the glue.

**CLAUDE.md files (×5):** Auto-context files for different Claude Code sessions (main trading system, operator, nautilus-bt ×2, and a second trading-system instance). These define what Claude knows when it starts a session.

**OpenClaw skills (×5):** trading-operator, ceo-briefing, claude-code-manager, shell-runner, genius-brain. Skill definitions for the autonomous agent system.

**Prompts (×5):** OVERNIGHT_MINE_PROMPT, MINE_V2_PROMPT, MW_CLAUDE_CODE_PROMPT, MOTIVEWAVE_PORT_PROMPT, V17_AUDIT_PROMPT. Reusable prompts for specific operations.

**Knowledge bases:** KNOWLEDGE_BASE.md, master.md, ARCHITECTURE.md, TEMPO.md (brains doc). System-level context documents.

**Infrastructure:** AGENTS.md (agent definitions), HEARTBEAT.md/USER.md/SOUL.md (OpenClaw personality files ×2 copies), health_check.md (operator log), MINING_SIGNAL_SPEC.md.

**Data/Excel:** README_EXCEL_ANALYSIS.md, EXCEL_STRATEGY_OVERVIEW.md, START_HERE.md (duplicate), bar-sim archive README.

## Status / caveats

Operational scaffolding — no analytical claims. Many are duplicates across import paths. The CLAUDE.md files are the most valuable as they encode session context that would otherwise be lost. OpenClaw skills may be outdated if the system has moved to the Unified Brain architecture.

## Source documents

- [[raw-sources/imports/2026-04-11/Documents/trading-system/CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/operator/CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/nautilus-bt/CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/CLAUDE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/AGENTS]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/HEARTBEAT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/USER]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/SOUL]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/MW_CLAUDE_CODE_PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/MOTIVEWAVE_PORT_PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/OVERNIGHT_MINE_PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/MINE_V2_PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/V17_AUDIT_PROMPT]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/trading-operator/SKILL]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/ceo-briefing/SKILL]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/claude-code-manager/SKILL]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/openclaw/skills/shell-runner/SKILL]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/HEARTBEAT]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/USER]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/SOUL]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/.openclaw/skills/genius-brain/SKILL]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/KNOWLEDGE_BASE]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/docs/brains/TEMPO]]
- [[raw-sources/imports/2026-04-11/Downloads/trading-system/docs/ARCHITECTURE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/knowledge-base/master]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/trading_operator/logs/health_check]]
- [[raw-sources/imports/2026-04-11/Documents/README_EXCEL_ANALYSIS]]
- [[raw-sources/imports/2026-04-11/Documents/EXCEL_STRATEGY_OVERVIEW]]
- [[raw-sources/imports/2026-04-11/Documents/START_HERE]]
- [[raw-sources/imports/2026-04-11/Documents/trading-system/results/archive/bar-sim-pre-2026-04-10/README]]
- [[raw-sources/imports/2026-04-11/trading-system/CLAUDE]]
- [[raw-sources/imports/2026-04-11/trading-system/LOCAL_SETUP]]
- [[raw-sources/imports/2026-04-11/Documents/strategies/04-bos-fvg/nautilus/nautilus-bt/MINING_SIGNAL_SPEC]]
