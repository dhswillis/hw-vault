---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/AI_Trading_Show_Blueprint.docx
related:
  - wiki/summaries/unified-brain-architecture
  - wiki/entities/nova-sable-brains
tags: [summary, ai-show, avatars, youtube, live-stream]
---

# The AI Trading Show Blueprint — Summary

> 2-hour daily YouTube live stream: two AI personalities (Nova, Sable) co-host NQ futures trading with real charts, banter, live trades, zero manual posting.

## Key findings

**Characters:** Nova (analyst, calm, data-driven, "the data says...") + Sable (instinct-driven, high-energy, "feel it in my bones"). Dual-host creates tension/chemistry; disagreements engage viewers, agreement signals high conviction. Different trading styles (Nova score-3 only, Sable score-2+) create natural debate.

**Show format (8:00-10:00 AM ET):** Opening + yesterday recap (2-10 min), pre-market brief (15 min), rules reminder (5 min), live trading (85 min, react to real price action, call setups, celebrate/roast results), closing (5 min, daily R summary, preview next day).

**Tech stack:** Claude API (Sonnet best dialogue) generates real-time scripts as Nova/Sable back-and-forth. ElevenLabs TTS (Flash v2.5, ~75ms latency) voices both characters. Avatar renderer (HeyGen, BocaLive, or self-hosted MuseTalk) streams realistic talking heads. FFmpeg compositor overlays live NQ chart + trade panel + branding. RTMP push to YouTube Live. **Total latency 3-8 sec to viewer** (acceptable; human streams similar).

**Avatar options:** BocaLive (\$58-200/mo) recommended to start; HeyGen (\$120-360/day overkill for production); self-hosted MuseTalk (\$0 after GPU). LLM tier comparison: 14B local ($0-95/mo GPU) serviceable, 70B Llama (\$190-255 GPU) production-quality, DeepSeek V3 API (\$2/mo) near-frontier. ElevenLabs Pro (\$99/mo) covers ~10-15 days streaming with credit management; Scale (\$330/mo) unlimited.

**Growth protocol:** Two AI characters attract viewers seeking education + entertainment. Real IBKR fills + transparent daily P&L build credibility vs hypothetical traders. YouTube algo favors long watch time; 2-hour shows generate massive impressions.

## Status / caveats

Architecture proven (5 layers: market brain → dialogue → TTS → avatar → composer → RTMP). Avatar quality critical—low quality kills show. Recommended path: BocaLive to validate concept, then migrate to self-hosted. LLM + TTS costs modest (~\$55-330/mo) with prompt caching. GPU cost (\$95-425/mo) dominates if self-hosting avatar. News blackout calendar trivial to implement. Integration with AIO system via same strategy signals and Discord bot already built.
