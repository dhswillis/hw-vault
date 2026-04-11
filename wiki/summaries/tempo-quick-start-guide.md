---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources: [raw-sources/Tempo_Quick_Start_Guide.docx]
related:
  - wiki/entities/tempo-trading-system.md
  - wiki/entities/quantconnect.md
  - wiki/entities/databento.md
tags: [trading, tempo, operations, onboarding]
---

# Tempo Quick Start Guide

**Source:** `raw-sources/Tempo_Quick_Start_Guide.docx`
**Subject:** How to start a session on [[tempo-trading-system|Tempo]], where files live, what's next.

## Starting a session

```bash
tempo
```

This alias does three things:
1. `cd ~/Documents/trading-system`
2. `git pull origin main`
3. Launches Claude Code with full project memory

If the alias isn't set up:
```bash
echo 'alias tempo="cd ~/Documents/trading-system && git pull origin main && claude"' >> ~/.zshrc
source ~/.zshrc
```

Claude Code auto-reads `CLAUDE.md` → `START_HERE.md` → `CLAUDE_BOOTSTRAP.md` for persistent context.

## File locations

### On the Mac
| Path | Purpose |
|---|---|
| `~/Documents/trading-system/` | All source, docs, configs |
| `~/Documents/trading-system/CLAUDE.md` | Auto-read by Claude Code |
| `~/Documents/trading-system/START_HERE.md` | Full project state (source of truth) |
| `~/Documents/trading-system/CLAUDE_BOOTSTRAP.md` | Tech details, credentials, integrations |
| `~/Documents/trading-system/.local/config.json` | Credentials (gitignored) |
| `~/Documents/trading-system/batch-runner/` | QC strategy + batch scripts |
| `~/Documents/Tempo_Context_Engine_Spec.docx` | Engineering spec |
| `~/Documents/Tempo_Batch2_Winners.xlsx` | Batch 2 results |
| `~/Downloads/openclaw.json` | OpenClaw / Telegram config |

### On the VPS (172.93.181.209)
| Path | Purpose |
|---|---|
| `~/trading-system/batch-runner/` | Strategy code + scripts (GitHub mirror) |
| `~/trading-system/batch-runner/results/` | Batch results (JSONL) |
| `/root/.env` | QC credentials |
| `~/batch3_costmodel.log` | Current batch progress |

### On GitHub
Source of truth: **`dhswillis/trading-system`** (explains the [[tempo-trading-system|Tempo repo]]'s ownership — connects to the known [[|github_ssh_identity]] fact that local SSH authenticates as `dhswillis`).

PAT token expires **2026-05-15**.

### On QuantConnect
Project `28083727`, `main.py` (uploaded from `main_v4.py`). The VPS sends variant configs to QC via API; QC runs them; VPS collects results.

## Tools

| Tool | For | When |
|---|---|---|
| Claude Code | SSH, git, APIs, code, analysis | Primary tool |
| Cowork | Browser tasks (GitHub tokens, web UIs) | Click-things work |
| VPS Terminal | Direct SSH | Quick checks, fallback |
| OpenClaw/Telegram | Phone monitoring/alerts | Phone checks (skill incomplete) |

Rule of thumb: command-line → Claude Code; browser clicking → Cowork.

## Current state (Feb 14, 2026)

**Running:** Batch 3 on VPS — 2000 variants with realistic cost modeling (slippage, fees, tick-through, BE offset).

**Check progress:**
```bash
ssh root@172.93.181.209
tail -20 ~/batch3_costmodel.log
```

**Built and working:**
- QC strategy `main_v4.py` with realistic execution costs
- Two-stage batch runner (Stage A: 3-month screen → Stage B: 12-month confirm)
- 2000-variant generator (signals, R targets, BE modes, sessions)
- GitHub repo with auto-deploy
- Claude Code persistent memory chain
- OpenClaw/Telegram bot (configured, custom skill not finished)
- [[tempo-context-engine-spec|Context engine spec document]]

**Next (priority order):**
1. When Batch 3 finishes: analyze results, build Context Engine Phase 1 (price-only day classifier)
2. Context Engine Phase 2: Databento tick data, volume profile
3. Context Engine Phase 3: order flow / footprint
4. Context Engine Phase 4: news + cross-asset
5. Context Engine Phase 5: live integration + Telegram
6. Finish OpenClaw `tempo-trading` skill
7. Stage B: run top 50 Batch 3 winners through 12-month confirmation

## Credentials reference

All credentials in `.local/config.json` (gitignored). Never paste in chat, docs, or committed code.

| Service | Location | Notes |
|---|---|---|
| QuantConnect API | `.local/config.json → quantconnect` | `user_id: 113205, project: 28083727` |
| GitHub PAT | `.local/config.json → github` | Token `cowork-tempo`, expires 2026-05-15 |
| VPS SSH | Direct: `root@172.93.181.209` | Password auth |
| Databento | VPS `/root/.env` | Key saved, not yet integrated |
| Telegram Bot | `~/Downloads/openclaw.json` | Bot token + Harrison's Telegram ID `451811002` |

## Memory system

Document chain:
```
CLAUDE.md                       (auto-read by Claude Code)
  ↓
START_HERE.md                   (project state, infra, next steps)
  ↓
CLAUDE_BOOTSTRAP.md             (credentials, integrations, tech)
CONTEXT.md                      (strategy mechanics, signals, batch results)
Tempo_Context_Engine_Spec.docx  (context engine eng spec)
```

**Golden rule:** if it's not in `START_HERE.md` or linked from it, it doesn't exist next session.

## End-of-session checklist

- Update `START_HERE.md` "Current State"
- Commit and push to GitHub
- If new tools/services were built, add to `CLAUDE_BOOTSTRAP.md` "Built Integrations"

(This is the pattern designed to prevent OpenClaw-style "built but undocumented, lost" regressions.)
