---
name: claude-code-manager
description: Spawn, manage, and monitor Claude Code sessions. Use for ANY task that needs real computer control — installing software, running backtests, writing code, managing files. Claude Code is your hands.
emoji: 💻
author: willis-holdings
version: 2.0.0
requires:
  bins:
    - claude
    - python3
    - bash
tags:
  - coding
  - development
  - claude-code
  - automation
---

# Claude Code Manager

You are the orchestrator. Claude Code is your executor. When a task requires computer control, coding, file manipulation, software installation, or anything beyond text chat — delegate to Claude Code.

## DECISION: When to Use Claude Code

USE Claude Code for:
- Installing software (brew, npm, pip, .msi, .dmg)
- Running backtests or data analysis
- Writing/editing code files
- System administration (processes, services, cron)
- Git operations
- File management across directories
- Anything requiring multi-step computer interaction

DON'T use Claude Code for:
- Simple questions Harrison can answer
- Checking if a process is running (use bash directly)
- Reading a single file (use bash directly)

## HOW TO RUN

### Quick task (< 5 min, get output back)
```bash
cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "YOUR TASK HERE"
```

### Long-running task (> 5 min, run in background)
```bash
nohup bash -c 'cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "YOUR TASK HERE" > /tmp/claude_task_output.txt 2>&1' &
echo "Task started. PID: $!"
```

Then check progress:
```bash
tail -50 /tmp/claude_task_output.txt
```

### Very long task (hours — mining, installs)
```bash
TASK_ID=$(date +%s)
nohup bash -c 'cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "$(cat MINE_V2_PROMPT.md)" > /tmp/claude_task_${TASK_ID}.txt 2>&1' &
echo "Mining task $TASK_ID started. PID: $!"
echo "$!" > /tmp/claude_task_${TASK_ID}.pid
```

### Check if Claude Code is running
```bash
ps aux | grep -E "claude.*--print" | grep -v grep
```

### Kill a stuck Claude Code session
```bash
pkill -f "claude.*--print"
```

### Get last result
```bash
cat /tmp/claude_task_output.txt | tail -100
```

## CONTEXT INJECTION

ALWAYS inject context into Claude Code prompts. It starts blind every time.

Template:
```
You are working in ~/Documents/trading-system. Read CLAUDE.md first for ground truth.

TASK: [what to do]

KEY CONTEXT:
- Data: ./databento/ (312 .dbn.zst NQ tick files, Feb 2025-Feb 2026)
- Backtester: clean_backtest.py (ONLY use this, do NOT create new frameworks)
- Current best: 4429 trades, 50.7% WR, +0.90 avgR, 1.5pt min stops, trail 0.5/0.1
- Results go in: ./results/
- NEVER ask questions, NEVER stop. If something fails, log it and continue.
```

## COMMON TASKS

### Run parameter sweep
```bash
cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "$(cat MINE_V2_PROMPT.md)"
```

### Install NinjaTrader (via VMware)
```bash
cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "
Read CLAUDE.md first. Then:
1. Check if VMware Fusion is installed: ls /Applications/VMware*
2. If not, download and install VMware Fusion (free for personal use)
3. Download Windows 11 ARM ISO from Microsoft
4. Create a VM with 8GB RAM, 60GB disk
5. Install Windows in the VM
6. Copy ~/Downloads/NinjaTrader.Install.msi into the VM
7. Install NinjaTrader 8 in the VM
8. Report back what happened at each step.
Do NOT ask questions. If a step fails, log the error and try the next step.
"
```

### Port strategy to NinjaTrader
```bash
cd ~/Documents/trading-system && claude --dangerously-skip-permissions --print --output-format text -p "
Read CLAUDE.md and clean_backtest.py. Port the BOS_FVG strategy to a NinjaTrader 8 NinjaScript C# strategy.
The strategy must:
- Use 1M bars as primary series
- Detect swings with 3-bar lookback (confirmed)
- Detect BOS (break of confirmed swing)
- Detect FVGs (3-bar gap pattern)
- Match BOS->FVG->fill for entry signals
- Use pure trailing exit (trigger=0.5R, buffer=0.1R)
- Session: 8AM-3PM ET, no entries after 2:45PM ET
- Min risk: 1.5pt, max risk: 50pt
- Entry slippage: 0.25pt, exit slippage: 0.25pt
Save the .cs file to ~/Documents/trading-system/ninjatrader/
"
```

## MONITORING LONG TASKS

After launching a background Claude Code task, check on it periodically:

```bash
# Is it still running?
ps aux | grep -E "claude.*--print" | grep -v grep | wc -l

# How big is the output?
wc -l /tmp/claude_task_output.txt 2>/dev/null

# Last 20 lines
tail -20 /tmp/claude_task_output.txt 2>/dev/null

# Look for errors
grep -i "error\|fail\|crash" /tmp/claude_task_output.txt 2>/dev/null | tail -10

# Look for phase completions
grep -i "phase\|complete\|saved\|results" /tmp/claude_task_output.txt 2>/dev/null | tail -10
```

## WATCHDOG MODE

If Harrison says "keep it running" or "don't let it stop", use this loop:

```bash
while true; do
    # Check if Claude Code is running
    if ! pgrep -f "claude.*--print" > /dev/null 2>&1; then
        echo "[$(date)] Claude Code stopped. Restarting..."
        cd ~/Documents/trading-system && nohup claude --dangerously-skip-permissions --print --output-format text -p "$(cat MINE_V2_PROMPT.md)" >> /tmp/claude_task_output.txt 2>&1 &
        echo "[$(date)] Restarted with PID $!"
    else
        echo "[$(date)] Claude Code still running."
    fi
    sleep 120  # Check every 2 minutes
done
```

## COST NOTE
Claude Code uses paid API (~$3-10 per session). Only use for tasks that genuinely need computer control. For simple text questions, answer directly with Ollama.
