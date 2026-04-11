# Operator Agent — Harrison's AI Desktop Assistant

## What This Is
A personal AI agent that runs on Harrison's Mac, chatted with via Telegram, capable of:
- Full desktop control (screenshots, mouse, keyboard) via Anthropic Computer Use API
- Spawning and managing Claude Code sessions for coding tasks
- Running shell commands directly
- Maintaining persistent memory (SQLite) across all interactions
- Automatically rotating Claude Code sessions when context gets stale

## Architecture

```
Harrison (Telegram) → operator_agent.py → routes to:
  ├── modules/computer_use.py    — Visual desktop control (Anthropic Computer Use API)
  ├── modules/claude_code_manager.py — Claude Code session lifecycle
  ├── modules/shell.py           — Direct terminal commands
  ├── modules/memory.py          — SQLite persistent memory
  └── Direct reasoning           — Anthropic API for conversation/planning
```

## Key Files
- `operator_agent.py` — Main orchestrator, Telegram bot, CLI test mode, message routing
- `modules/computer_use.py` — Screenshot + mouse/keyboard via Computer Use API
- `modules/claude_code_manager.py` — Spawn/rotate/manage Claude Code sessions
- `modules/shell.py` — Safe shell execution with confirmation for destructive ops
- `modules/memory.py` — SQLite: facts, sessions, interactions, tasks
- `config.py` — All configuration (API keys, paths, thresholds)
- `setup.sh` — Mac setup script (venv, deps, cliclick, permissions)
- `requirements.txt` — Python dependencies

## How It Works

### Message Routing
When Harrison sends a message, the agent uses Claude Haiku to classify it:
1. **Computer Use** → Visual tasks (open app, click button, navigate UI)
2. **Claude Code** → Coding tasks (write code, debug, complex analysis)
3. **Shell** → Quick terminal commands (git, ls, process management)
4. **Direct** → Conversation, questions, planning
5. **Memory** → Store/retrieve facts

### Claude Code Session Management
- Sessions auto-rotate after 40 turns (configurable in config.py)
- Memory context is injected into each new session
- Session summaries are saved for carry-forward
- Can open interactive Terminal sessions for hands-on work

### Memory System
SQLite database at `state/memory.db` with:
- **Facts** — Key-value knowledge store with categories
- **Sessions** — Claude Code session logs with summaries
- **Interactions** — Full conversation history
- **Tasks** — Persistent task queue

### Computer Use
Uses Anthropic's Computer Use API (computer_20250124):
- Screenshots via macOS `screencapture`
- Mouse/keyboard via `cliclick` (brew install) or AppleScript fallback
- Full agentic loop: screenshot → analyze → act → repeat
- Safety: max 20 steps per task

### Safety
- Destructive shell commands require /confirm
- Blocked commands list (rm -rf /, etc.)
- Only responds to Harrison's Telegram chat ID
- Computer use has step limit

## Setup
```bash
cd operator
bash setup.sh
source .venv/bin/activate
export ANTHROPIC_API_KEY='sk-ant-...'
export TELEGRAM_BOT_TOKEN='...'
python operator_agent.py --test   # CLI mode
python operator_agent.py          # Telegram mode
```

## Telegram Commands
- `/status` — System status
- `/memory [search]` — View/search memory
- `/remember key: value` — Store a fact
- `/tasks` — View pending tasks
- `/screen` — Describe what's on screen
- `/shell command` — Run terminal command
- `/claude task` — Run Claude Code task
- `/terminal task` — Open Claude Code in Terminal
- `/sessions` — Claude Code session status

## Environment Variables
- `ANTHROPIC_API_KEY` — Required for all AI features
- `TELEGRAM_BOT_TOKEN` — Required for Telegram mode
- `CLAUDE_CODE_BIN` — Path to claude binary (default: "claude")
- `CLAUDE_CODE_WORKDIR` — Working directory for Claude Code sessions

## Integration with Willis Holdings Conglomerate
This agent can serve as an upgraded CEO interface — instead of just receiving briefings,
Harrison can now issue commands that the agent routes to the appropriate division tools.
The conglomerate's existing Telegram bot (ceo/telegram_bot.py) can be merged into this
agent as a briefing source.
