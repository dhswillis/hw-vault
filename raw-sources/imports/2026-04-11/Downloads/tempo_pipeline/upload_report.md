# Twelve Labs Upload Script - Implementation Report

## Status
The upload script has been successfully created and configured, but cannot execute due to infrastructure constraints (external API access is blocked by the network proxy).

## What Was Prepared

### 1. Upload Script Created
**File:** `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_pipeline/upload_videos.py`

This script implements a complete Twelve Labs video upload pipeline with the following features:

#### Core Features:
- **Index Management**: Creates or retrieves an existing "tempo-trades" index
- **URL Processing**: Reads video URLs from a text file, filtering comments and empty lines
- **Video Upload**: Uploads videos from Discord CDN URLs to Twelve Labs via the Python SDK
- **Error Handling**: Gracefully handles expired URLs, rate limits, and API errors
- **Progress Tracking**: Saves upload progress to JSON, allowing resumption of interrupted uploads
- **Status Monitoring**: Polls task status and waits for videos to be indexed
- **Comprehensive Logging**: Detailed console output with progress tracking

#### API Configuration:
- Uses Twelve Labs Python SDK (`twelvelabs` package)
- Engine: Marengo 2.6 with visual, conversation, and text-in-video analysis
- Handles both successful uploads and graceful degradation for failed URLs
- Implements exponential retry logic (default 2 retries per video)

### 2. Video URL Data Analysis
- **Source**: `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/video_urls.txt`
- **Total Videos**: 324 videos
- **Format**: Discord CDN URLs with signed expiration parameters
- **Categories**:
  - None/👏🏽testimonials: 1 video
  - 🤑Tempo Trades/📝trade-recaps: 323 videos

### 3. API Key Configuration
- API Key: `tlk_3MNDZES1W42HJW2898BHS0WE7C04`
- Index Name: `tempo-trades`
- Status: Valid format (tlk_ prefix, 32 characters)

## How to Run (When External Access is Available)

### Command:
```bash
python /sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_pipeline/upload_videos.py \
  --api-key "tlk_3MNDZES1W42HJW2898BHS0WE7C04" \
  --urls "/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/video_urls.txt"
```

### Expected Behavior:

1. **Initialization Phase** (5-10 seconds)
   - Connect to Twelve Labs API
   - Check for existing "tempo-trades" index
   - Create index if it doesn't exist

2. **Upload Phase** (Hours, depending on number of successful videos)
   - Load all 324 video URLs from file
   - For each video:
     - Submit upload task to Twelve Labs
     - Poll status every 10 seconds
     - Wait for "ready" status (indicates video is indexed)
     - Handle errors gracefully:
       - Expired URLs: Logged and skipped
       - API errors: Retried up to 2 times before skipping
       - Network errors: Handled gracefully
   - 2-second delay between uploads (rate limiting)

3. **Progress Tracking**
   - Progress saved after each video to: `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/upload_progress.json`
   - Can resume from same point if interrupted

4. **Final Summary** (Printed to console)
   - Number of successfully uploaded videos
   - Number of failed uploads
   - Number of expired URLs
   - Final results saved to: `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/upload_results.json`

## Expected Results

### Likely Success Rate:
- **High Success Rate Expected**: Most Discord CDN links should work if downloaded recently
- **Some Expired URLs Expected**: Discord CDN URLs have 24-48 hour expiration windows
- **Successful Upload Range**: Likely 200-320 out of 324 videos (62-99%)

### Output Files Created:
1. **upload_progress.json** - Intermediate progress tracking
2. **upload_results.json** - Final detailed results with:
   - Completed uploads (video_id, task_id, URL, filename)
   - Failed uploads (error details)
   - Expired URLs (need re-download)

### Console Output Example:
```
============================================================
TWELVE LABS VIDEO UPLOAD PIPELINE
============================================================

  Connecting to Twelve Labs...
  Setting up index...
  Index 'tempo-trades' already exists (id=abc123...)
  Loading URLs from /sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/video_urls.txt...
  Total videos to upload: 324

  [1/324] video_1
    URL: https://cdn.discordapp.com/attachments/...
    Task created: id=task_12345
      Status: indexing_enqueued
      Status: indexing_in_progress
    ✅ Done! video_id=vid_abc123

  [2/324] video_2
    ...

============================================================
UPLOAD SUMMARY
============================================================
  ✅ Successfully uploaded: 287
  ❌ Failed: 15
  ⚠️  Expired URLs: 22

  🎉 287 videos indexed and searchable in Twelve Labs!
  You can now search across all videos using queries like:
    "entry criteria for ES long"
    "stop loss placement"
    "fair value gap fill"
    "session filter rules"
```

## Implementation Details

### Key Functions:

1. **`create_index(client, index_name)`**
   - Lists existing indexes
   - Returns ID of existing index or creates new one
   - Configures Marengo 2.6 engine with visual, conversation, and text_in_video analysis

2. **`upload_video_from_url(client, index_id, url, video_name, retry_count)`**
   - Creates upload task with `client.tasks.create()`
   - Polls status with `client.tasks.retrieve()`
   - Waits up to 1 hour per video for indexing to complete
   - Handles expired URLs gracefully
   - Implements retry logic for transient failures

3. **`load_urls_from_file(urls_file)`**
   - Reads text file line by line
   - Filters comments (lines starting with #)
   - Filters empty lines
   - Returns clean list of URLs

4. **`upload_all_videos(client, index_id, urls_file, progress_file)`**
   - Loads all URLs from file
   - Checks progress file for resume capability
   - Uploads each URL sequentially
   - Saves progress after each video
   - Applies rate limiting (2s between requests)

5. **`print_summary(results)`**
   - Displays statistics
   - Shows sample of failed/expired URLs
   - Explains what users can now query

## Error Handling

The script gracefully handles:
- **Expired URLs**: Detected by 403/Forbidden responses, logged separately
- **Network Errors**: Retried up to 2 times with 5-second backoff
- **API Rate Limits**: 2-second delay between uploads
- **Indexing Timeouts**: 1-hour timeout per video
- **Missing SDK**: Auto-installs `twelvelabs` package if not present

## Files Involved

### Input:
- `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/video_urls.txt` (324 URLs)

### Output:
- `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/upload_progress.json` (intermediate)
- `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_extracted/upload_results.json` (final)

### Script:
- `/sessions/awesome-brave-heisenberg/mnt/Downloads/tempo_pipeline/upload_videos.py` (executable)

## Infrastructure Limitations

The script cannot currently execute due to:
- Network proxy blocking access to `api.twelvelabs.io`
- System proxy configured with allowlist
- External API access not available in this environment

## Next Steps When External Access is Available

1. Ensure network connectivity to `api.twelvelabs.io`
2. Run the upload script with the command provided above
3. Monitor progress via JSON files
4. Re-download any expired Discord CDN URLs if needed
5. Query indexed videos using natural language

## Expected Search Capabilities After Upload

Once videos are indexed in Twelve Labs, you can search using queries like:
- "entry criteria for ES long"
- "stop loss placement"
- "fair value gap fill"
- "session filter rules"
- "risk management strategies"
- "market analysis techniques"
- And hundreds more trading-related queries
