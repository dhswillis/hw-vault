# Genius Brain - Full Context Claude API Caller

## Description
Calls Claude API with the entire trading knowledge base as system prompt.
Use this whenever you need deep strategy analysis, trade evaluation,
or any decision that requires full context of all strategies.

## When to Use
- Evaluating whether a setup matches strategy rules
- Analyzing market context against documented filters
- Making decisions that require cross-strategy knowledge
- Debugging backtest results or identifying why trades aren't triggering
- Any question where partial context could lead to errors

## How It Works
1. Reads ~/trading-system/KNOWLEDGE_BASE.md
2. Reads all files in ~/trading-system/docs/brains/
3. Optionally reads ~/trading-system/context/daily.json for real-time context
4. Concatenates everything into a system prompt
5. Sends user question + system prompt to Claude API
6. Returns the response

## Implementation
Run: `node ~/trading-system/genius-brain.js "<user question>"`

## Options
- `--model opus` : Use Claude Opus for complex analysis (default: Sonnet)
- `--context` : Include daily context file in system prompt
- `--verbose` : Print full system prompt size and token estimate

## Examples
```
node ~/trading-system/genius-brain.js "Is this a valid Tempo setup: NQ swept London low, IFVG formed on 1m at 21380, quality 3, SMT confirmed"
node ~/trading-system/genius-brain.js --model opus "Why did V3.4 only produce 37 trades in 12 months?"
node ~/trading-system/genius-brain.js --context "Given today's context, what are the most likely setups?"
```
