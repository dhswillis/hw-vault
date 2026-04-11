# ARCHITECTURE - Harrison Willis Trading System
## Last Updated: February 10, 2026

---

## System Design: Option 4 - Hybrid Full Stack

### Overview
Three-layer architecture: Knowledge (persistent files), Intelligence (Claude API), Execution (Quantower + Replikanto).

```
┌─────────────────────────────────────────────────────┐
│                    HARRISON (Telegram)               │
│              Directs, reviews, approves              │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│               OPENCLAW AGENT (VPS)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Telegram  │  │ Heartbeat│  │  Persistent Mem  │  │
│  │ Interface │  │ Scheduler│  │  (Mem0 + Qdrant) │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                      │                               │
│  ┌───────────────────▼──────────────────────────┐   │
│  │           GENIUS-BRAIN SKILL                  │   │
│  │  Reads knowledge base → Calls Claude API      │   │
│  │  with full system prompt (all strategies)     │   │
│  └───────────────────┬──────────────────────────┘   │
│                      │                               │
│  ┌───────────────────▼──────────────────────────┐   │
│  │          KNOWLEDGE BASE (Disk)                │   │
│  │  KNOWLEDGE_BASE.md  │  docs/brains/*.md       │   │
│  │  context/           │  strategies/            │   │
│  │  backtest/          │  execution/             │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
┌──────────────┐ ┌────────────┐ ┌────────────────┐
│ QuantConnect │ │ Quantower  │ │  Prop Firm     │
│ (Backtest)   │ │ (Signals)  │ │  Accounts x20  │
│ API only     │ │            │ │  (Replikanto)  │
└──────────────┘ └────────────┘ └────────────────┘
```

---

## Layer 1: Knowledge Base (Persistent Files)

Strategy knowledge lives as markdown files on the VPS disk. These files NEVER reset. They are the single source of truth.

### File Responsibilities

| File | Purpose | Updated By |
|------|---------|------------|
| `KNOWLEDGE_BASE.md` | Master reference: all rules, state, context | Manually + OpenClaw |
| `docs/brains/TEMPO.md` | Full Tempo strategy rules (655 lines) | Manually via Git |
| `docs/brains/LANTO.md` | Lanto strategy rules | TBD |
| `docs/brains/CARMINE.md` | Carmine strategy rules | TBD |
| `docs/brains/OMEGA.md` | Omega strategy rules | TBD |
| `docs/brains/FLIPPING_MARKETS.md` | Flipping Markets rules | TBD |
| `docs/ARCHITECTURE.md` | This file - system design | Manually via Git |
| `context/daily.json` | Daily session levels, VIX, bias | Heartbeat (pre-market) |
| `context/signals.json` | Active trade signals | Real-time monitoring |
| `strategies/main_v34.py` | QuantConnect algorithm code | Dev via Git |
| `backtest/results/` | Backtest results archive | After each QC run |
| `execution/trades.log` | Live trade log | Execution engine |
| `logs/` | Session logs, errors | Automatic |

### Context Variables (Updated Daily)
The `context/daily.json` file is rebuilt every pre-market session:

```json
{
  "date": "2026-02-10",
  "vix": 18.5,
  "regime": "normal",
  "bias": "bullish",
  "levels": {
    "london_high": 21450,
    "london_low": 21380,
    "asia_high": 21420,
    "asia_low": 21395,
    "prev_day_high": 21475,
    "prev_day_low": 21350,
    "prev_day_close": 21410,
    "fifty_pct": 21412
  },
  "news": [
    {"time": "08:30", "event": "CPI", "impact": "high"}
  ],
  "draw_on_liquidity": {
    "upside": [21475, 21500],
    "downside": [21350, 21300]
  }
}
```

---

## Layer 2: Intelligence (Genius-Brain)

The `genius-brain.js` script is a Claude API wrapper that injects the full knowledge base as system prompt. This gives Claude complete strategy context on every call.

### How It Works
1. Reads `KNOWLEDGE_BASE.md` (~5K tokens)
2. Reads all `docs/brains/*.md` files (~30K tokens for Tempo alone)
3. Optionally reads `context/daily.json` for real-time context
4. Concatenates into system prompt
5. Sends question + system prompt to Claude API
6. Returns response

### Cost Optimization
- Anthropic automatically caches repeated system prompts
- Cached tokens cost ~90% less
- First call: ~$0.05 (35K tokens system prompt)
- Subsequent calls: ~$0.005 (cached)
- Estimated daily cost: $1-3 (20-50 calls/day)

### Model Selection
- **claude-sonnet-4-5-20250929**: Default. Fast, cheap, good for most analysis
- **claude-opus-4-5-20251101**: For complex multi-strategy decisions or debugging
- Selection can be automatic based on question complexity

---

## Layer 3: Execution (Future)

### Phase 1: Signal Generation (Current Target)
- OpenClaw monitors market data
- Genius-brain evaluates setups against strategy rules
- Valid signals posted to Telegram
- Harrison manually executes or approves

### Phase 2: Semi-Automated
- Signals written to `execution/signals.json`
- Quantower reads signal files
- Harrison confirms via Telegram before execution

### Phase 3: Full Automation
- Signals auto-execute via Quantower API
- Replikanto copies to 15-20 prop firm accounts
- Real-time P&L tracking
- Circuit breakers: 2-loss daily limit, max drawdown kill switch

---

## Development Workflow

```
Mac (Cowork/Claude Code)          GitHub              VPS (OpenClaw)
        │                           │                       │
        │  git push                 │                       │
        ├──────────────────────────►│                       │
        │                           │   git pull            │
        │                           ├──────────────────────►│
        │                           │                       │
        │                           │   Or: Telegram msg    │
        │                           │   "Pull latest"       │
        │                           ├──────────────────────►│
        │                           │                       │
        │  Telegram reports         │                       │
        │◄──────────────────────────┼───────────────────────┤
        │                           │                       │
```

- **Mac**: Strategy development, backtesting iteration, code editing
- **GitHub**: Sync bridge, version control, collaboration
- **VPS**: Production execution, monitoring, persistent knowledge

---

## QuantConnect Integration (Backtesting Only)

### API Details
- **Project ID**: 28083727
- **Auth**: SHA256(api_token + ":" + timestamp), Basic auth header
- **Base URL**: https://www.quantconnect.com/api/v2/
- **File size limit**: 64,000 characters per file

### Endpoints Used
| Endpoint | Purpose |
|----------|---------|
| `files/update` | Upload strategy code |
| `compile/create` | Trigger compilation |
| `compile/read` | Check compile status |
| `backtests/create` | Launch backtest |
| `backtests/read` | Get backtest results |

### Automated Backtest Workflow
1. Push code changes via Git
2. OpenClaw detects changes (heartbeat check or Telegram command)
3. genius-brain reads code, compresses if needed
4. Uploads to QC via API
5. Compiles and launches backtest
6. Polls for results
7. Analyzes results against targets
8. Reports via Telegram
9. Logs results to `backtest/results/`

---

## Infrastructure

### VPS Requirements
- **Provider**: QuantVPS Pro
- **Specs**: 4 cores, 8GB RAM
- **Location**: Chicago (close to CME for future live execution)
- **OS**: Ubuntu 22.04

### Services Running
| Service | Port | Purpose |
|---------|------|---------|
| OpenClaw Gateway | - | Main agent process |
| Qdrant | 6333 | Vector database for Mem0 memory |
| Docker | - | Container runtime for Qdrant |

### Monthly Cost
| Item | Cost |
|------|------|
| QuantVPS Pro | $70/mo |
| Anthropic API (Sonnet + caching) | ~$50/mo |
| OpenAI Embeddings (Mem0) | ~$0.02/mo |
| **Total New Cost** | **~$120/mo** |
