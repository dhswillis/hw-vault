# Usage Examples

Complete examples for using the QuantConnect Batch Runner.

## Example 1: Basic Setup and Test

### Create credentials file
```bash
cat > /root/.env << 'EOF'
QC_USER_ID=123456
QC_API_TOKEN=your_actual_api_token_here
QC_PROJECT_ID=28083727
EOF

chmod 600 /root/.env
```

### Test API connectivity
```bash
cd /sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner
python3 scripts/test_single_backtest.py
```

**Expected output:**
```
================================================================================
QC API TEST - Single Backtest
================================================================================
Project ID: 28083727
User ID: 123456
[16:45:30] Compiling project 28083727...
[16:45:30] POST /api/v2/compile/create
[16:45:30] Payload: {...}
[16:45:40] Response: {"success": true, "compileId": "abc123def456"}
[16:45:40] Compilation successful: abc123def456
[16:45:42] Creating backtest: test_default_1707845140...
...
[16:46:05] Polling backtest (attempt 5, elapsed 25.2s)
[16:46:05] Progress: 100%
[16:46:05] Backtest completed!

================================================================================
BACKTEST RESULTS
================================================================================
Backtest ID: 12345
Name: test_default_1707845140
...
Result Data:
{
  "totalTrades": 115,
  "sharpeRatio": 1.45,
  "netProfit": 2886.50
}
...
```

If you get an error about credentials, check `/root/.env`.
If trade count is 0, your strategy might not be passing the required data.

## Example 2: Create Variants File

Create `variants.json`:

```json
[
  {
    "name": "baseline",
    "fast_ma": 20,
    "slow_ma": 50,
    "risk_per_trade": 0.02,
    "use_atr": true
  },
  {
    "name": "aggressive_short",
    "fast_ma": 10,
    "slow_ma": 30,
    "risk_per_trade": 0.03,
    "use_atr": true
  },
  {
    "name": "conservative_long",
    "fast_ma": 40,
    "slow_ma": 100,
    "risk_per_trade": 0.01,
    "use_atr": false
  },
  {
    "name": "medium_balanced",
    "fast_ma": 15,
    "slow_ma": 60,
    "risk_per_trade": 0.02,
    "use_atr": true
  },
  {
    "name": "high_frequency",
    "fast_ma": 5,
    "slow_ma": 20,
    "risk_per_trade": 0.025,
    "use_atr": true
  }
]
```

Each variant becomes a backtest with those parameters accessible to your QC strategy via the variant_json parameter.

## Example 3: Run Simple Batch

```bash
cd /sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner

# Create a simple 2-variant test
cat > test_variants.json << 'EOF'
[
  {
    "name": "test_1",
    "fast_ma": 20,
    "slow_ma": 50
  },
  {
    "name": "test_2",
    "fast_ma": 15,
    "slow_ma": 40
  }
]
EOF

# Run the batch
python3 scripts/run_batch.py test_variants.json test_results.jsonl
```

**Output:**
```
Loaded 2 variants from test_variants.json

Compiling project...
[16:50:15] Compiling project 28083727...
[16:50:15] Compilation successful: abc123

[1/2] Running variant: test_1
[16:50:20] Backtest created: 12345
[16:50:25] Progress: 35%
[16:50:30] Progress: 67%
[16:50:35] Progress: 100%

[2/2] Running variant: test_2
[16:50:40] Backtest created: 12346
...

================================================================================
SUMMARY
================================================================================
Variant                        Status     Approach   Total Trades
────────────────────────────────────────────────────────────────────────
test_1                         success    1          115
test_2                         success    1          125
────────────────────────────────────────────────────────────────────────
Completed: 2/2 successful
```

Results saved to `test_results.jsonl`.

## Example 4: Run with Verbose Logging

```bash
python3 scripts/run_batch.py variants.json results.jsonl --verbose > batch.log 2>&1
```

This captures all debug output for troubleshooting. Check `batch.log` to see:
- Exact JSON sent to each API endpoint
- Exact responses from API
- Timing of each operation
- Which approach was used for each variant

**Verbose output sample:**
```
[16:50:15] Compiling project 28083727...
[16:50:15] POST /api/v2/compile/create
[16:50:15] Payload: {
  "api_token": "your_token",
  "timestamp": 1707845415,
  "projectId": 28083727
}
[16:50:25] Response: {
  "success": true,
  "compileId": "abc123def456"
}
```

## Example 5: Parse Results with Python

```python
import json

# Read and analyze results
with open('results.jsonl', 'r') as f:
    results = [json.loads(line) for line in f]

# Find best result by Sharpe ratio
best = max([r for r in results if r.get('status') == 'success'],
           key=lambda x: x.get('statistics', {}).get('sharpe_ratio', -999))

print(f"Best variant: {best['variant_name']}")
print(f"  Sharpe Ratio: {best['statistics']['sharpe_ratio']}")
print(f"  Total Trades: {best['statistics']['total_trades']}")
print(f"  Return: {best['statistics']['return']}")

# Show all variants ranked by return
print("\nAll variants by return:")
for r in sorted([r for r in results if r.get('status') == 'success'],
                key=lambda x: x.get('statistics', {}).get('return', -999),
                reverse=True):
    return_pct = r['statistics'].get('return', 0) * 100
    print(f"  {r['variant_name']}: {return_pct:.2f}%")
```

## Example 6: Parse Results with Shell

```bash
# Show variant names and trade counts
cat results.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    r = json.loads(line)
    name = r.get('variant_name', 'unknown')
    trades = r.get('statistics', {}).get('total_trades', 'N/A')
    status = r.get('status', 'unknown')
    print(f'{name:30} {status:10} {trades}')
"

# Count successful runs
cat results.jsonl | grep -c '"status": "success"'

# Get total trades across all successful variants
cat results.jsonl | python3 -c "
import sys, json
total = sum(json.loads(line).get('statistics', {}).get('total_trades', 0)
            for line in sys.stdin if json.loads(line).get('status') == 'success')
print(f'Total trades: {total}')
"

# Find failed variants
cat results.jsonl | grep '"status": "failed"' | python3 -c "
import sys, json
for line in sys.stdin:
    r = json.loads(line)
    print(f\"{r['variant_name']}: {r.get('error', 'unknown error')}\")
"
```

## Example 7: Run Large Batch with Progress Tracking

```bash
# Create 50 variants
python3 << 'EOF'
import json
import itertools

fast_mas = [10, 15, 20, 25, 30]
slow_mas = [40, 50, 60, 70, 80, 100]
risks = [0.01, 0.02, 0.03]

variants = []
for fast, slow, risk in itertools.product(fast_mas, slow_mas, risks):
    if fast < slow:
        variants.append({
            "name": f"fm{fast}_sm{slow}_r{risk}",
            "fast_ma": fast,
            "slow_ma": slow,
            "risk_per_trade": risk
        })

with open('large_batch.json', 'w') as f:
    json.dump(variants, f, indent=2)

print(f"Created {len(variants)} variants")
EOF

# Run the batch (save output to file for later review)
python3 scripts/run_batch.py large_batch.json large_results.jsonl --verbose \
  | tee batch_run.log

# Monitor while it's running in another terminal
tail -f batch_run.log
```

## Example 8: Compare Two Runs

```bash
# Run with different strategy versions
python3 scripts/run_batch.py variants_v1.json results_v1.jsonl
python3 scripts/run_batch.py variants_v2.json results_v2.jsonl

# Compare results
python3 << 'EOF'
import json

with open('results_v1.jsonl', 'r') as f:
    v1 = {r['variant_name']: r for r in (json.loads(line) for line in f)}

with open('results_v2.jsonl', 'r') as f:
    v2 = {r['variant_name']: r for r in (json.loads(line) for line in f)}

print("Variant".ljust(30), "V1 Sharpe".ljust(15), "V2 Sharpe".ljust(15), "Change")
print("-" * 75)

for name in sorted(set(v1.keys()) & set(v2.keys())):
    v1_sharpe = v1[name].get('statistics', {}).get('sharpe_ratio', 0)
    v2_sharpe = v2[name].get('statistics', {}).get('sharpe_ratio', 0)
    change = ((v2_sharpe - v1_sharpe) / v1_sharpe * 100) if v1_sharpe else 0
    print(name.ljust(30), f"{v1_sharpe:.2f}".ljust(15),
          f"{v2_sharpe:.2f}".ljust(15), f"{change:+.1f}%")
EOF
```

## Example 9: Retry Failed Variants

```bash
# Extract only failed variants from a previous run
python3 << 'EOF'
import json

with open('results.jsonl', 'r') as f:
    results = [json.loads(line) for line in f]

failed_variants = [r for r in results if r.get('status') == 'failed']

if failed_variants:
    # Regenerate the original variants file with just the failed ones
    # (This is pseudocode - depends on your variant format)
    print(f"Found {len(failed_variants)} failed variants")
    print("Failed variants:")
    for r in failed_variants:
        print(f"  - {r['variant_name']}: {r.get('error')}")
else:
    print("No failed variants!")
EOF

# Re-run just the failed variants with verbose output
python3 scripts/run_batch.py failed_variants.json retry_results.jsonl --verbose
```

## Example 10: Generate Report

```bash
python3 << 'EOF'
import json
from datetime import datetime

with open('results.jsonl', 'r') as f:
    results = [json.loads(line) for line in f]

successful = [r for r in results if r.get('status') == 'success']

print("=" * 80)
print("BATCH EXECUTION REPORT")
print("=" * 80)
print(f"Timestamp: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print(f"Total Variants: {len(results)}")
print(f"Successful: {len(successful)}")
print(f"Failed: {len(results) - len(successful)}")
print()

if successful:
    stats_list = [r['statistics'] for r in successful]

    print("STATISTICS SUMMARY")
    print("-" * 80)
    print(f"Average Sharpe Ratio: {sum(s.get('sharpe_ratio', 0) for s in stats_list) / len(stats_list):.2f}")
    print(f"Average Return: {sum(s.get('return', 0) for s in stats_list) / len(stats_list) * 100:.2f}%")
    print(f"Average Trades: {sum(s.get('total_trades', 0) for s in stats_list) / len(stats_list):.0f}")
    print(f"Average Win Rate: {sum(s.get('win_rate', 0) for s in stats_list) / len(stats_list) * 100:.2f}%")
    print()

    print("TOP 5 BY SHARPE RATIO")
    print("-" * 80)
    for i, r in enumerate(sorted(successful,
                                  key=lambda x: x['statistics'].get('sharpe_ratio', 0),
                                  reverse=True)[:5], 1):
        s = r['statistics']
        print(f"{i}. {r['variant_name']}: {s.get('sharpe_ratio', 0):.2f}")
        print(f"   Return: {s.get('return', 0)*100:.2f}% | Trades: {s.get('total_trades', 0)} | Win Rate: {s.get('win_rate', 0)*100:.1f}%")

print()
print("=" * 80)
EOF
```

## Troubleshooting Examples

### Example: 0 Trades Issue

```bash
# Run test with verbose to see what's happening
python3 scripts/test_single_backtest.py > test_output.txt 2>&1

# Check if parameters are in the request
grep -A 5 "Payload:" test_output.txt

# Check if trades are in the response
grep -i "total.*trade\|trade.*count" test_output.txt

# Check full response structure
tail -100 test_output.txt | grep -A 50 "BACKTEST RESULTS"
```

### Example: API Authentication Issues

```bash
# Verify credentials file exists and has correct format
cat /root/.env

# Check credential values
grep "QC_" /root/.env

# Run test with verbose to see auth attempts
python3 scripts/test_single_backtest.py --verbose 2>&1 | grep -i "auth\|token\|api"
```

### Example: Timeout Issues

Edit the scripts to increase timeout if backtests take > 1 hour:

```python
# In run_batch.py or test_single_backtest.py, change:
MAX_POLL_TIME = 3600  # 1 hour

# To:
MAX_POLL_TIME = 7200  # 2 hours
```

Then re-run the batch.

---

For more information, see README.md, QUICKSTART.md, or IMPLEMENTATION_NOTES.md.
