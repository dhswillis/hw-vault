# QuantConnect Batch Runner

A Python-based batch runner for QuantConnect backtests that replaces the previous bash implementation. This runner handles multiple strategy variants, manages API authentication, and collects comprehensive results.

## Features

- Hash-based SHA256 authentication with QuantConnect REST API
- Compiles projects once, then runs multiple backtest variants
- Dual-approach parameter passing for robustness:
  - Approach 1: Direct parameter passing in backtest creation request
  - Approach 2: Fallback to project-level parameters via `/api/v2/projects/update`
- Automatic polling with configurable timeouts
- Full result extraction and statistics collection
- JSONL output format (one result per line) for easy parsing
- Summary table display upon completion
- Comprehensive error handling and retry logic
- Verbose logging mode for debugging

## Setup

### 1. Create Credentials File

Create `/root/.env` with your QuantConnect credentials:

```env
QC_USER_ID=your_user_id
QC_API_TOKEN=your_api_token
QC_PROJECT_ID=your_project_id
```

Replace:
- `your_user_id`: Your numeric QuantConnect user ID
- `your_api_token`: Your QuantConnect API token
- `your_project_id`: The numeric ID of your QuantConnect project

### 2. Install Python (if needed)

The scripts require Python 3.6+ with only standard library dependencies:
- `urllib` for HTTP requests
- `hashlib` for SHA256 authentication
- `json` for data parsing
- `time`, `os`, `sys` for system operations

No external packages needed!

### 3. Prepare Variants File

Create a JSON file with your strategy variants. Each variant is a dictionary with parameters:

```json
[
  {
    "name": "baseline_default",
    "fast_ma": 20,
    "slow_ma": 50,
    "risk_per_trade": 0.02
  },
  {
    "name": "aggressive",
    "fast_ma": 10,
    "slow_ma": 30,
    "risk_per_trade": 0.03
  }
]
```

See `variants.json.example` for a complete example.

## Usage

### Test Single Backtest

First, verify API connectivity with a single backtest (no parameters):

```bash
python3 /sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner/scripts/test_single_backtest.py
```

This will:
1. Compile your project
2. Create a single backtest with default parameters
3. Poll until completion
4. Display full results including statistics

Use this to verify:
- Credentials are correct
- API connectivity works
- Your strategy produces expected trade counts
- Result data structure matches expectations

### Run Batch

Run multiple variants:

```bash
python3 /sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner/scripts/run_batch.py \
  variants.json \
  results.jsonl \
  --verbose
```

Arguments:
- `variants.json`: Input file with variant definitions (required)
- `results.jsonl`: Output file for results (optional, defaults to `results.jsonl`)
- `--verbose`: Enable debug logging (optional)

Output:
- Creates JSONL file with one result per line
- Prints summary table to console
- Each result contains:
  - Variant name
  - Backtest ID
  - Status (success/failed)
  - Approach used (1 or 2)
  - Full statistics object

### Example Run

```bash
cd /sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner

# Copy example variants file
cp variants.json.example variants.json

# Test single backtest first
python3 scripts/test_single_backtest.py

# If test succeeds, run batch
python3 scripts/run_batch.py variants.json results.jsonl --verbose
```

## Results Format

### JSONL Output

Each line is a complete JSON object:

```json
{"variant_name": "baseline_default", "backtest_id": "12345", "status": "success", "approach": 1, "statistics": {"backtestId": "12345", "backtestName": "baseline_default_1707842400", "total_trades": 115, "sharpe_ratio": 1.45, "net_profit": 2886.50, "return": 0.2886, "win_rate": 0.65, "max_drawdown": -0.15, "cagr": 0.35}}
```

### Summary Table

```
============================================================================
SUMMARY
============================================================================
Variant                        Status     Approach   Total Trades
────────────────────────────────────────────────────────────────────────
baseline_default               success    1          115
aggressive_short_ma            success    1          145
conservative_long_ma           success    2          87
medium_balanced                failed     -          -
────────────────────────────────────────────────────────────────────────
Completed: 3/4 successful
```

## Troubleshooting

### "0 trades" Issue

If backtests consistently return 0 trades:

1. **Verify web IDE works**: Confirm your strategy produces trades when run from the QC web IDE
2. **Check parameters**: The runner uses two approaches:
   - Approach 1: Passes `variant_json` in the backtest create request
   - Approach 2: Sets project parameters via `/api/v2/projects/update` first
3. **Enable verbose logging**: Add `--verbose` flag to see exactly what's being sent
4. **Check result fields**: Look at full backtest response to identify actual field names for statistics
5. **Verify strategy code**: Ensure your QuantConnect project has working strategy code

### API Errors

- **401 Unauthorized**: Check credentials in `/root/.env`
- **404 Not Found**: Verify project ID is correct
- **Timeout**: Check internet connection, QC API status
- **Connection errors**: Retry with exponential backoff (handled automatically, max 3 attempts)

### Parameter Not Applied

If you suspect parameters aren't being passed correctly:

1. Run with `--verbose` to see exact JSON sent to API
2. Check if approach 2 (project-level parameters) works
3. Modify strategy to log received parameters to QuantConnect output
4. Review QC API documentation for parameter format expectations

## API Endpoints Used

The runner uses these QuantConnect REST API endpoints:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v2/compile/create` | POST | Compile project code |
| `/api/v2/backtests/create` | POST | Create a backtest run |
| `/api/v2/backtests/read` | POST | Poll backtest status and get results |
| `/api/v2/projects/update` | POST | Update project-level parameters (fallback) |

All requests include authentication via:
- `api_token`: Your API token
- `timestamp`: Current Unix timestamp
- `auth_hash`: SHA256("{api_token}:{timestamp}")

## Performance

Typical execution times:
- Project compilation: 10-30 seconds
- Single backtest: 30-120 seconds (depends on data and complexity)
- Batch of 10 variants: 5-20 minutes total

Polling:
- Interval: 5 seconds between checks
- Timeout: 1 hour maximum per backtest

## Files

```
/sessions/magical-eager-mayer/mnt/Documents/trading-system/batch-runner/
├── scripts/
│   ├── run_batch.py              # Main batch runner
│   └── test_single_backtest.py   # API test utility
├── variants.json.example          # Example variants file
├── results.jsonl                  # Generated results (after running)
└── README.md                       # This file
```

## Advanced Usage

### Extracting Results with Python

```python
import json

with open('results.jsonl', 'r') as f:
    for line in f:
        result = json.loads(line)
        print(f"{result['variant_name']}: {result['statistics']['total_trades']} trades")
```

### Extracting with Shell

```bash
# Show all variant names and trade counts
cat results.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    r = json.loads(line)
    print(f\"{r['variant_name']}: {r['statistics'].get('total_trades', 'N/A')}\")
"
```

### Filtering Successful Results

```bash
# Only successful results
cat results.jsonl | grep '"status": "success"'
```

## Development

To modify the scripts:

1. The main logic is in `QCBatchRunner` class in `run_batch.py`
2. Key methods:
   - `_make_request()`: Handles all API communication
   - `create_backtest()`: Creates a backtest
   - `poll_backtest()`: Polls for completion
   - `extract_statistics()`: Parses results
   - `run_variant()`: Orchestrates a single variant run
   - `run_batch()`: Main batch execution

3. Customize field extraction in `extract_statistics()` to match your API response structure

## Notes

- Each variant gets a unique backtest name with timestamp: `{variant_name}_{unix_timestamp}`
- Project is compiled once at the start; variants reuse that compilation
- Requests include automatic retry logic (exponential backoff, max 3 attempts)
- Verbose logging shows all JSON sent to/received from API
- Results are saved line-by-line to JSONL for durability

## License

This script is provided as-is for use with QuantConnect. Refer to QuantConnect's API documentation and terms of service.
