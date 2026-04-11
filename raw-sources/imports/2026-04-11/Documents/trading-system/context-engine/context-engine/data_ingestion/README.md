# Data Ingestion Pipeline

Pulls trading knowledge from external sources (YouTube, Discord) and extracts structured rules, setups, and session insights for the context engine.

## YouTube Ingestion

```bash
# On VPS or local machine (needs internet access):
pip install youtube-transcript-api

# Single video
python youtube_ingest.py https://youtube.com/watch?v=VIDEO_ID

# Multiple videos from a file
python youtube_ingest.py --file video_urls.txt

# Process already-downloaded transcripts
python youtube_ingest.py --process-local transcripts/
```

### Suggested Free YouTube Content

Create a file `video_urls.txt` with URLs of trading videos to analyze:

```
# ICT / Smart Money Concepts
# (add video URLs here, one per line)

# Volume Profile / Order Flow
# (add video URLs here)

# NQ / Futures specific
# (add video URLs here)
```

### What Gets Extracted

- **Trading concepts**: FVGs, sweeps, order blocks, SMT, market structure
- **Session insights**: London, NY, Asia session-specific rules
- **Day type patterns**: Trend days, range days, news days
- **Volume/flow concepts**: POC, delta, absorption, imbalance
- **Risk rules**: Position sizing, stops, breakeven, targets
- **Actionable rules**: "Always wait for...", "Never trade during..."

### Output

- `transcripts/VIDEO_ID.json` — Raw transcript + extracted concepts per video
- `extracted_rules/knowledge_TIMESTAMP.json` — Aggregated rules across all videos

## Discord Ingestion (TODO)

Will pull from Discord channels using webhooks or bot API to extract:
- Trade calls and results
- Market commentary with timestamps
- Pattern recognition discussions
