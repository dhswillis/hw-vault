# Implementation Notes

## Architecture Overview

The batch runner is implemented as a single Python class `QCBatchRunner` with the following key responsibilities:

1. **Authentication**: SHA256 hash-based authentication with timestamps
2. **Project Management**: Compile project once, reuse compilation
3. **Backtest Execution**: Create backtests with variant parameters
4. **Result Collection**: Poll for completion and extract statistics
5. **Dual-Approach Robustness**: Handles both direct and project-level parameter passing

## Core Components

### Authentication (`_hash_auth`, `_make_request`)

All API requests use QuantConnect's hash-based authentication:

```python
auth_string = f"{api_token}:{timestamp}"
auth_hash = hashlib.sha256(auth_string.encode()).hexdigest()
```

Each request includes:
- `api_token`: Raw API token
- `timestamp`: Unix timestamp (seconds)
- `auth_hash`: Not explicitly sent; authentication is implicit in the request

Request format:
```json
{
  "api_token": "your_token",
  "timestamp": 1707842400,
  "projectId": 12345,
  "compileId": "abc123",
  ...
}
```

### Compilation Phase

```python
def compile_project(self) -> str:
    response = self._make_request("/api/v2/compile/create", {"projectId": pid})
    return response["compileId"]
```

This happens once per batch run. All variants use the same compilation to save time.

### Parameter Passing: Two Approaches

#### Approach 1: Direct Parameters in Create Request

```python
request_data = {
    "projectId": pid,
    "compileId": cid,
    "backtestName": name,
    "parameters": {"variant_json": json.dumps(variant)}
}
response = self._make_request("/api/v2/backtests/create", request_data)
```

**Pros**: Direct, no extra API calls
**Cons**: May not work if API doesn't accept parameters in create request

#### Approach 2: Project-Level Parameters

```python
# First, set parameters at project level
self._make_request("/api/v2/projects/update", {
    "projectId": pid,
    "parameters": {"variant_json": json.dumps(variant)}
})

# Then create backtest (will inherit project parameters)
self._make_request("/api/v2/backtests/create", {
    "projectId": pid,
    "compileId": cid,
    "backtestName": name
})
```

**Pros**: Works with APIs that only accept project-level parameters
**Cons**: Requires extra API call per variant, slower

### Dual-Approach Strategy

The `run_variant()` method implements a fallback mechanism:

```python
try:
    # Try approach 1
    parameters = {"variant_json": json.dumps(variant)}
    backtest_id = self.create_backtest(cid, name, parameters=parameters)
    backtest = self.poll_backtest(backtest_id)
    stats = self.extract_statistics(backtest)

    # Check if we got results
    if stats.get("total_trades") == 0:
        raise Exception("Zero trades with approach 1")

    return {"status": "success", "approach": 1, "statistics": stats}

except Exception as e:
    # Fall back to approach 2
    self.set_project_parameters({"variant_json": json.dumps(variant)})
    backtest_id = self.create_backtest(cid, f"{name}_retry")
    backtest = self.poll_backtest(backtest_id)
    stats = self.extract_statistics(backtest)

    return {"status": "success", "approach": 2, "statistics": stats}
```

This ensures robustness against different API configurations.

### Polling Implementation

```python
def poll_backtest(self, backtest_id: str) -> dict:
    start_time = time.time()

    while True:
        elapsed = time.time() - start_time

        if elapsed > MAX_POLL_TIME:
            raise Exception("Timeout")

        response = self._make_request("/api/v2/backtests/read", {...})
        backtest = response.get("backtest", {})
        progress = backtest.get("progress", 0)

        if progress >= 100:
            return backtest

        time.sleep(POLL_INTERVAL)
```

**Parameters**:
- `POLL_INTERVAL = 5` seconds between checks
- `MAX_POLL_TIME = 3600` seconds (1 hour) maximum

This is reasonable for QC backtests which typically take 30-120 seconds.

### Statistics Extraction

The `extract_statistics()` method handles variable field naming:

```python
field_mappings = {
    "total_trades": ["totalTrades", "total_trades", "TradesCount"],
    "sharpe_ratio": ["sharpeRatio", "sharpe_ratio", "Sharpe"],
    "net_profit": ["netProfit", "net_profit", "NetProfit"],
    # ... etc
}

for stat_name, possible_fields in field_mappings.items():
    for field in possible_fields:
        if field in result:
            stats[stat_name] = result[field]
            break
```

This tries multiple possible field names for each metric, checking both the `result` and `statistics` sections of the backtest response.

**Why this is needed**: Different API versions or response structures may use different field names. This makes the runner more resilient.

## Data Flow

```
Load Variants JSON
    ↓
Load Credentials from ~/.env
    ↓
Compile Project (once)
    ↓
For Each Variant:
    ├─ Attempt Approach 1: Direct parameters
    │  ├─ Create backtest with parameters
    │  ├─ Poll for completion
    │  ├─ Extract statistics
    │  └─ If 0 trades, fall through to Approach 2
    │
    └─ Attempt Approach 2: Project parameters
       ├─ Set project-level parameters
       ├─ Create backtest
       ├─ Poll for completion
       └─ Extract statistics
    ↓
Save Results to JSONL (one per line)
    ↓
Print Summary Table
```

## Output Formats

### JSONL Result Format

Each line contains a complete result object:

```json
{
  "variant_name": "baseline_default",
  "backtest_id": "12345",
  "status": "success",
  "approach": 1,
  "statistics": {
    "backtestId": "12345",
    "backtestName": "baseline_default_1707842400",
    "total_trades": 115,
    "sharpe_ratio": 1.45,
    "net_profit": 2886.50,
    "return": 0.2886,
    "win_rate": 0.65,
    "drawdown": -0.15,
    "probability_cagr": 0.35
  }
}
```

**Advantages of JSONL**:
- One result per line (easy to stream/parse)
- Each result is valid JSON (easy to parse individually)
- Append-only (results saved immediately, not buffered)
- Works well with Unix tools (grep, sed, etc.)

### Summary Table Format

Printed to console after completion:

```
============================================================================
SUMMARY
============================================================================
Variant                        Status     Approach   Total Trades
────────────────────────────────────────────────────────────────────────
baseline_default               success    1          115
aggressive_short_ma            success    1          145
────────────────────────────────────────────────────────────────────────
Completed: 2/4 successful
```

## Error Handling Strategy

### API Request Errors

```python
for attempt in range(max_retries):
    try:
        # Make request
    except urllib.error.HTTPError as e:
        if attempt < max_retries - 1:
            wait_time = 2 ** attempt  # Exponential backoff
            time.sleep(wait_time)
        else:
            raise
    except urllib.error.URLError as e:
        # Same retry logic
```

**Exponential Backoff**: 1s, 2s, 4s for 3 attempts

### Variant Errors

```python
try:
    result = self.run_variant(variant, compile_id)
except Exception as e:
    results.append({
        "variant_name": variant.get("name", "unknown"),
        "status": "failed",
        "error": str(e)
    })
    continue  # Process next variant
```

Failures in one variant don't stop the batch. All variants are attempted.

### Polling Timeout

```python
if elapsed > MAX_POLL_TIME:
    raise Exception(f"Backtest polling timeout after {MAX_POLL_TIME}s")
```

Prevents infinite loops if a backtest gets stuck.

## Credential Handling

Credentials are loaded from `/root/.env`:

```env
QC_USER_ID=123456
QC_API_TOKEN=abc123def456
QC_PROJECT_ID=28083727
```

Format: `KEY=VALUE` pairs, one per line, `#` for comments.

**Security considerations**:
- File is read-only (mode 600 recommended)
- Token is loaded into memory only
- Token is not logged (except in verbose mode, which logs requests)
- Token is not saved anywhere except in-memory for current session

## Logging and Verbosity

### Verbose Mode Output

When `--verbose` flag is used:

```
[HH:MM:SS] Compilation successful: abc123def456
[HH:MM:SS] POST /api/v2/backtests/create
[HH:MM:SS] Payload: {"projectId": 123, ...}
[HH:MM:SS] Response: {"success": true, ...}
[HH:MM:SS] Polling backtest (attempt 1, elapsed 5.2s)
[HH:MM:SS] Progress: 45%
```

Useful for debugging:
- See exact JSON being sent/received
- Track timing of each operation
- Identify which API call is causing issues

### Progress Output

Non-verbose mode still shows:
- Compilation success
- Backtest creation for each variant
- Polling progress updates
- Final summary table

## Configuration Tuning

### For Large Batches (50+ variants)

Increase `MAX_POLL_TIME` to accommodate more variations:

```python
MAX_POLL_TIME = 7200  # 2 hours
```

### For Fast Polling

Decrease `POLL_INTERVAL` to check more frequently:

```python
POLL_INTERVAL = 2  # Check every 2 seconds
```

Risk: Higher API usage, may hit rate limits.

### For Unreliable Networks

Increase `max_retries` in `_make_request()`:

```python
response = self._make_request(..., max_retries=5)
```

This increases total retry time with exponential backoff.

## Known Limitations

1. **Rate Limiting**: QC API may rate-limit high-frequency polling. Increase `POLL_INTERVAL` if you hit limits.

2. **Project Compilation**: All variants use the same compilation. If you need to change strategy code between variants, they must be in separate projects or you must manually recompile.

3. **Parameter Encoding**: Parameters are passed as JSON strings. Complex nested structures should be JSON-serializable.

4. **Statistics Field Names**: If QC API changes field names in responses, update `field_mappings` in `extract_statistics()`.

5. **Authentication**: Hash is computed client-side but not explicitly sent. This matches QC's expected behavior.

## Testing Strategy

Use `test_single_backtest.py` to:

1. Verify credentials work
2. Test baseline strategy (no parameters)
3. Confirm trade generation is working
4. Inspect full API response structure
5. Identify actual field names in results

Before running batch, ensure:
- Single test completes successfully
- Trade counts are > 0
- All expected statistics are in response
- No errors in compilation or backtest creation

## Future Enhancements

Potential improvements (not implemented):

1. **Parallel Execution**: Create multiple backtests simultaneously instead of sequentially
2. **Result Filtering**: Skip variants below certain thresholds
3. **Parameter Optimization**: Use results to suggest better parameters
4. **Caching**: Skip re-running identical variants
5. **Multi-Project Support**: Run variants across multiple projects
6. **Database Output**: Save results to database instead of JSONL
7. **Webhooks**: Notify external service when batch completes

## Debugging Tips

1. **Enable verbose mode**: Always use `--verbose` when troubleshooting
2. **Check credentials**: Verify `/root/.env` has correct values
3. **Test API independently**: Use `test_single_backtest.py` first
4. **Inspect raw responses**: Look at JSON in verbose output
5. **Check QC web IDE**: Confirm strategy works there before running batch
6. **Monitor API usage**: Check QC account for rate limiting issues
7. **Save logs**: Redirect output to file: `run_batch.py ... --verbose > batch.log 2>&1`

## Performance Characteristics

### Typical Execution Times

- Project Compilation: 10-30 seconds
- Single Backtest: 30-120 seconds (depends on data, strategy complexity)
- Batch of 10 variants: 5-20 minutes (sequential)

### Optimizations Applied

1. Compile once, use for all variants
2. Exponential backoff on failures (avoids hammering API)
3. Configurable polling interval (balance latency vs. API usage)
4. Line-by-line result writing (stream processing, less memory)

### Bottleneck Analysis

- **I/O**: Network requests dominate execution time
- **CPU**: Negligible (JSON parsing, hash computation)
- **Memory**: Minimal (one result at a time)
- **API Rate Limits**: Main constraint for large batches

## Code Quality

### Design Principles

1. **Single Responsibility**: Each method does one thing
2. **Error Handling**: Try-except blocks for all API calls
3. **Logging**: Informative log messages at each step
4. **Type Hints**: Python 3.6+ type annotations
5. **Documentation**: Docstrings for all public methods
6. **Configuration**: Parameterized constants for tuning

### Testing Coverage

Not a fully tested codebase, but key paths:

- Successful variant execution
- API error handling and retries
- Parameter passing (both approaches)
- Result extraction
- File I/O and formatting

### Dependencies

**Zero external dependencies!**

Uses only Python standard library:
- `urllib.request`, `urllib.error` - HTTP requests
- `hashlib` - SHA256 hashing
- `json` - JSON parsing
- `time`, `os`, `sys`, `pathlib` - System utilities

This makes the script:
- Easy to deploy (no pip install)
- Secure (no third-party vulnerabilities)
- Lightweight
- Compatible with any Python 3.6+

---

For questions or issues, refer to README.md or enable `--verbose` mode for detailed debugging output.
