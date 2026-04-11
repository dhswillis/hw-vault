# Databento Data Setup Guide for ES/NQ Futures Backtesting

## Overview

This guide walks you through setting up historical tick and bar data for E-mini S&P 500 (ES) and E-mini Nasdaq-100 (NQ) futures to use with the backtesting pipeline.

You have two main options:
1. **Databento** - Programmatic API access with $125 free credits
2. **FirstRateData** - One-time CSV download (simpler for beginners)

---

## Option 1: Databento (Recommended for Automation)

### Step 1: Create a Databento Account

1. Go to https://databento.com
2. Click "Sign up" or "Get started free"
3. Enter your email and create a password
4. Verify your email address
5. You'll receive **$125 in free API credits** automatically

### Step 2: Get Your API Key

1. Log in to your Databento account
2. Navigate to **Settings** → **API Keys** (or https://databento.com/portal/api-keys)
3. Click "Create API Key" or "Generate New Key"
4. Copy your API key and save it securely (you'll use this to download data)
5. Note: Keep this key private! Don't commit it to version control.

### Step 3: Install Required Packages

```bash
pip install databento pandas
```

### Step 4: Understanding Databento Pricing

**Free Tier:**
- New users get $125 in free credits
- Credits expire after 6 months
- Apply to all historical data requests

**Usage-Based Pricing:**
- Databento charges based on the amount of data downloaded
- Example: 6 trading days of ES tick data costs ~$2-3
- Typical pricing tiers:
  - Tick data: Higher cost per record
  - 1-minute bars: Lower cost than tick data
  - Daily/weekly: Minimal cost

**Cost Estimation:**
- **1 year of 1-minute ES/NQ data:** ~$20-40 total
- **1 year of tick data for 1 symbol:** ~$30-50
- **5 years of 1-minute data:** ~$100-200

You can estimate costs using the `--estimate` flag (see below).

### Step 5: Download Historical Data

#### Basic Usage (1-minute data for 2024)

```bash
python databento_downloader.py \
  --api-key YOUR_API_KEY \
  --symbols ES.FUT NQ.FUT \
  --start 2024-01-01 \
  --end 2024-12-31 \
  --resolution 1min
```

#### Download Multiple Resolutions

```bash
python databento_downloader.py \
  --api-key YOUR_API_KEY \
  --symbols ES.FUT NQ.FUT \
  --start 2020-01-01 \
  --end 2025-12-31 \
  --resolutions 1min 5min 1h 1d
```

#### Download Tick Data (High Resolution)

```bash
python databento_downloader.py \
  --api-key YOUR_API_KEY \
  --symbols ES.FUT \
  --start 2024-01-01 \
  --end 2024-01-31 \
  --resolution tick
```

#### Specify Custom Output Directory

```bash
python databento_downloader.py \
  --api-key YOUR_API_KEY \
  --symbols ES.FUT NQ.FUT \
  --start 2024-01-01 \
  --end 2024-12-31 \
  --output /path/to/data/directory
```

### Step 6: Monitor Your Credits

1. Go to https://databento.com/portal/usage
2. Check your remaining credits and historical usage
3. Download data before credits expire (6 months)

### Step 7: Verify Downloaded Data

After downloading, check the output directory for CSV files:

```bash
ls -lh ~/Downloads/tempo_data/
```

Expected files:
- `es_fut_1min.csv`
- `nq_fut_1min.csv`
- `es_fut_5min.csv`
- `es_fut_1h.csv`
- `es_fut_1d.csv`

Each CSV contains columns: `datetime`, `open`, `high`, `low`, `close`, `volume`

---

## Option 2: FirstRateData (Simpler Alternative)

### Step 1: Visit FirstRateData

1. Go to https://firstratedata.com
2. Browse to **ES (E-mini S&P 500)** or **NQ (E-mini Nasdaq-100)**

### Step 2: Select Data Package

FirstRateData offers historical data in multiple formats:
- **Tick Data:** Individual trades with exact time and price
- **Minute Bars:** 1-minute, 5-minute, 30-minute OHLCV bars
- **Hourly/Daily:** 1-hour and 1-day OHLCV bars

### Step 3: Pricing

Typical pricing (subject to change):
- **ES or NQ (individual symbol):** $30-50 for full history
- **ES + NQ bundle:** $50-80 for full history
- **Data includes:** 15-18 years of historical data

### Step 4: Download & Import

1. Purchase and download the CSV files
2. Place them in `~/Downloads/tempo_data/`
3. Rename to match expected format:
   - `es.csv` → `es_fut_1min.csv` (adjust resolution suffix as needed)
   - `nq.csv` → `nq_fut_1min.csv`

### Step 5: Verify Format

Ensure CSV files have columns:
- `datetime` (or `date`, `time`)
- `open`
- `high`
- `low`
- `close`
- `volume`

If column names differ, you may need to manually rename them or use a conversion script.

---

## Integration with Backtesting Pipeline

### Data Location

The backtesting engine expects data in:
```
~/Downloads/tempo_data/
```

CSV file naming convention:
- `{symbol}_{resolution}.csv`
- Examples: `es_fut_1min.csv`, `nq_fut_5min.csv`, `es_fut_tick.csv`

### Loading Data in Backtest

```python
import pandas as pd

# Load ES 1-minute data
es_data = pd.read_csv('~/Downloads/tempo_data/es_fut_1min.csv', parse_dates=['datetime'])

# Load NQ tick data
nq_data = pd.read_csv('~/Downloads/tempo_data/nq_fut_tick.csv', parse_dates=['datetime'])
```

### Data Requirements

The backtest engine expects:
1. **Datetime index:** Must be in chronological order
2. **OHLCV columns:** open, high, low, close, volume (required for bar data)
3. **No gaps:** Should have continuous data for your backtest period
4. **No NaN values:** All rows must have complete data

---

## Troubleshooting

### "API Key Invalid" Error

- Verify you copied the key correctly from Databento portal
- Check that spaces/newlines weren't accidentally included
- Generate a new key and try again

### "Insufficient Credits" Error

- Check your remaining balance at https://databento.com/portal/usage
- $125 in free credits may have expired
- Upgrade to a paid plan or wait for account credits

### "Symbol Not Found" Error

- Verify symbol format: Use `ES.FUT` and `NQ.FUT` (not `ES` or `ESM24`)
- Contact Databento support if symbol is unavailable

### Data Has Gaps or NaN Values

- This may indicate market hours or halts
- Verify dates fall within trading hours (Sun-Fri, 5pm-4pm ET)
- Check Databento documentation for holiday/halts

### "Network Connection Error"

- Verify internet connection is working
- Check that Databento API servers are online
- Try again in a few minutes

---

## Advanced: Continuous Contracts vs. Individual Contracts

### Continuous Contracts (Recommended)

Use `ES.FUT` and `NQ.FUT` for continuous front-month data:
- Automatically chains contracts to avoid gaps
- Adjusted for roll costs
- Better for long-term backtests

Example: Data from 2020-2025 is seamless, spanning multiple contract cycles

### Individual Contracts

For specific contract months, use format like:
- `ESM24` (March 2024 contract)
- `NQU24` (September 2024 contract)

Requires manual handling of rolls and adjustments.

---

## Reference: Data Resolutions Available

| Resolution | Use Case | Typical Cost |
|-----------|----------|--------------|
| **tick** | High-frequency analysis, micro-patterns | ~$30-50/year |
| **1min** | Intraday trading strategies | ~$5-10/year |
| **5min** | Swing trading, day trading | ~$2-5/year |
| **1h** | Medium-term strategies | <$1/year |
| **1d** | Long-term trend analysis | <$1/year |

---

## Next Steps

1. Choose Databento or FirstRateData
2. Set up your account and get your API key (Databento only)
3. Download 1-2 months of data to test
4. Verify the CSV files are in the correct format
5. Update your backtest configuration to point to the data directory
6. Run a test backtest with the downloaded data
7. If successful, download full historical data for your strategy

---

## Support & Resources

**Databento:**
- Docs: https://databento.com/docs
- Blog: https://databento.com/blog
- Support: support@databento.com

**FirstRateData:**
- Website: https://firstratedata.com
- Support: Available on website

---

## Security Note

Never commit your Databento API key to version control. Store it in environment variables:

```bash
export DATABENTO_API_KEY="your_api_key_here"
```

Then use in Python:

```python
import os
api_key = os.environ.get('DATABENTO_API_KEY')
```

Or use `.env` file (make sure to add `.env` to `.gitignore`):

```bash
# In .env file
DATABENTO_API_KEY=your_api_key_here
```

```python
# In Python
from dotenv import load_dotenv
import os
load_dotenv()
api_key = os.environ.get('DATABENTO_API_KEY')
```

---

Last Updated: 2026-02-09
