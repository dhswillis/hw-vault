---
created: 2026-04-20
updated: 2026-04-20
type: summary
sources:
  - "audit of local machine (darwin 25.2.0) on 2026-04-20"
related:
  - wiki/summaries/openclaw-vps-setup-guide.md
  - wiki/summaries/autonomous-trading-system-progress-summary.md
  - wiki/maps/automation-moc.md
tags: [summary, infrastructure, openclaw, audit, in-flight]
---

# OpenClaw Install Audit — 2026-04-20

> Point-in-time audit of what's actually installed, configured, and running for OpenClaw on Harrison's Mac. Captured because the setup has been sitting untouched since mid-February — useful to know what still works before pivoting attention.

## TL;DR

OpenClaw is **installed and configured but not running**. The gateway launchd service is loaded but the runtime is stopped. Ollama (the primary model provider) is not responding. Telegram is paired. There are zero cron jobs scheduled. Custom agent skills for the "Willis Holdings" CEO-briefing/trading-operator setup are on disk. Main session transcript was lost, so chat history will appear reset on next start.

**Verdict**: Revivable in 15 minutes if desired. Not actively doing anything today.

## What's installed

| Component | Version | Location | Status |
|---|---|---|---|
| `openclaw` (npm global) | `2026.2.15` | `/opt/homebrew/bin/openclaw` | installed |
| `@anthropic-ai/claude-code` | `2.1.42` | `~/.local/bin/claude` | installed |
| Node.js | `v25.6.1` | system | OK |
| Docker | — | — | **not installed** |
| Ollama | — | expected at `127.0.0.1:11434` | **not responding** |

Config file: `~/.openclaw/openclaw.json` (last touched 2026-02-19). Gateway auth token is set (redacted from this note).

## Gateway

- launchd service `ai.openclaw.gateway` is **loaded but stopped** (PID column shows `-`)
- LaunchAgent plist: `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
- File logs: `/tmp/openclaw/openclaw-2026-04-20.log`, `~/.openclaw/logs/gateway.{log,err.log}`
- Bind: loopback only (`ws://127.0.0.1:18789`)
- doctor says: "Service is loaded but not running (likely exited immediately)"
- To revive: `openclaw gateway start` (or `openclaw doctor --fix`)

## Model routing

Primary model provider is **local Ollama** — llama3.3 as "Brain", llama3.1:8b as "Fast". No Anthropic/OpenAI cloud fallback wired in the config file today. Doctor reports `TypeError: fetch failed` discovering Ollama models → Ollama daemon isn't running.

Implication: even if the gateway started, no model would respond until Ollama is up.

## Channels

- **Telegram**: enabled, paired with `@parabolic_dreams` (chat ID 451811002). DM policy = pairing, group policy = allowlist, stream mode = partial.
- **Discord, BlueBubbles, Slack, Matrix**: plugins present but disabled.

Plugins loaded: 5/36 — `device-pair`, `memory-core`, `phone-control`, `talk-voice`, `telegram`.

## Agents

The OpenClaw instance hosts a "**Willis Holdings**" persona — a CEO-operator (named "Operator") running a conglomerate of AI-managed business divisions. Defined in `~/.openclaw/workspace/IDENTITY.md`.

**Active divisions** (per identity file):
1. Trading (TEMPO — NQ futures)
2. Finance (CFO — tax calendar, FP&A)
3. HR & Bots (agent roster, model routing, ~$55/mo API)
4. IT Infrastructure (VPS `172.93.181.209`, cron, deploy)

**Planned divisions**: Arbitrage, Content, Research, BizDev, Fundraising.

**Decision thresholds**: under $500 execute autonomously; $500–$5k execute + flag; over $5k propose and wait.

### Custom workspace skills (on disk)
- `ceo-briefing` — daily morning/EOD briefing generator, pulls status across divisions, pushes to Telegram
- `claude-code-manager` — spawns Claude Code sessions for complex coding work
- `shell-runner` — safe shell command runner with destructive-op confirmation
- `trading-operator` — monitors backtest status, NautilusTrader pipeline, NQ results

### Stock skills available
`clawhub`, `coding-agent`, `github`, `healthcheck`, `openai-image-gen`, `skill-creator`, `video-frames`, `weather`.

## Memory

- Personality + continuity files at `~/.openclaw/workspace/{SOUL,USER,AGENTS,TOOLS,IDENTITY,HEARTBEAT}.md`
- Daily memory notes convention: `memory/YYYY-MM-DD.md` + curated `MEMORY.md` (main sessions only)
- **HEARTBEAT.md is empty** → no periodic heartbeat tasks running
- Memory search: **configured but disabled** — no embedding provider (no `OPENAI_API_KEY` or `GEMINI_API_KEY` found). Semantic recall off.

## Cron

`~/.openclaw/cron/jobs.json` → `{ "version": 1, "jobs": [] }`. **Zero scheduled jobs.** Nothing is running on a timer.

## State integrity issues (flagged by doctor)

- **Main session transcript missing** at `~/.openclaw/agents/main/sessions/29301c9b-c25c-4f9e-a168-265f06943350.jsonl`. History will appear to reset on next session.
- **42 skills** show "missing requirements" (likely bins not on PATH or credentials absent). 11 skills are usable today.

## What's working vs. what's broken

| Working | Broken / not running |
|---|---|
| Binary + config installed, version current | Gateway not running |
| Telegram paired | Ollama not responding |
| Custom skills on disk | No embedding provider |
| Launchd service registered | Main session transcript lost |
| Willis-Holdings identity/persona files intact | Zero cron jobs scheduled |
| VPS referenced in scripts | VPS connectivity not verified today |

## Revival checklist (if desired)

1. Start Ollama (`ollama serve` or the app) so local models respond
2. `openclaw gateway start` — bring the gateway up
3. `openclaw doctor` — re-verify everything
4. (Optional) add `OPENAI_API_KEY` / `GEMINI_API_KEY` to `~/.openclaw/.env` to re-enable semantic memory
5. (Optional) wire a cloud-model fallback (Anthropic provider) in the config so the gateway isn't single-threaded on Ollama
6. (Optional) schedule the `ceo-briefing` skill via `openclaw cron` for morning/EOD delivery to Telegram

## Related / cross-refs

- [[wiki/summaries/openclaw-vps-setup-guide|OpenClaw VPS setup guide]] — the ~$120/mo QuantVPS deployment pattern
- [[personal/projects/home-manager/whitepaper-claude-code-kit|Whitepaper: Home-Manager-in-a-box]] — uses this install shape as the reference architecture for the productized kit
- Auto-memory cross-ref: (no direct overlap today)
