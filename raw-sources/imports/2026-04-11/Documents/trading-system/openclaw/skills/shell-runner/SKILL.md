---
name: shell-runner
description: Run shell commands safely on Harrison's Mac or VPS. Includes safety checks, blocked commands, and confirmation for destructive operations.
emoji: 🐚
author: willis-holdings
version: 1.0.0
requires:
  bins:
    - bash
tags:
  - shell
  - terminal
  - system
---

# Shell Runner

Execute shell commands with safety guardrails.

## Safety Rules

### NEVER run these commands without Harrison's explicit approval:
- `rm -rf` on any system directory
- `kill -9` on unknown processes
- `git push --force` to main/master
- `git reset --hard`
- Any command that modifies system files
- Any command that exposes credentials

### Always confirm before:
- Deleting files (`rm`, `rmdir`)
- Killing processes (`kill`, `killall`)
- System changes (`shutdown`, `reboot`)
- Uninstalling packages (`brew uninstall`, `pip uninstall`)
- Force operations (`--force`, `--hard`)

## Common Operations

### Local Mac
```bash
# System info
uname -a && sw_vers
df -h /
top -l 1 | head -10

# Process management
ps aux | grep python
ps aux | grep claude
ps aux | grep nautilus

# Git operations
cd ~/trading_operator && git status
cd ~/trading_operator && git log --oneline -10

# Ollama
ollama list
ollama ps
curl -s http://localhost:11434/api/tags | python3 -m json.tool
```

### VPS (172.93.181.209)
```bash
# Quick health check
ssh -o ConnectTimeout=5 root@172.93.181.209 "uptime && df -h / && free -h"

# Check batch runner
ssh root@172.93.181.209 "tail -20 ~/batch3_costmodel.log"

# Check cron jobs
ssh root@172.93.181.209 "crontab -l"

# Check running processes
ssh root@172.93.181.209 "ps aux | grep python"
```

### Databento Data
```bash
# Count data files
ls ~/trading_operator/data2/GLBX-*/*.dbn.zst 2>/dev/null | wc -l

# Check data size
du -sh ~/trading_operator/data2/

# List contracts
ls ~/trading_operator/data2/GLBX-*/ | head -20
```
