# Willis Holdings Brain — Local Setup

Everything runs on your Mac. No data leaves your machine.

## Prerequisites

You already have these:
- **Ollama** — installed with `llama3.1:8b` and `llama3.2:latest`
- **Python 3** — comes with macOS
- **Telegram bot** — already configured (token in .local/config.json)

## Start the Brain (one command)

```bash
cd ~/Documents/trading-system

# Set your Telegram bot credentials
export TELEGRAM_BOT_TOKEN="8389410384:AAHub-1hd82qTMsaSkbRJ7eqeHV28Kl7UvQ"
export TELEGRAM_CHAT_ID="451811002"
export TRADING_DIR="$(pwd)"
export OLLAMA_MODEL="llama3.1:8b"

# Make sure Ollama is running (it should be — it starts with your Mac)
ollama list

# Start the brain
python3 -m conglomerate.brain.daemon
```

That's it. The brain will:
1. Connect to your local Ollama (llama3.1:8b)
2. Start listening for your Telegram commands
3. Run scheduled briefings (morning 7AM UTC, EOD 5PM UTC)
4. Process task queue in the background

## Talk to It via Telegram

Send any message to your Telegram bot. It routes automatically:
- Quick questions → Llama answers locally
- Strategy/analysis → Deep local analysis via Llama
- Complex coding → Queued for Claude Code
- Big decisions → Asks you to handle manually

### Commands

| Command | What it does |
|---------|-------------|
| `/status` | Brain health, disk, memory |
| `/think <question>` | Quick local analysis |
| `/strategic <question>` | Deep strategic reasoning (local) |
| `/swarm <task>` | Multi-agent decomposition (local) |
| `/security` | Run data integrity + security audit |
| `/mining` | Latest batch/mining results |
| `/tasks` | Task queue status |
| `/brief` | Force a morning briefing |
| `/spawn <name> <purpose>` | Design a new division agent |
| `/evolve <division>` | Self-improve a division |
| `/learn` | Show what the brain has learned |
| `/run <task>` | Queue a task for execution |
| `/stop` | Shut down the brain |

## Security Model

- **Llama (Ollama)** — Runs on your Mac. Zero network calls. All proprietary trading data, signals, backtests, and alpha analysis go through this. Nothing leaves your machine.
- **Claude Code** — Uses Anthropic API for complex coding only. Anthropic's API policy: they don't train on your data.
- **Telegram** — Only sends briefing summaries (no raw data, no signals, no alpha). You control what you ask.
- **No Kimi, no OpenRouter, no GPT** — No third-party model APIs touch your proprietary data.

## Upgrading the Local Model

Your Mac can run bigger models for smarter analysis:

```bash
# If you have 16GB RAM (good enough):
ollama pull llama3.1:8b          # Already installed

# If you have 32GB RAM (much smarter):
ollama pull llama3.1:70b
export OLLAMA_MODEL="llama3.1:70b"

# If you have 64GB+ RAM (near GPT-4 level):
ollama pull llama3.1:70b         # Same model, just runs faster with more RAM
```

The 8b model is fast but basic. The 70b model is significantly smarter — better at strategy reasoning, pattern recognition, and complex analysis. If you've got the RAM, use it.

## Run as Background Service (optional)

To keep the brain running even when you close Terminal:

```bash
# Start in background
nohup python3 -m conglomerate.brain.daemon > brain.log 2>&1 &

# Check if running
ps aux | grep daemon

# Stop it
kill $(ps aux | grep 'conglomerate.brain.daemon' | grep -v grep | awk '{print $2}')
```

## Test Components

```bash
python3 -m conglomerate.brain.daemon --test
```

## Troubleshooting

**"Ollama not reachable"** — Run `ollama serve` in another terminal, or open the Ollama app.

**"Model not found"** — Run `ollama list` to see what's installed, then `ollama pull llama3.1:8b` if needed.

**Telegram not responding** — Check your bot token is correct. Open Telegram, search for your bot, send `/start`.
