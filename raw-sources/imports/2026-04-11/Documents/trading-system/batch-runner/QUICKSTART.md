# Quick Start Guide

## 1 Minute Setup

### Step 1: Create credentials file
```bash
cat > /root/.env << 'EOF'
QC_USER_ID=your_numeric_id
QC_API_TOKEN=your_api_token
QC_PROJECT_ID=your_project_id
EOF
```

### Step 2: Test the API
```bash
python3 scripts/test_single_backtest.py
```

If this completes successfully with > 0 trades, your setup is correct.

### Step 3: Create variants file
```bash
cp variants.json.example variants.json
# Edit variants.json with your strategy parameters
```

### Step 4: Run batch
```bash
python3 scripts/run_batch.py variants.json results.jsonl --verbose
```

Watch the output for progress. Results are saved to `results.jsonl`.

## Common Commands

```bash
# Show all results
cat results.jsonl | python3 -m json.tool | head -50

# Count successful runs
grep '"status": "success"' results.jsonl | wc -l

# Extract trade counts
cat results.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    r = json.loads(line)
    trades = r.get('statistics', {}).get('total_trades', 'N/A')
    print(f\"{r['variant_name']}: {trades} trades\")
"

# Find best Sharpe ratio
cat results.jsonl | python3 -c "
import sys, json
results = [json.loads(line) for line in sys.stdin]
best = max([r for r in results if r.get('status')=='success'],
           key=lambda x: x.get('statistics', {}).get('sharpe_ratio', -999))
print(f\"Best: {best['variant_name']} - Sharpe: {best['statistics']['sharpe_ratio']}\")
"
```

## Troubleshooting

**"0 trades" error**: Run `--verbose` to see what's being sent to the API. Check if parameters are being passed correctly.

**"Connection failed"**: Verify internet connection and QC API status at https://www.quantconnect.com

**"Unauthorized"**: Check credentials in `/root/.env` match your QuantConnect account.

See full README.md for detailed documentation.
