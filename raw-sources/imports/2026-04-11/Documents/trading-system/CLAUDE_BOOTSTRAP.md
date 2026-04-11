# Tempo Trading System — Claude Session Bootstrap

> **Read this file first at the start of every session.**
> It tells you who Harrison is, what this project does, where credentials live, and how to get productive immediately.

## Quick Start Checklist
1. Read this file (you're doing that now)
2. Read `CONTEXT.md` for strategy details and architecture
3. Load credentials from `.local/config.json` (gitignored, never commit)
4. Clone or pull latest from `dhswillis/trading-system` using the GitHub PAT in config
5. Check `CONTEXT.md` "Known Issues / TODO" for current priorities

## Who Is Harrison
- Trader, not a developer. Handle: @prop_profitable
- Building an autonomous NQ futures backtesting pipeline
- Communicates casually — keep responses practical, not academic
- Cares deeply about realistic execution modeling (slippage, fees, fill quality)

## Credentials Location
All secrets are in `.local/config.json` (gitignored):
```json
{
  "quantconnect": {
    "user_id": "...",
    "api_token": "...",
    "project_id": 28083727
  },
  "github": {
    "repo": "dhswillis/trading-system",
    "token": "github_pat_..."
  },
  "broker": {
    "commission_round_trip": 4.50,
    "instrument": "NQ",
    "point_value": 20.0
  }
}
```

### Using GitHub Token
To clone/pull/push:
```bash
# Read token from config
TOKEN=$(python3 -c "import json; print(json.load(open('.local/config.json'))['github']['token'])")
git clone https://x-access-token:${TOKEN}@github.com/dhswillis/trading-system.git
# Or set remote on existing clone:
git remote set-url origin https://x-access-token:${TOKEN}@github.com/dhswillis/trading-system.git
```

### Using QuantConnect API
```bash
USER_ID=$(python3 -c "import json; print(json.load(open('.local/config.json'))['quantconnect']['user_id'])")
API_TOKEN=$(python3 -c "import json; print(json.load(open('.local/config.json'))['quantconnect']['api_token'])")
# Auth test:
curl -u "${USER_ID}:${API_TOKEN}" https://www.quantconnect.com/api/v2/authenticate
```

## Project Structure
```
trading-system/
├── CLAUDE_BOOTSTRAP.md    ← You are here
├── CONTEXT.md             ← Strategy details, architecture, batch results
├── .local/                ← Credentials (gitignored)
│   └── config.json
├── .gitignore
├── batch-runner/
│   ├── main_v4.py         ← The QC strategy (Tempo V4 Modular Brain)
│   ├── generate_variants.py
│   ├── run_twostage.py    ← Two-stage batch runner
│   ├── strategy_config.py ← Default config template
│   └── variants_batch2_2000.json
└── signals/               ← Runtime signal data (gitignored)
```

## Key Strategy Parameters (main_v4.py Initialize)
These were added Feb 2026 for realistic execution modeling:
- `SLIPPAGE_PTS = 0.25` — 1 NQ tick per side, applied adversely
- `COMMISSION_PER_CONTRACT = 4.50` — Round-trip, deducted at exit
- `TICK_THROUGH_PTS = 0.25` — Price must trade 1 tick past FVG entry for fill
- `BE_COST_OFFSET_PTS = 0.50` — BE stop set 2 ticks above entry to cover exit costs ($9.50/contract)

### Why BE Offset Matters
With 0.25R breakeven mode and a high number of BE exits, every "breakeven" exit actually costs ~$9.50/contract (slippage $5 + commission $4.50). At $20/point for NQ, that's 0.475 pts. We round up to 0.50 pts (2 ticks) so the BE stop is set above entry, ensuring true breakeven after costs.

## Batch Running Workflow
1. Generate variants: `python3 generate_variants.py`
2. Upload strategy to QC: Uses QC API (credentials in config)
3. Run two-stage batch: `python3 run_twostage.py`
   - Stage A: 3-month screen (Jul–Sep 2025)
   - Stage B: 12-month confirmation on winners
4. Results → Excel spreadsheet for analysis

## Current Status (as of Feb 14, 2026)
- Batch 2 (no costs): 2000 variants, 842 completed Stage A, 290 winners
- Batch 3 (with costs): RUNNING NOW on VPS — cost modeling active (slippage, fees, tick-through, BE offset)
  - Log: `~/batch3_costmodel.log` on VPS
  - Results dir: `~/trading-system/batch-runner/results/batch3_costmodel/`
  - Check progress: `tail -20 ~/batch3_costmodel.log`
- Code pushed to GitHub with all cost modeling changes
- NEXT: When Batch 3 finishes → build context engine v1 (day-type classification + volume features)
- NEXT: Complete OpenClaw tempo-trading skill for Telegram-based monitoring
- FUTURE: Databento tick data integration for footprint/order flow features

## Built Integrations

### OpenClaw / Telegram Bot
- **Status**: Configured, skill incomplete
- **Bot token**: `8389410384:AAHub-1hd82qTMsaSkbRJ7eqeHV28Kl7UvQ`
- **Harrison's Telegram ID**: `451811002`
- **Model**: `anthropic/claude-opus-4-6`
- **Gateway port**: 18789
- **Config files on Mac**:
  - `~/Downloads/openclaw.json` — main OpenClaw config
  - `~/Downloads/telegram-allowFrom.json` — Telegram allowlist
  - `~/Documents/trading-system/openclaw-skills/` — skill directory (empty, tempo-trading skill was started but not completed)
  - `~/Documents/trading-system/.openclaw/` — local OpenClaw config directory
- **What was built**: Telegram bot connected to Claude via OpenClaw, DM-only access restricted to Harrison
- **What was NOT finished**: The `tempo-trading` skill (custom OpenClaw skill for checking batch results, VPS status, etc.)
- **To restart**: OpenClaw needs to be running as a process — check if it was installed via npm/pip on Mac or VPS
- **Vision**: Use Telegram to check batch progress, get morning summaries, trigger reruns from phone

### Batch Runner (VPS)
- **VPS**: QuantVPS Pro at `172.93.181.209` (root SSH)
- **Nightly cron**: 10 PM CT, runs `nightly_batch.sh`
- **Code location on VPS**: `~/trading-system/batch-runner/`
- **Credentials on VPS**: `/root/.env` (QC_USER_ID, QC_API_TOKEN, QC_PROJECT_ID)
- **Databento key**: Saved (not yet integrated)

## GitHub Token Expiry
- Token name: `cowork-tempo`
- Created: Feb 14, 2026
- Expires: May 15, 2026
- If expired, Harrison needs to generate a new fine-grained PAT at:
  https://github.com/settings/personal-access-tokens/new
  Permissions needed: Contents (Read and write), All repositories
