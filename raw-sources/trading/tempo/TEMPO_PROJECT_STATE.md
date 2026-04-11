# TEMPO TRADING SYSTEM — MASTER PROJECT STATE

**Last Updated:** February 9, 2026
**Owner:** Harrison Willis (dhswillis@gmail.com)
**Purpose:** Read this file FIRST in any new Claude session to avoid going in circles.

---

## READ THIS FIRST — CRITICAL CONTEXT

This project builds an **automated NQ/ES futures trading system** based on the Tempo Trading methodology. The CORRECT architecture is described in `~/Downloads/Option4_Hybrid_Handoff_Document.docx` — a four-layer system using Claude Code, a Custom API Wrapper, OpenClaw, and Quantower. DO NOT build a standalone QuantConnect algo. QC is for backtesting only.

Harrison has corrected Claude multiple times:
1. **Do NOT invent rules.** Only use rules explicitly stated by Tempo.
2. **Follow the Option 4 architecture.** Not a monolithic QC algo.
3. **The real rules live in 324 trade recap videos.** The 23 text-extracted rules are a skeleton only.
4. **SMT means Smart Money Technique** (comparing NQ vs ES for divergence), NOT "Supply Manipulation Test."

---

## 1. THE CORRECT ARCHITECTURE (Option 4: Hybrid Full Stack)

### Four Layers

| Layer | Role | Technology | Where It Runs |
|-------|------|------------|---------------|
| 1. Claude Code | The Builder — writes all code | CLI tool on MacBook | Harrison's Mac |
| 2. Custom API Wrapper | The Genius Brain — 100k+ token knowledge base | FastAPI + Claude API | VPS |
| 3. OpenClaw | The Operator — autonomous 24/5 execution | Agent framework + vector DB | VPS |
| 4. Claude Business Suite | Harrison's daily tools | Claude.ai, Chrome, Cowork | Mac/browser |

### Signal Flow (Production)

```
Brain detects setup
  → Calls API Wrapper for validation (full context)
  → Writes signal.json to shared folder
  → Quantower C# FileWatcher reads signal
  → Quantower executes order
  → Replikanto copies to prop firm accounts
  → Quantower writes fill.json
  → OpenClaw reads fill, logs trade, updates P&L
```

### Where QuantConnect Fits

QC is used ONLY in the development cycle for backtesting. It is NOT the production execution layer. The flow is: Claude Code writes brain → Claude Code writes QC backtest → Review results → Iterate → Package as OpenClaw Skill.

---

## 2. TEMPO STRATEGY — WHAT WE ACTUALLY KNOW

### Core Sequence (From Handoff Doc)

1. FVG exists on the entry timeframe (dynamic: 5s to 5m depending on vol/tick regime)
2. A candle closes through the FVG → makes it "valid"
3. That close simultaneously sweeps a higher timeframe level:
   - 15m fractal (swing H/L)
   - Major session H/L (London, Asia, NY)
   - 5m/15m FVG fill
4. Checklist passes: no news, not extreme momentum against
5. Entry: Inverse (fade the sweep)

### Key Concepts

- **Draw on Liquidity (DOL):** Where price is heading. Targets = session H/L, 15m fractals, unmitigated FVGs on 5m/15m.
- **Reaction Points:** Levels where price reverses after sweeping DOL. Entry zones.
- **Dynamic Entry Timeframe:** NOT fixed. Adapts to volume/tick regime. High volume = bigger levels only. Low volume = 1m levels become DOL.
- **15m Candle Liquidity:** High-energy 15m candles create sweepable liquidity above/below.
- **SMT = Smart Money Technique:** Compare NQ vs ES. If NQ makes new high but ES doesn't → bearish divergence. If NQ makes new low but ES doesn't → bullish divergence.

### What We Still Don't Know (Needs Video Extraction)

- FVG minimum size, body vs wick, "clean" criteria
- Validation: does "close through" mean any part or fully through?
- Sweep targets: 15m fractal definition (how many bars), which sessions count
- Exact entry placement (FVG retest, immediate, limit or market)
- Stop: below sweep, below FVG, fixed ticks, or ATR-based?
- Target: fixed R, opposing FVG, opposing liquidity?
- The Tempo Indicator: what platform, is source code visible?

### 23 Text-Extracted Rules (From Discord Messages)

Located at: `~/Downloads/tempo_extracted/tempo_rules.json`
These are high-level and vague. The video recaps contain the actual methodology details.

---

## 3. WHAT HAS BEEN BUILT

### QuantConnect Backtest (Project ID: 28083410)

**Account:** Daniel Willis, Researcher tier, User ID 113205
**Files:**
- QC cloud: `main.py` in project 28083410
- Local V3 algo: `~/Downloads/tempo_pipeline/qc_tempo_brain.py` (V3 with 281-transcription rules)
- Launch script: `~/Downloads/tempo_pipeline/qc_launch_backtest.py` (fixed: compile endpoint corrected)

#### V1 Results (Backtest: "Focused Brown Viper")
- Period: 2020-2025, $50k starting capital
- 2020-2021 bull market: $50k → ~$200k (incredible gains)
- 2022 bear market: $200k → $16k (complete wipeout, -92% from peak)
- Root causes: No drawdown protection, slow daily bias detection, bull-biased entries, no volatility scaling
- Key lesson: The algorithm traded well with the trend but had ZERO ability to detect regime change

#### V2 Changes Applied
Six improvements added to address V1 collapse:
1. **20% max drawdown circuit breaker** — halts trading when equity drops 20% from peak
2. **2% daily loss limit** — stops trading for the day after 2% loss
3. **Faster daily bias** — 3-bar short-term (weighted 2x) + 5-bar medium-term + EMA confirmation
4. **Volatility scaling** — 50% position size when ATR > 1.5x median
5. **Strict bias filtering** — sweep longs ONLY when bullish, sweep shorts ONLY when bearish
6. **ATR history tracking** — rolling 20-value ATR for median calculation

#### V2 Results (Backtest: "Energetic Fluorescent Pink Fox")
- $50k → $68,229 (36.46% return) then FLAT for 5 years
- Circuit breaker fired permanently after initial drawdown from ~$75k peak
- **Design flaw:** `circuit_breaker_active = True` NEVER resets. Once tripped, trading stops forever.
- PSR decayed from 81% → 9% as time passed with zero trades
- The circuit breaker needs a recovery mechanism (e.g., reset after equity recovers to 90% of peak, or weekly reset)

### Known Code Issues Fixed
1. `.TotalSeconds` → `.total_seconds()` (Python timedelta, not C# property)
2. `self.Debug()` calls caused OOM on second-resolution data (15 calls removed)
3. RollingWindow sizes reduced (200→30, 50→20) to save memory
4. Empty Python blocks after Debug removal needed `pass` statements
5. Multiple IndentationError fixes from empty except/if blocks

---

## 4. FILE INVENTORY

### Architecture & Strategy Documents
- `~/Downloads/Option4_Hybrid_Handoff_Document.docx` — **THE master architecture document.** Read this for the correct system design.
- `~/Downloads/Video_Ingestion_Pipeline_Guide.docx` — How to extract strategies from educator videos.
- `~/Downloads/tempo_extracted/tempo_rules.json` — 23 text-extracted rules (high-level, incomplete).
- `~/Downloads/tempo_extracted/tempo_rules_summary.md` — Summary of extracted rules.

### Code Files
- `~/Downloads/qc_tempo_brain_v2.py` — Latest QC algorithm (V2 with circuit breaker, saved Feb 9 2026).
- `~/Downloads/tempo_pipeline/qc_tempo_brain.py` — Older local copy (31,417 bytes, out of date).
- `~/Downloads/tempo_pipeline/run_twelve_labs_upload.py` — Script to upload 324 videos to Twelve Labs.
- `~/Downloads/tempo_pipeline/transcribe_videos.py` — Script to download audio + transcribe with Whisper.
- `~/Downloads/tempo_pipeline/upload_videos.py` — Original upload script (older version).

### Video Data
- `~/Downloads/tempo_extracted/video_urls.txt` — 334 lines, 324 video URLs from Discord.
- `~/Downloads/tempo_extracted/video_manifest.json` — Metadata for all 324 videos.
  - 312 trade recaps from tony31. (Tempo himself)
  - 11 announcements from tony31.
  - 1 testimonial
  - Total size: 81.4 GB
  - **LINKS EXPIRE: Feb 10, 2026 ~3:51 AM ET** — must upload/download before then.

### Price Data
- `~/Downloads/tempo_extracted/price_data/` — ES and NQ futures data (1d and 1h).

### Discord Full Export (ALL Channels)
- `~/Downloads/tempo_full_export/` — Complete Discord server export (JSON per channel).
- Key education channels NOT yet fully mined (see Section 5 below).

### Existing Transcriptions
- `~/Downloads/tempo_extracted/transcriptions/` — 29 Whisper transcriptions already completed.
- `~/Downloads/tempo_extracted/transcribe_progress.json` — Progress tracker for Whisper pipeline.
- Total words transcribed so far: ~29,636 across 29 videos (Oct 2 – Nov 18, 2024).

---

## 5. VIDEO & CONTENT EXTRACTION PIPELINE (Twelve Labs + Whisper + Discord)

This is the most critical section. The algorithm cannot improve without extracting the real rules from Tempo's content. There are THREE sources of Tempo's methodology, and all three must be mined.

### 5.1 Source A: Discord Trade Recap Videos (324 videos, 81.4 GB)

These are daily recordings where Tempo walks through his trades, explains his reasoning, shows his charts, and discusses what worked and what didn't. This is where the execution-level detail lives.

**Status:**
- 324 video URLs scraped from Discord CDN (file: `~/Downloads/tempo_extracted/video_urls.txt`)
- 29 already transcribed via Whisper (Oct 2 – Nov 18 2024, stored in `~/Downloads/tempo_extracted/transcriptions/`)
- 295 remaining to transcribe
- Discord CDN links have expiry timestamps — check before attempting download
- Manifest with metadata: `~/Downloads/tempo_extracted/video_manifest.json`

**Twelve Labs Upload Pipeline:**
- API Key: `tlk_3MNDZES1W42HJW2898BHS0WE7C04`
- Index Name: `tempo-trades`
- Upload script: `~/Downloads/tempo_pipeline/run_twelve_labs_upload.py`
- Run from Mac terminal (VM proxy blocks API): `python3 ~/Downloads/tempo_pipeline/run_twelve_labs_upload.py`
- Supports resume — progress saved to `~/Downloads/tempo_extracted/upload_progress.json`
- Once indexed, query at https://playground.twelvelabs.io or via API

**Whisper Transcription Pipeline (Backup/Free):**
- Script: `~/Downloads/tempo_pipeline/transcribe_videos.py`
- Requires: ffmpeg + openai-whisper (`brew install ffmpeg && pip3 install openai-whisper`)
- Downloads audio only (not full video), transcribes locally
- Supports resume — progress saved to `~/Downloads/tempo_extracted/transcribe_progress.json`
- Output: individual .txt files per video + combined ALL_TRANSCRIPTS.txt

**What the 29 Existing Transcriptions Already Reveal (examples from Oct 2 recap):**
- Uses ES liquidity sweep to take NQ trade (SMT in practice)
- Checks Asia H/L, London H/L at session open — if all swept, "no overnight areas to use"
- Waits for "significant area of liquidity to sweep" — patience, not forcing trades
- After sweep: looks for "bearish reaction candles right away" as confirmation
- Trims at 20pts and 40pts on NQ — two-trim exit strategy
- Targets "retracement to the daily range"
- Avoids lunchtime window / outside NY Killzone
- Uses premium/discount framework — "still in premiums, shorting into discount"
- 4H FVG as resistance/target level
- Data wick candles from news events (NFP at 8:15) as potential entry triggers

### 5.2 Source B: Discord Education Channels (Text + Embedded Videos)

The Discord server has dedicated education channels with structured content. These have NOT been fully extracted into rules yet.

**Channel: 📜Education - ⚡terminology (Tempo's ICT Glossary)**
Confirmed definitions:
- SMT = Smart Money Technique (NOT "Supply Manipulation Test")
- DOL = Draw of Liquidity
- IFVG = Inverted Fair Value Gap
- FVG = Fair Value Gap
- BPR = Balanced Price Range
- PO3 = Power of 3 (Accumulation, Manipulation, Distribution)
- MMXM = Market Maker Sell/Buy Model
- KZ = Killzone
- NWOG = New Week Opening Gap
- PD Array = OBs, FVGs, IFVGs grouped
- EQ = Equilibrium (50% fib from H to L)
- PDEQ/PWEQ/PMEQ = Previous Day/Week/Month Equilibrium
- ITH/ITL = Intermediate Term High/Low
- RTO = Return to Origin
- OVN = Overnight

**Channel: 📜Education - ✍premium-education (42 video lessons — GOLD MINE)**
These are the structured teaching videos. Topics include:
1. My A+ IFVG Model Checklist
2. When To Go Breakeven During A Setup
3. How To Find Good Stop Losses When Trading IFVGs
4. How To Find The Best Possible FVG To Inverse
5. How I Find My Daily Bias
6. How To Trade Orderblocks With IFVGs and Find Good Re-entries
7. How To Trade Data Highs and Lows
8. How long after the liquidity sweep is a play still valid?
9. How To Trade IFVGs During A Trend Day
10. The Main Reasons Why IFVGs Fail
11. How to trade the 50% level with IFVGs
12. How To Decide When To "Chase" An IFVG Entry
13. How To Trade The New York PM Session Using IFVGs
14. How to trade London Session With IFVGs
15. How to identify chop and bad price conditions
16. Liquidity Tier List Ranking video
17. How to quickly pass evals with CPI and NFP
18. How I Manage Trades Using Quant Tower
19. What is a Tempo Sweep?
20. Negative Confluences to look for when trading
21. Reviewing 5 A+ IFVG Setups
22. How to trade New Week Opening Gap With IFVGs
23. The difference between A and B setups with My Model
24. When its appropriate to de-risk on a setup
25. How to manage sizing on different prop accounts (25k, 50k, and 150k)
26. Best settings and tutorial for Tempo Trades Indicator
27. UPDATED 2026 BEST IFVG TRIMMING STRATEGY!
28. All the indicators I use

**Channel: 📜Education - 🎓premium-classes (18 YouTube class URLs — PERMANENT, no expiry)**
These are full-length classes posted as YouTube links. They do NOT expire like Discord CDN:
- https://www.youtube.com/watch?v=Djph6Pd9oAc (8/11/2024 Sunday Class)
- https://www.youtube.com/watch?v=sUvizgvWdLg (10/06/2024 Premium Sunday Class)
- https://www.youtube.com/watch?v=rjF3yiCQ-Sc (2020 Election Week Backtest)
- https://www.youtube.com/watch?v=hyA0uTrr_MQ (Failed IFVG Setups)
- https://www.youtube.com/watch?v=9KWRxPYgJos (1/09/2025 Premium Class)
- https://www.youtube.com/watch?v=egpMndjbQkw (01/26/2025 SMT Class)
- https://www.youtube.com/watch?v=43jVoSQFDg8 (02/05/2025 Backtesting IFVGs)
- https://www.youtube.com/watch?v=3KtsUg8DkaA (02/20/2025 IFVG Chasing + Sizing)
- https://www.youtube.com/watch?v=-CX04Tc1L-E (03/05/2025 Member Setups Review)
- https://www.youtube.com/watch?v=Y9MCA0cgN9k (03/19/2025 Psychology + IFVG)
- https://www.youtube.com/watch?v=BOhvhgEuLog (04/16/2025 Prop Firm + IFVG Help)
- https://www.youtube.com/watch?v=sg_7kytBCKM (05/14/2025 IFVG Setups)
- https://www.youtube.com/watch?v=PsLN1DbEP7M (06/12/2025 IFVG Backtesting)
- https://www.youtube.com/watch?v=FUYBctoCd78 (07/23/2025 Backtesting Session)
- https://www.youtube.com/watch?v=o8SX-sJKxJw (08/26/2025 Order Blocks + Setups)
- https://www.youtube.com/watch?v=eAul_KT2eBg (10/10/2025 Prop Firm Road Map)
- https://www.youtube.com/watch?v=QI5Kl6jJozg (11/28/2025 High Volatility Markets)
- https://youtu.be/D7TLDMEncr8 (01/21/2025 Live Journal Review)

To transcribe these YouTube videos (no expiry concerns):
```bash
pip3 install yt-dlp openai-whisper
yt-dlp -x --audio-format wav "URL" -o "%(title)s.%(ext)s"
# Then run whisper on the .wav file
```

**Channel: 🤑Tempo Trades - 🖥️my-indicator**
Tempo's custom indicator features:
- Automatically maps 50% of the daily range level
- Marks out all equal highs and equal lows (liquidity targets)
- On-screen built-in checklist with all of Tempo's favorite confluences
- Marks out all session killzones
- Plots all FVGs and IFVGs with a sensitivity setting
- Available at: https://whop.com/tempotrades/tempo-trades-indicator/

**Channel: 🤑Tempo Trades - ❓faq (CRITICAL — corrects our code)**
Key facts from Tempo himself:
- **Timeframe:** "Mainly the 1 minute. But if the 1m is sloppy I will go to 30s, 2m, 3m, and even 5m sometimes"
- **Risk per trade:** "$200-300 per trade on a 50k apex account" = **0.4-0.6% risk, NOT 1%**
- **Markets:** ES/NQ futures
- **Broker:** Quantower (matches Option 4 architecture)
- **Charting:** TradingView
- **Model:** "ICT concept called IFVGs (inverted fair value gaps)"
- **News calendar:** ForexFactory.com
- **Prop firms:** Apex (primary), Topstep, MFFU, Bulenox
- **Trading schedule:** Live trades every morning at 9:30 AM ET with premium group

**Channel: 📜Education - 💥free-course**
Free course at https://tempoeducation.com/strategy covering:
- "The Exact IFVG Strategy That Got Me A 42-Day Win Streak" (90% win rate claimed)
- "Master Psychology & Emotions From a 7-Figure Trader"
- "The Perfect Course For Anyone Trying To Master ICT"

### 5.3 Systematic Query Framework for Rule Extraction

Once videos are indexed in Twelve Labs (or transcribed via Whisper), run these queries systematically to extract every aspect of Tempo's methodology:

**Entry Rules:**
- "What makes an IFVG valid for entry?"
- "How does Tempo determine which FVG to inverse?"
- "What is an A+ setup vs a B setup?"
- "When does Tempo chase an IFVG entry?"
- "How does the liquidity sweep confirm the entry?"
- "How long after a sweep is a play still valid?"
- "What is a Tempo Sweep?"

**Daily Bias:**
- "How does Tempo find his daily bias?"
- "What determines if the day is bullish or bearish?"
- "How does Tempo handle trend days?"
- "How does Tempo identify chop and bad conditions?"

**Stop Loss & Risk:**
- "Where does Tempo place stop losses on IFVG trades?"
- "How does Tempo manage risk on different account sizes?"
- "When does Tempo go breakeven?"
- "When does Tempo de-risk?"

**Targets & Exits:**
- "What is Tempo's trimming strategy?"
- "How does Tempo decide take profit levels?"
- "How does Tempo trail stops?"

**Session & Timing:**
- "How does Tempo trade London session?"
- "How does Tempo trade NY PM session?"
- "What are the killzone times?"
- "How does Tempo trade around CPI and NFP?"
- "How does Tempo use data wicks?"

**SMT & Multi-Market:**
- "How does Tempo use ES to confirm NQ trades?"
- "What is Smart Money Technique in Tempo's model?"

**Advanced:**
- "What negative confluences make Tempo avoid a trade?"
- "What is the 50% daily range level and how is it used?"
- "How does Tempo trade New Week Opening Gap?"
- "How does Tempo use orderblocks with IFVGs?"

### 5.4 Extraction Priority Order

1. **YouTube premium classes** (18 URLs, permanent, no expiry) — transcribe with yt-dlp + Whisper
2. **Discord premium-education videos** (42 videos) — find URLs in the JSON, upload to Twelve Labs or transcribe
3. **Remaining Discord trade recaps** (295 of 324) — continue Whisper transcription
4. **Free course at tempoeducation.com** — may have structured written content
5. **Discord text channels** (premium-chat, trading-chat, premium-alerts) — mine for rule clarifications

---

## 6. CONTEXT ENGINE (From Architecture Doc)

The production system needs a Context Engine that runs before market open and continuously during the session. It outputs `context.json` that every brain reads.

### Hard Filters (Binary: Trade or Don't)
- High-impact news within 15 min → no trades
- VIX > 35 (crisis) → no trades
- Before 8:30 AM or after 3:00 PM ET → no new positions
- GEX extreme bearish (DIX < 0.35) → no longs
- Price beyond SpotGamma 1-Day Expected Range → mean reversion only

### Soft Filters (Position Size Multipliers)
- DIX bullish (>0.47) + long setup → 1.25x
- Price near SpotGamma Call/Put Wall → 1.15x
- Dark pool cluster aligns with S/D zone → 1.2x
- CVD divergence at zone → 1.15x
- GEX positive + inside day → 1.1x

### Day Fingerprinting
Each trading day gets a statistical fingerprint: day of week, FOMC proximity, CPI/NFP proximity, OPEX proximity, VIX regime, GEX regime, gap size, IB range vs ATR. Historical outcomes for that fingerprint adjust brain parameters.

---

## 7. WHAT TO DO NEXT (Priority Order)

### URGENT: Video Ingestion (Links Expire Feb 10)
Run from Mac terminal:
```bash
# Option A: Twelve Labs (recommended — AI-powered video search)
pip3 install twelvelabs
python3 ~/Downloads/tempo_pipeline/run_twelve_labs_upload.py

# Option B: Whisper transcription (free, local)
brew install ffmpeg
pip3 install openai-whisper
python3 ~/Downloads/tempo_pipeline/transcribe_videos.py
```

### After Videos Are Processed
1. Extract detailed Tempo rules from transcriptions/video search
2. Update `tempo_rules.json` with video-extracted rules
3. Identify the missing pieces: FVG sizing, entry placement, stop rules, target rules

### V3 Algorithm Improvements
1. Fix circuit breaker: add recovery mechanism (reset after equity recovers to X% of peak, or time-based reset)
2. Implement DOL (Draw on Liquidity) framework from Tempo's actual method
3. Add dynamic entry timeframe based on volume regime (not fixed 30s/1m/2m/3m)
4. Add session-specific behavior (London open, NY open, power hour)
5. Add 15m fractal sweep detection as a higher-timeframe confirmation
6. Implement the correct SMT: NQ vs ES divergence at key levels, not generic correlation

### Production Architecture Build (From Option 4 Doc)
1. **Week 1:** MacBook setup (Homebrew, Node, Claude Code, Python 3.12, Git)
2. **Week 2:** Build Custom API Wrapper (FastAPI + Claude API with 100k+ token knowledge base)
3. **Week 2:** Build Context Engine v1 (VIX, economic calendar, day type, session ranges)
4. **Week 3+:** Build Tempo Brain as OpenClaw Skill
5. **Week 4+:** Quantower C# FileWatcher bridge
6. **Week 5+:** VPS deployment + Replikanto for prop firm account copying

---

## 8. LESSONS LEARNED (Don't Repeat These Mistakes)

1. **Don't invent rules.** Harrison has explicitly said this. Only codify what Tempo actually teaches. If unsure, leave it out or ask.
2. **Don't build a monolithic QC algo.** QC is for backtesting only. The production system uses the four-layer Option 4 architecture.
3. **Don't use `self.Debug()` on second-resolution data.** It causes OOM via BaseResultsHandler log accumulation. Use `self.Log()` sparingly (entry/exit/EOD only).
4. **RollingWindow sizes matter.** Don't use 200 when 20-30 suffices. Each one consumes memory proportional to size × data frequency.
5. **Python timedelta uses `.total_seconds()` not `.TotalSeconds`.** The latter is C# syntax.
6. **Circuit breakers need recovery.** A permanent halt is worse than no circuit breaker — it means one bad week ends your entire backtest/live run forever.
7. **Empty Python blocks need `pass`.** When removing code from if/except/try blocks via regex, always scan for empty blocks afterward.
8. **The 23 text rules are a skeleton.** The real methodology is in the 324 trade recap videos. Don't over-optimize based on incomplete rules.
9. **SMT = Smart Money Technique** (NQ vs ES divergence), not "Supply Manipulation Test."
10. **Daily bias detection is critical.** V1 failed because it couldn't detect the 2022 bear market fast enough. Bias needs to be responsive AND accurate.
11. **Risk per trade is 0.4-0.6%, NOT 1%.** Tempo risks $200-300 on a $50k account. Our code uses 1% which is too aggressive.
12. **The 42 premium education video titles are a roadmap.** Each one answers a specific question we need (stop losses, daily bias, breakeven, A vs B setups, trimming strategy). Prioritize transcribing these.

---

## 9. QUANTCONNECT TECHNICAL REFERENCE

- **Project ID:** 28083410 (V3, created Feb 9 2026; old V2 was 28078389)
- **Compile endpoint:** `/compile/create` (NOT `/compiler/create` — the extra "r" was causing HTML responses)
- **Browser API:** `fetch('/api/v2/files/read', {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({projectId: 28083410, fileName: 'main.py'})})`
- **Update file:** `fetch('/api/v2/files/update', {method:'POST', ...})`
- **Backtest button:** Position ~(778, 57), tooltip "Backtest Project (⌘F5)"
- **Futures:** `Futures.Indices.NASDAQ100EMini` (NQ), `Futures.Indices.SP500EMini` (ES)
- **Data resolution:** `Resolution.Second` for entry, `TradeBarConsolidator` for 30s/1m/2m/3m/5m bars
- **Active contracts:** Track via `FuturesChain` in `OnData`, not `self.Securities`
- **RollingWindow[T]:** `.Count` (not `len()`), `.Add()`, `.IsReady`, `[0]` = most recent
- **Brokerage:** Must set `self.SetBrokerageModel(BrokerageName.QuantConnectBrokerage)` for futures

---

## 10. EDUCATOR PIPELINE (Future Brains)

The system is designed to support multiple educator strategies:
1. **Tempo** (Brain #1) — NQ/ES futures, IFVG + liquidity sweeps. IN PROGRESS.
2. **Flipping Markets** — Has a PDF strategy doc. RECOMMENDED as second brain (easiest extraction).
3. **Lanto** — Discord-based educator.
4. **Carmine** — YouTube-based educator.
5. **Omega** — Discord/YouTube educator.

Each brain follows the same lifecycle: Extract rules → Codify as Python → QC backtest → Iterate → Deploy as OpenClaw Skill.

---

## 11. VIDEO-EXTRACTED RULES

### V1 Extraction (29 transcriptions, 29,636 words — Feb 9, 2026)
- **`~/Downloads/tempo_extracted/tempo_rules_from_videos.md`** — Initial extraction
- **`~/Downloads/tempo_extracted/tempo_rules_from_videos.json`** — Initial JSON version

### V2 Extraction (90 transcriptions, 93,278 words — Feb 9, 2026)
- **`~/Downloads/tempo_extracted/tempo_rules_v2_complete.md`** — Comprehensive human-readable rules (18 sections, 500+ lines)
- **`~/Downloads/tempo_extracted/tempo_rules_v2_complete.json`** — Complete machine-readable version for algorithm consumption
- Covers Oct 2, 2024 through Mar 7, 2025 (5 months of daily recaps)
- 3x more data than V1; includes methodology evolution tracking

### V3 Extraction (281 transcriptions, 257,335 words — Feb 9, 2026) ← USE THIS
- **`~/Downloads/tempo_extracted/tempo_rules_v3_complete.md`** — Comprehensive V3 rules (20 sections)
- **`~/Downloads/tempo_extracted/tempo_rules_v3_complete.json`** — Machine-readable V3 rules with algorithm parameter recommendations
- Covers Oct 2, 2024 through Jan 24, 2026 (16 months of daily recaps across 5 methodology phases)
- 3x more data than V2; tracks full methodology evolution from foundation → mastery → refinement
- Mined in 3 parallel batches: Oct-Dec 2024, Jan-Mar 2025, Apr 2025-Jan 2026

### V3 Key Changes vs V2
- **SMT is now PRIMARY confluence** — required on funded accounts (V3 emphasis from Nov 2025+)
- **Conservative trimming: 55% at TP1** (was 40% in V2) — Tempo shifted to base hit mentality
- **Soft stops default (90%)** — hard stops only for specific scenarios
- **BPR (Balance Price Range)** = "FVG on steroids" — recognized as premium entry in Nov 2025+
- **88% win rate reported across 400+ trades** — base hit approach works
- **Data Candle Setup** — 8:30 AM news candles as structured entry
- **Trend Day Protocol** — specific rules for 300+ pt days
- **Copy Trading guidance** — Replikanto settings and account management

### Key Numbers Extracted (USE THESE, not guesses)
- IFVG optimal gap: 8–15 pts NQ (5–6 risky but valid, 20–30 strong, skip >30 pts)
- TP1: 15–20 pts (mechanical, non-negotiable), TP2: 30–40 pts, TP3: 50–70 pts, Runner: 70–260+ pts
- V3 Trimming: 55% at TP1, 30% at TP2, 15% runner (was 40/35/25 in V2)
- Stop loss: 20–35 pts (FVG), 10–25 pts (swing), 8–12 pts (small gap), 35–45 pts (extended high-conviction)
- Break-even: 15–20 pts profit (non-negotiable), do NOT BE below 10 pts (too tight)
- Risk per trade: 0.4–0.6% ($200–300 on $50k) — NOT 1%
- Position size: 10–15 micros on $50k, 15–20 on $150k, 5–10 pre-market/risky, 4–8 extreme vol
- Max 2 consecutive losses then STOP for the day and lock all funded accounts
- Primary window: 9:30–11:00 AM EST (standard exit at 11 AM)
- Wait 2–5 minutes after open before entering (too volatile immediately)
- Avoid: lunchtime 11:00–12:30, 30 min around news, bank holidays, last 2 weeks Dec, Christmas week
- Half size in choppy/thin/holiday/election/FOMC conditions
- Bad day range: 70–100 pts (skip or A+ only), Terrible: <70 pts (DO NOT TRADE)
- Good day range: 200–400 pts
- Quick check: if still in 15m opening range after 2–3 candles (~10:15 AM) = bad day

### V3 Methodology Evolution Phases (from 281 transcriptions)
1. **Foundation (Oct 2024):** IFVG basics, liquidity sweep entries, 2-trim exit
2. **SMT Mastery (Nov 2024):** NQ vs ES divergence becomes standard confluence
3. **Advanced Entries (Dec 2024):** Order blocks, BPR, data candle setups
4. **Integration (Jan–Mar 2025):** Full system maturity, 88% WR, base hit focus
5. **Refinement (Apr 2025–Jan 2026):** Conservative trimming (55%), soft stops default, SMT primary, funded account discipline

### What Still Needs Extraction
- ~43 remaining trade recap videos (Whisper running on Mac — 281/324 complete)
- 42 premium education videos (titles known, content not yet transcribed — THESE ARE GOLD)
- 18 YouTube premium classes (permanent links, transcription script ready: `transcribe_youtube_classes.py`)
- Tempo Trades Indicator source/settings (available at whop.com)

---

*This document is the single source of truth for project state. Update it at the end of every session.*
