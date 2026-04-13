---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Downloads/OpenClaw_VPS_Setup_Guide.docx
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/summaries/autonomous-trading-system-progress-summary.md
  - wiki/maps/automation-moc.md
tags: [summary, infrastructure, vps, openclaw]
---

# OpenClaw VPS Setup Guide — Summary

> Step-by-step infrastructure guide for running a persistent autonomous trading agent on a VPS via OpenClaw + Telegram + Claude API. Estimated cost ~$120/month.

## Key findings

Solves the persistent knowledge problem — every Claude instance has limited memory, and strategy knowledge gets lost between sessions. The setup has three components: a QuantVPS running OpenClaw as a daemon, a Telegram bot for command/control, and the Claude API for reasoning when needed.

Installation walks through SSH access, Node.js 22+, Docker, OpenClaw global install, onboarding wizard (Anthropic provider, model selection), and Telegram bot setup via @BotFather. The knowledge base lives as files on disk that OpenClaw reads before each API call, providing full context without manual bridging.

## Status / caveats

Operational guide — instructions may be version-sensitive. The OpenClaw framework has since been superseded by the [[wiki/summaries/unified-brain-architecture|Unified Brain]] architecture in the overall system direction, but the VPS infrastructure it describes is still the execution substrate.
