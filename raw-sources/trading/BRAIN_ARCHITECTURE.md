# TEMPO Trading System Brain Architecture

## Ralph Loop + OpenClaw + Berman Trifecta Integration

---

## 1. High-Level Architecture

The trading system "brain" combines three architectural layers:

**Layer 1 — OpenClaw Gateway (The Nervous System)**
Self-hosted agent runtime that provides the always-on infrastructure: multi-channel messaging, tool execution, memory persistence, and model routing. This is the 24/7 backbone.

**Layer 2 — Ralph Loop (The Reasoning Engine)**
Autonomous loop architecture that breaks complex trading tasks into atomic units, executes each in a fresh AI context, persists learnings via git + progress files, and iterates until acceptance criteria pass. This handles strategy development, backtesting, and system evolution.

**Layer 3 — Berman Trifecta (The Model Stack)**
OpenClaw + Opus 4.6 + GPT-5.3 Codex working in concert — Opus for deep reasoning/strategy design, GPT Codex for code generation/execution, and OpenClaw as the orchestrator routing tasks to the right model.

---

## 2. System Topology

```
┌─────────────────────────────────────────────────────────┐
│                    YOU (Harrison)                         │
│              WhatsApp / Telegram / Discord                │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              OPENCLAW GATEWAY (:18789)                    │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐   │
│  │ Channels │  │  Memory  │  │    Model Router      │   │
│  │ WhatsApp │  │ Long-term│  │ Opus 4.6 (reasoning) │   │
│  │ Telegram │  │ Daily    │  │ GPT-5.3 (codex)      │   │
│  │ Discord  │  │ Sessions │  │ Gemini 3 (fallback)  │   │
│  └──────────┘  └──────────┘  └──────────────────────┘   │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │              AGENT WORKSPACES                     │    │
│  │                                                    │    │
│  │  Agent: STRATEGIST    Agent: EXECUTOR              │    │
│  │  (research, design)   (backtest, deploy)           │    │
│  │                                                    │    │
│  │  Agent: MONITOR       Agent: RISK                  │    │
│  │  (live tracking)      (drawdown, limits)           │    │
│  └──────────────────────────────────────────────────┘    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                RALPH LOOP ENGINE                         │
│                                                           │
│  ┌─────────┐    ┌──────────┐    ┌───────────────┐       │
│  │ prd.json│───▶│ Fresh AI │───▶│ Quality Gates │       │
│  │ (tasks) │    │ Instance │    │ (tests/checks)│       │
│  └─────────┘    └──────────┘    └───────┬───────┘       │
│       ▲                                  │               │
│       │         ┌──────────────┐         │               │
│       └─────────│ progress.txt │◀────────┘               │
│                 │ (learnings)  │                          │
│                 └──────────────┘                          │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              QUANTCONNECT EXECUTION LAYER                │
│                                                           │
│  ┌───────────┐  ┌───────────┐  ┌────────────────┐      │
│  │ main_v4.py│  │ Batch     │  │ Live Trading   │      │
│  │ (strategy)│  │ Runner    │  │ Engine         │      │
│  └───────────┘  └───────────┘  └────────────────┘      │
└─────────────────────────────────────────────────────────┘
```

---

## 3. The Ralph Loop Applied to Trading

### 3.1 Why Ralph Loop for Trading

Traditional trading system development suffers from the exact problems Ralph solves:

- **Context window drift**: A single AI session trying to analyze data, design strategies, write backtest code, run tests, interpret results, and iterate degrades over time. Each new question builds on potentially hallucinated prior context.
- **No feedback loop**: Most AI-assisted trading work is "generate strategy → hope it works." Ralph forces test-driven iteration: every strategy must pass quantified acceptance criteria.
- **No persistent memory**: Insights from one session are lost in the next. Ralph's `progress.txt` + `AGENTS.md` pattern preserves learnings across iterations.

### 3.2 Trading-Specific PRD Format

```json
{
  "branchName": "tempo-v5-strategy-refinement",
  "tradingContext": {
    "system": "TEMPO",
    "instruments": ["ES", "NQ"],
    "dataSource": "databento",
    "backtestPlatform": "QuantConnect",
    "currentBestSharpe": 2.571,
    "currentBestVariant": "v4_combo_0056"
  },
  "userStories": [
    {
      "id": "slippage-audit",
      "title": "Add realistic slippage model to main_v4.py",
      "acceptanceCriteria": [
        "ConstantSlippageModel with 2 ticks per side applied",
        "All 50 Stage B variants re-tested with slippage",
        "New Sharpe ratios documented in results",
        "Top variant still profitable after slippage"
      ],
      "passes": false,
      "priority": 1,
      "estimatedR": "Critical — determines if system is viable"
    },
    {
      "id": "limit-fill-model",
      "title": "Replace ImmediateFillModel with trade-through requirement",
      "acceptanceCriteria": [
        "Custom fill model requires price to trade 1 tick through limit",
        "FVG limit entries only fill when bid/ask crosses the level",
        "Re-run Stage B with new fill model",
        "Document fill rate change (expected: 10-30% fewer fills)"
      ],
      "passes": false,
      "priority": 2
    },
    {
      "id": "repaint-audit",
      "title": "Verify no signal repainting in FVG detection",
      "acceptanceCriteria": [
        "FVG confirmed only after 3rd candle closes",
        "Entry orders placed on next bar after confirmation",
        "No current-bar OHLC used for same-bar fills",
        "Audit log added showing signal_time vs fill_time"
      ],
      "passes": false,
      "priority": 3
    },
    {
      "id": "look-ahead-audit",
      "title": "Eliminate forward-looking bias in day classification",
      "acceptanceCriteria": [
        "Inside/outside day determined ONLY after daily close",
        "Next-day trades use prior-day classification",
        "Session filter uses real-time clock, not retroactive tagging",
        "Gap calculations use only pre-market data"
      ],
      "passes": false,
      "priority": 4
    },
    {
      "id": "carmine-lvn-signal",
      "title": "Implement LVN detection using volume profile",
      "acceptanceCriteria": [
        "Volume profile computed on 15M bars",
        "LVN zones identified where volume < 30th percentile",
        "FVG + LVN alignment scored as confluence factor",
        "Backtest shows LVN-aligned trades outperform non-aligned"
      ],
      "passes": false,
      "priority": 5
    },
    {
      "id": "confluence-scorer",
      "title": "Build real-time confluence scoring engine",
      "acceptanceCriteria": [
        "All 8 TEMPO factors + 4 Carmine factors scored in real-time",
        "Minimum 60-point threshold enforced for entry",
        "Score logged per trade for post-analysis",
        "Backtest with scoring filter shows 40%+ WR improvement"
      ],
      "passes": false,
      "priority": 6
    },
    {
      "id": "portfolio-constructor",
      "title": "Build 4 sub-strategy portfolio with session separation",
      "acceptanceCriteria": [
        "Sub-A (Overlap), Sub-B (Lunch), Sub-C (LVN), Sub-D (Wide Net) all running",
        "Session isolation verified — no time overlap between subs",
        "Portfolio-level drawdown < max individual DD",
        "Combined daily R > 8R on backtest"
      ],
      "passes": false,
      "priority": 7
    },
    {
      "id": "weekly-profitability",
      "title": "Validate all-weeks-profitable on 12-month backtest",
      "acceptanceCriteria": [
        "Run portfolio across full 12-month window",
        "Calculate weekly P&L for all 52 weeks",
        "Document any losing weeks and root cause",
        "If losing weeks exist, adjust sub-strategy allocation to fix"
      ],
      "passes": false,
      "priority": 8
    }
  ]
}
```

### 3.3 The Trading Ralph Loop

```bash
#!/bin/bash
# trading_ralph.sh — Ralph Loop for TEMPO Trading System
# Each iteration: pick one story, implement, test, commit, learn

MAX_ITERATIONS=${1:-50}
TOOL=${2:-claude}  # claude or codex

for i in $(seq 1 $MAX_ITERATIONS); do
  echo "=== RALPH ITERATION $i / $MAX_ITERATIONS ==="

  # Check if all stories pass
  ALL_PASS=$(python3 -c "
import json
prd = json.load(open('prd.json'))
print('COMPLETE' if all(s['passes'] for s in prd['userStories']) else 'CONTINUE')
  ")

  if [ "$ALL_PASS" = "COMPLETE" ]; then
    echo "<promise>COMPLETE</promise>"
    echo "All trading stories pass. System ready."
    exit 0
  fi

  # Pick highest priority incomplete story
  STORY=$(python3 -c "
import json
prd = json.load(open('prd.json'))
incomplete = [s for s in prd['userStories'] if not s['passes']]
incomplete.sort(key=lambda x: x['priority'])
print(json.dumps(incomplete[0]))
  ")

  echo "Working on: $STORY"

  # Spawn fresh AI instance with clean context
  if [ "$TOOL" = "claude" ]; then
    claude code --prompt "
You are working on the TEMPO trading system.

CURRENT TASK (from prd.json):
$STORY

LEARNINGS FROM PREVIOUS ITERATIONS:
$(cat progress.txt 2>/dev/null || echo 'No previous learnings.')

INSTRUCTIONS:
1. Read the codebase to understand current state
2. Implement ONLY the acceptance criteria for this story
3. Write/update tests to verify each criterion
4. Run tests and fix any failures
5. If all criteria pass, update prd.json setting passes: true
6. Append any learnings to progress.txt
7. Commit with message: '[ralph] story: <id> — <title>'
" --allowedTools "Bash,Read,Write,Edit"
  fi

  # Post-iteration: verify tests still pass
  python3 -m pytest tests/ --tb=short
  if [ $? -ne 0 ]; then
    echo "WARNING: Tests failed after iteration $i"
    echo "Iteration $i: Tests failed. Investigate before next iteration." >> progress.txt
  fi

  # Commit progress
  git add -A && git commit -m "[ralph] iteration $i complete" --allow-empty

  echo "=== ITERATION $i COMPLETE ==="
  sleep 2
done

echo "WARNING: Max iterations reached without completion"
```

---

## 4. OpenClaw Agent Configuration

### 4.1 Agent Definitions

Four specialized agents running in OpenClaw, each with isolated sessions:

```json
{
  "agents": {
    "strategist": {
      "model": "anthropic/claude-opus-4-6",
      "systemPrompt": "You are the TEMPO trading strategist. You analyze backtest data, design new strategy configurations, optimize parameters, and integrate Carmine Rosato order flow concepts. You think deeply about market structure, confluences, and edge. You NEVER write code — you design strategies and hand specifications to the executor agent.",
      "tools": ["web_search", "web_fetch", "file_read", "memory"],
      "memory": {
        "longTerm": true,
        "dailyContext": true
      },
      "channels": ["whatsapp", "telegram"]
    },
    "executor": {
      "model": "openai/gpt-5.3-codex",
      "systemPrompt": "You are the TEMPO code executor. You take strategy specifications from the strategist and implement them in Python on QuantConnect. You write main_v4.py modifications, batch runner configs, analysis scripts, and test suites. You follow the Ralph Loop pattern — one task per session, test-driven, commit and exit.",
      "tools": ["bash", "file_read", "file_write", "file_edit"],
      "memory": {
        "longTerm": false,
        "dailyContext": true
      }
    },
    "monitor": {
      "model": "anthropic/claude-opus-4-6",
      "systemPrompt": "You are the TEMPO live monitor. You watch for nightly batch results, analyze new hot zones, track live trade performance, and alert Harrison of significant events. You run on a cron schedule checking results every morning.",
      "tools": ["file_read", "web_search", "cron", "notify"],
      "cron": {
        "morningCheck": "0 7 * * 1-5",
        "nightlyAnalysis": "0 20 * * 1-5"
      }
    },
    "risk": {
      "model": "anthropic/claude-opus-4-6",
      "systemPrompt": "You are the TEMPO risk manager. You enforce drawdown limits, position sizing rules, and circuit breakers. You have veto power over any trade that exceeds risk parameters. You monitor portfolio-level exposure across all 4 sub-strategies.",
      "tools": ["file_read", "notify"],
      "alerts": {
        "dailyLoss3R": "Halt new entries 1 hour",
        "dailyLoss5R": "Halt remainder of day",
        "weeklyLoss15R": "Reduce size 50% next week",
        "monthlyLoss30R": "Pause 3 days, full review"
      }
    }
  }
}
```

### 4.2 Inter-Agent Communication Flow

```
Harrison (via WhatsApp/Telegram)
    │
    ▼
OpenClaw Gateway
    │
    ├── "Analyze last night's batch results"
    │       └── Routes to: STRATEGIST agent
    │           └── Strategist reads results, designs next batch config
    │           └── Hands spec to EXECUTOR via session handoff
    │
    ├── "Run the next batch with these parameters"
    │       └── Routes to: EXECUTOR agent
    │           └── Executor writes config, triggers QuantConnect batch
    │           └── MONITOR watches for completion
    │
    ├── "How did we do today?"
    │       └── Routes to: MONITOR agent
    │           └── Monitor summarizes daily P&L, trade count, R-multiple
    │           └── Flags any RISK alerts
    │
    └── "Are we within risk limits?"
            └── Routes to: RISK agent
                └── Risk checks portfolio exposure, circuit breakers
                └── Alerts if any thresholds approaching
```

---

## 5. The Berman Trifecta Applied

### 5.1 Model Routing Strategy

Following Berman's insight that different models excel at different tasks:

| Task Type | Model | Why |
|-----------|-------|-----|
| Strategy design & deep analysis | Opus 4.6 | Best at complex reasoning, multi-step logic, understanding market dynamics |
| Code generation & execution | GPT-5.3 Codex | Fastest code generation, best at QuantConnect API, strongest at debugging |
| Live monitoring & alerting | Opus 4.6 | Best at interpreting ambiguous data, contextual awareness |
| Risk calculations | Opus 4.6 | Most reliable for high-stakes numerical reasoning |
| Data parsing & formatting | GPT-5.3 Codex | Fastest at structured data manipulation |
| Research & web analysis | Opus 4.6 | Best at synthesizing information from multiple sources |

### 5.2 Model Handoff Pattern

```
[Strategy Request]
     │
     ▼
  Opus 4.6 (STRATEGIST)
  "Design a combo signal config targeting
   Sharpe > 2.0 on overlap session,
   outside days only, with LVN confluence"
     │
     ▼ (specification document)
     │
  GPT-5.3 Codex (EXECUTOR)
  "Implement this spec in main_v4.py,
   add to batch runner, write tests"
     │
     ▼ (code + test results)
     │
  Opus 4.6 (MONITOR)
  "Analyze backtest results, compare to
   prior best, flag any concerns"
     │
     ▼ (analysis + recommendations)
     │
  [Report to Harrison]
```

---

## 6. Addressing Backtesting Integrity (From Your Questions)

The Ralph Loop is PERFECTLY suited for addressing the backtesting issues we discussed. Here's how each concern maps to a Ralph story:

### 6.1 Limit Order Fill (Trade-Through Requirement)

**Ralph Story**: `limit-fill-model`

The fresh-context approach is critical here. A single long session trying to audit and fix fill models tends to introduce new bugs while fixing old ones (context drift). Ralph's approach: one fresh AI instance reads the current main_v4.py, identifies the fill model, replaces it with a trade-through model, runs the full backtest, and commits only if results are valid. If it fails, the next iteration reads the progress.txt note about what went wrong and tries a different approach.

### 6.2 Repainting Prevention

**Ralph Story**: `repaint-audit`

The acceptance criteria are binary: "FVG confirmed only after 3rd candle closes" and "entry orders placed on next bar after confirmation." The Ralph loop will keep iterating until a test specifically validates that no signal generates an order on the same bar that completed the FVG pattern.

### 6.3 Slippage Model

**Ralph Story**: `slippage-audit`

This is where Ralph's test-driven approach shines. The acceptance criterion "top variant still profitable after slippage" is a quantitative gate. The AI adds the slippage model, re-runs all 50 variants, checks if v4_combo_0056 still has positive Sharpe. If not, the story doesn't pass, and the next iteration either adjusts the strategy parameters or documents that the system isn't viable under realistic conditions — which is exactly what you need to know.

---

## 7. Implementation Roadmap

### Phase 1: OpenClaw Setup (Day 1)

```bash
# Install OpenClaw
curl -fsSL https://get.openclaw.ai | bash

# Configure with Opus 4.6 as primary model
openclaw config set model anthropic/claude-opus-4-6

# Set up WhatsApp/Telegram channel
openclaw channel add whatsapp
openclaw channel add telegram

# Create agent workspaces
openclaw agent create strategist --model anthropic/claude-opus-4-6
openclaw agent create executor --model openai/gpt-5.3-codex
openclaw agent create monitor --model anthropic/claude-opus-4-6
openclaw agent create risk --model anthropic/claude-opus-4-6
```

### Phase 2: Ralph Loop Integration (Day 2-3)

```bash
# Initialize Ralph in the trading-system repo
cd trading-system
mkdir -p scripts/ralph

# Create prd.json with trading stories (see Section 3.2)
# Create trading_ralph.sh (see Section 3.3)
# Create progress.txt (empty, will accumulate learnings)
# Create AGENTS.md with trading-specific conventions

# First run — start with the slippage audit
./scripts/ralph/trading_ralph.sh 10 claude
```

### Phase 3: Nightly Automation (Day 4-5)

```bash
# OpenClaw cron: run batch every night at 4 AM UTC
openclaw cron add "nightly-batch" \
  --schedule "0 4 * * 1-5" \
  --agent executor \
  --command "cd /root/trading-system && python3 batch-runner/run_batch.py"

# OpenClaw cron: analyze results every morning at 7 AM
openclaw cron add "morning-analysis" \
  --schedule "0 7 * * 1-5" \
  --agent monitor \
  --command "Analyze last night's batch results and send summary to Harrison"
```

### Phase 4: Live Trading Brain (Week 2+)

```bash
# OpenClaw skill: real-time confluence scoring
openclaw skill create confluence-scorer \
  --description "Real-time confluence scoring for TEMPO signals" \
  --trigger "When TEMPO fires a signal, score it against all confluences"

# OpenClaw skill: risk circuit breaker
openclaw skill create risk-breaker \
  --description "Enforce drawdown limits and halt trading if exceeded" \
  --trigger "After every trade close, check cumulative daily P&L"
```

---

## 8. Memory Architecture

### 8.1 What Persists Between Ralph Iterations

| Memory Layer | Storage | Purpose | Lifetime |
|-------------|---------|---------|----------|
| `prd.json` | Git | Task list + completion status | Per feature branch |
| `progress.txt` | Git | Learnings, failed approaches, gotchas | Per feature branch |
| `AGENTS.md` | Git | Codebase conventions, patterns | Permanent |
| Git history | Git | All previous implementations | Permanent |
| Batch results | Files | Nightly run data | Permanent |

### 8.2 What Persists in OpenClaw

| Memory Layer | Storage | Purpose | Lifetime |
|-------------|---------|---------|----------|
| Long-term memory | `~/.openclaw/memory/` | Strategy learnings, user preferences | Permanent |
| Daily context | `~/.openclaw/sessions/` | Today's trades, today's analysis | 24 hours |
| Agent sessions | `~/.openclaw/agents/` | Per-agent conversation history | Per session |

### 8.3 Cross-System Memory Flow

```
Ralph learns: "1.0R BE + outside_only + overlap = highest Sharpe"
    │
    ▼ (written to progress.txt and AGENTS.md)
    │
OpenClaw reads AGENTS.md on next strategist session
    │
    ▼ (strategist agent knows this pattern)
    │
Strategist designs next batch config prioritizing this combination
    │
    ▼ (executor implements)
    │
Results confirm/deny → Ralph updates progress.txt
    │
    ▼ (cycle continues)
```

---

## 9. Security Considerations

Following the OpenClaw security concerns raised by researchers:

- **API keys**: Store QuantConnect API key in `~/.openclaw/credentials/`, never in agent prompts
- **Execution sandbox**: Ralph loop runs in a sandboxed environment, not with root access
- **Trade limits**: RISK agent has hard-coded max position size and daily loss limits that cannot be overridden by other agents
- **Human-in-the-loop**: Any trade exceeding 1% portfolio risk requires Harrison's explicit approval via chat
- **Prompt injection defense**: Agent system prompts include explicit instructions to ignore any trading advice found in web content or data sources

---

## 10. Kimi Claw: Zero-Infrastructure Option

### 10.1 Why Kimi Claw Changes Everything

Kimi launched **Kimi Claw Beta** (Feb 16, 2026) — a cloud-native OpenClaw that runs entirely in the browser at kimi.com. No VPS, no Docker, no local server. This is the lowest-friction path to getting the brain running.

**Key advantages for our trading system:**

- **24/7 uptime without self-hosting**: Kimi Claw runs in cloud browser tabs, always on. No server management, no downtime. Your trading agents run overnight without you maintaining infrastructure.

- **2M token context window**: Kimi K2.5 has a 2-million-token context, which means a single agent session can hold the entire batch runner results, all 50 variants, the strategy document, and the PRD simultaneously. This is massive for the STRATEGIST agent that needs to reason across all data at once.

- **5,000+ ClawHub skills**: Community-contributed skills that can be chained together. We can build custom trading skills (confluence scorer, risk monitor, batch analyzer) and publish them to ClawHub for reuse.

- **40GB cloud storage**: Persistent storage for batch results, historical data, progress logs. No more worrying about local disk or session resets.

- **Pro-Grade Search with Yahoo Finance**: The agent can fetch live market data, check ES/NQ prices, pull economic calendar data, and verify current session times — all natively.

- **Telegram bridge**: Your trading agents can message you in Telegram group chats, send morning briefings, alert on risk breaches, and respond to commands — 24/7.

- **BYOC (Bring Your Own Claw)**: If you already have a self-hosted OpenClaw instance, you can connect it to Kimi's cloud interface. Best of both worlds.

### 10.2 Kimi Claw for the Trading Brain

The Kimi Claw deployment would look like:

```
┌─────────────────────────────────────────────────────────┐
│                 KIMI.COM (Browser)                        │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │           KIMI CLAW (Cloud OpenClaw)              │    │
│  │                                                    │    │
│  │  Model: Kimi K2.5 (1T params, 2M context)        │    │
│  │  + Opus 4.6 via BYOC for deep reasoning           │    │
│  │  + GPT-5.3 Codex via API for code execution       │    │
│  │                                                    │    │
│  │  Skills (from ClawHub + custom):                   │    │
│  │    - tempo-batch-analyzer                          │    │
│  │    - confluence-scorer                             │    │
│  │    - risk-monitor                                  │    │
│  │    - morning-briefing                              │    │
│  │    - strategy-optimizer                            │    │
│  │                                                    │    │
│  │  Storage: 40GB cloud (batch results, history)      │    │
│  │  Search: Pro-Grade (Yahoo Finance, econ calendar)  │    │
│  └──────────────────────────────────────────────────┘    │
│                         │                                 │
│                         ▼                                 │
│              Telegram Bridge → Harrison's phone           │
└─────────────────────────────────────────────────────────┘
```

### 10.3 Kimi K2.5 Advantages for Trading

- **Mixture-of-Experts (MoE)**: 1T total params, 32B active per inference. This means fast responses despite massive model size. Good for real-time monitoring.
- **Native tool use**: K2.5 was trained specifically for agentic tool calling. It can chain skills (fetch data → analyze → score → alert) more reliably than prompt-engineered chains.
- **Visual understanding**: K2.5 can analyze chart screenshots, interpret candlestick patterns visually, and reason about order flow heatmap images. This enables Carmine's visual order flow analysis to be automated.
- **100 parallel sub-agents**: K2.5 supports up to 100 parallel sub-agents. This means the STRATEGIST could spawn 50 sub-agents to analyze 50 variants simultaneously, then aggregate results.

### 10.4 Self-Hosted vs Kimi Claw Decision Matrix

| Factor | Self-Hosted OpenClaw | Kimi Claw |
|--------|---------------------|-----------|
| Setup effort | High (VPS, Docker, config) | Zero (browser login) |
| Cost | $7-20/mo VPS + API costs | Kimi membership + API costs |
| Uptime | You manage it | 24/7 managed |
| Context window | Model-dependent (200k for Opus) | 2M tokens (K2.5) |
| Model flexibility | Any model via API | K2.5 native + BYOC for others |
| Storage | Local/VPS disk | 40GB cloud |
| Security | Full control | Cloud provider trust |
| Customization | Unlimited | ClawHub skills + some config |
| Live data | Requires MCP setup | Pro-Grade Search built-in |
| Trading recommendation | Best for live execution (local = low latency) | Best for strategy/analysis/monitoring |

### 10.5 Hybrid Architecture (Recommended)

Use BOTH:

- **Kimi Claw** for: Strategy design (STRATEGIST agent), overnight analysis, morning briefings, research, and monitoring. The 2M context + visual understanding + Pro-Grade Search makes this ideal for the "thinking" layer.

- **Self-hosted OpenClaw** for: Live trade execution (EXECUTOR agent), risk management (RISK agent), and anything touching real money. Keep execution local for low latency and full control.

```
Kimi Claw (cloud)                Self-Hosted OpenClaw (local)
├── STRATEGIST agent             ├── EXECUTOR agent
├── MONITOR agent                ├── RISK agent
├── Research & analysis          ├── Live QuantConnect connection
├── Morning briefings            ├── Trade execution
└── Telegram notifications       └── Position management
         │                                │
         └──── Specs & configs ──────────▶│
         │◀─── Results & P&L ────────────┘
```

---

## 11. Mapping to Existing Infrastructure (Discovered Feb 17)

After full repo analysis, the brain architecture maps to EXISTING code:

### 11.1 What Already Exists (Don't Rebuild)

| Component | Location | Status | Brain Role |
|-----------|----------|--------|------------|
| Context Engine V2 | `context-engine/context_engine_v2.py` | BUILT (68.6% accuracy) | STRATEGIST's session classifier |
| Volume Profile | `context-engine/features/volume_profile.py` | BUILT | LVN detection for Carmine integration |
| Footprint Analysis | `context-engine/features/footprint.py` | BUILT | Order flow confluence scoring |
| SMT Divergence | `context-engine/features/smt.py` | BUILT | Cross-asset confirmation |
| Bayesian Classifier | `context-engine/features/bayesian_classifier.py` | BUILT | Session type prediction |
| Markov Model | `context-engine/features/markov_model.py` | BUILT | Session transition forecasting |
| Strategy Bridge | `context-engine/strategy_bridge.py` | BUILT | V2 output → StrategyConfig overrides |
| Live Processor | `context-engine/live_processor.py` | BUILT | Real-time tick processing |
| Batch Runner | `batch-runner/scripts/run_twostage.py` | BUILT | Two-stage variant screening |
| 2000 Variant Generator | `batch-runner/generate_variants.py` | BUILT | V4 signal-aware variant creation |
| Nightly Cron | `batch-runner/scripts/nightly_batch.sh` | RUNNING | 10 PM CT on VPS |
| VPS Deploy | `batch-runner/scripts/deploy_to_vps.sh` | BUILT | Code deployment |
| OpenClaw Config | `~/Downloads/openclaw.json` | CONFIGURED | Telegram bot interface |
| Databento Data | `312 files, ~1.6GB` | PURCHASED | Tick data for context engine |

### 11.2 What Needs Building (New)

| Component | Purpose | Priority |
|-----------|---------|----------|
| FVG Entry Bifurcation | Passive vs momentum regime classification | P2 (Harrison's key request) |
| Same-bar fix | Pending FVG buffer for next-bar entry | P3 |
| V7aa Skip-Contra filter | 5M candle contradiction gate for QC model | P4 |
| Streak-based sizing | VA_fade wait-for-win, sweep pause-after-3L | P5 |
| Model correlation analysis | Measure trade overlap between Model 1 + Model 2 | P7 |
| Combined portfolio sim | Blend both models targeting 20R/day | P8 |
| OpenClaw tempo-trading skill | Telegram monitoring (batch status, alerts) | P10 |
| Kimi Claw deployment | Cloud-native brain for strategy/analysis | P11 |

### 11.3 The Real Architecture (Updated)

```
┌─────────────────────────────────────────────────────────────┐
│                    TEMPO BRAIN (Target State)                 │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │           EXISTING INFRASTRUCTURE                    │     │
│  │                                                      │     │
│  │  Context Engine V2 (68.6% session classification)    │     │
│  │  ├── Volume Profile (POC, VA, HVN/LVN)             │     │
│  │  ├── Footprint (delta, imbalance, absorption)       │     │
│  │  ├── SMT (NQ vs ES divergence)                      │     │
│  │  ├── Bayesian Classifier (5-class)                  │     │
│  │  ├── Markov Model (session transitions)             │     │
│  │  └── Strategy Bridge (session → config overrides)   │     │
│  │                                                      │     │
│  │  Model 1 — QC (Sweep+IFVG, market orders)          │     │
│  │  ├── main_v4.py (1,066 lines, cost model built)    │     │
│  │  ├── Batch runner (2-stage, 2000 variants)          │     │
│  │  └── VPS nightly cron (10 PM CT)                    │     │
│  │                                                      │     │
│  │  Model 2 — Backtester (BOS/sweep/VA/PGF, limits)   │     │
│  │  ├── micro_structure.py (pattern engine)            │     │
│  │  ├── intraday_scanner.py (13 signals, confluence)   │     │
│  │  └── V7y optimized: 7.25R/day, Calmar 137.8        │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                    NEW ADDITIONS                               │
│                           │                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │           BRAIN ORCHESTRATION LAYER                  │     │
│  │                                                      │     │
│  │  Ralph Loop (autonomous iteration engine)            │     │
│  │  ├── prd.json (9 prioritized stories)               │     │
│  │  ├── progress.txt (accumulated learnings)            │     │
│  │  └── Git persistence between iterations              │     │
│  │                                                      │     │
│  │  OpenClaw Gateway (multi-agent routing)              │     │
│  │  ├── STRATEGIST (Opus 4.6) → Context Engine V2      │     │
│  │  ├── EXECUTOR (GPT-5.3 Codex) → Batch Runner        │     │
│  │  ├── MONITOR (Opus) → Results Analysis               │     │
│  │  └── RISK (Opus) → Drawdown/Position Limits          │     │
│  │                                                      │     │
│  │  Kimi Claw (cloud brain, optional)                   │     │
│  │  ├── 2M token context for full-system reasoning      │     │
│  │  ├── Pro-Grade Search (Yahoo Finance, econ calendar)  │     │
│  │  └── Telegram bridge for Harrison notifications       │     │
│  └─────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 11.4 V7 Study Results Summary (Critical for Brain Training)

The V7r-V7aa studies represent the deepest edge analysis done on Model 2. The brain must know these findings:

| Study | Key Finding | Impact |
|-------|-------------|--------|
| V7r | Doji at FVG fill = best (40.8% WR, 1.483 avg R). Strong close = worst. | Entry timing |
| V7t | 5M_good_AND_no_contra = best practical gate (33% WR, 1.369 avg R) | Signal filtering |
| V7u | Scanner-filtered sweeps > raw sweeps. Market entry already well-timed. | Execution |
| V7v | VA_fade streak-dependent: after WIN = 60%+ WR. Aggressive sizing = 2.2x R. | Position sizing |
| V7w | Combined optimizations: MaxDD cut 62%, Calmar doubled to 123.99 | Portfolio |
| V7x | VA_fade DISABLE in ny_am/power_hour/silver_bullet. BOS_FVG GOLD everywhere. | Session allocation |
| V7y | Full optimized: 7.25R/day, MaxDD 16.4R, Calmar 137.8 | Production config |
| V7z | Wait-for-first-win on VA_fade: 50.9% WR, 3.4x better per trade | Regime detection |
| V7z-fix | FOLLOW large wick sweeps (never FADE). Dynamic stops MUST scale with wick. | Sweep mechanics |
| V7aa | SKIP 5M candle contradictions: +118% Calmar, +47% DD reduction | THE biggest edge |

---

## 12. Expected Outcomes

With this architecture fully deployed (building on what already exists):

- **Nightly autonomous optimization**: Ralph loop iterates on strategy improvements while you sleep
- **Morning briefings**: OpenClaw monitor sends daily summary to your phone
- **Real-time risk management**: Risk agent enforces limits 24/7
- **Continuous learning**: Every iteration makes the system smarter via progress.txt + AGENTS.md
- **Multi-model advantage**: Opus handles strategy thinking, Codex handles implementation
- **Fresh context every time**: No hallucination accumulation, no context drift, no stale assumptions
