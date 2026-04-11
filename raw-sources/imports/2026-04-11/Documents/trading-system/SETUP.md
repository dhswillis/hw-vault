# MacBook Air Setup — Week 1

Run these in Terminal on your Mac, in order.

## 1. Install Homebrew
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
After it finishes, follow the instructions it prints to add brew to your PATH.

## 2. Install tools
```bash
brew install node python@3.12 git
```

## 3. Install Claude Code
```bash
npm install -g @anthropic-ai/claude-code
```

## 4. Clone the project
```bash
cd ~/Documents
git init trading-system
cp -r /path/to/these/files/* ~/Documents/trading-system/
cd ~/Documents/trading-system
git add .
git commit -m "Initial scaffold: API wrapper, context engine, brain framework"
```
(Or create a GitHub repo first and clone that instead.)

## 5. Set up Python environment
```bash
cd ~/Documents/trading-system
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## 6. Configure environment
```bash
cp .env.example .env
# Edit .env and add your Anthropic API key
```

## 7. Test the API Wrapper
```bash
source .venv/bin/activate
cd api-wrapper
python main.py
# Should start on http://localhost:8000
# Visit http://localhost:8000/docs for the API explorer
```

## 8. Test the Context Engine
```bash
source .venv/bin/activate
cd context-engine
python run.py --mode once --log-level DEBUG
# Should output context.json to ../signals/
```

## What's next after setup

1. **Sign up for QuantVPS Pro** at quantvps.com ($70/mo)
2. **Load Tempo strategy materials** — videos, Discord exports, PDFs, checklist
3. **Extract real rules** into the knowledge base and Tempo brain
4. **Backtest** once rules are codified
