---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/trading/tempo/Unified_Brain_Options_and_Growth.docx
related:
  - wiki/summaries/unified-brain-architecture
  - wiki/entities/nova-sable-brains
tags: [summary, architecture, cost-analysis, monetization]
---

# Unified Brain — Architecture Options & Growth Plan — Summary

> Three deployment tiers (PoC $180/mo, Production $420-520/mo, Premium $600+/mo) with honest pricing and quality tradeoffs; multi-tier LLM/GPU/TTS/avatar options.

## Key findings

**LLM options (monthly cost at stream volume):** DeepSeek R1 14B local ($0), Llama 70B Q4 local ($0), DeepSeek V3 API ($2), Claude Haiku ($15), GPT-4o ($40), Claude Sonnet ($55, **best dialogue**), Claude Opus ($80). Dialogue quality increases with larger/better models; API costs modest with prompt caching (90% savings on repeated context).

**GPU infrastructure (24/7 rental):** 1x RTX 3090 ($95/mo), 2x RTX 3090 ($190/mo), 1x A6000 ($255-365/mo), 1x RTX 4090 ($212/mo), 2x RTX 4090 ($424/mo recommended premium). 14B models fit single 24GB; 70B needs ~43-48GB (2x 3090 or 1x A6000).

**Text-to-speech (critical quality):** ElevenLabs Flash v2.5 best-in-class, 75ms latency. Creator $22/mo (100K credits ~2-3 days streaming), Pro $99/mo (500K credits ~10-15 days), Scale $330/mo (2M credits unlimited month). Self-hosted (Coqui, Bark) $0 but lower quality.

**Avatar options:** BocaLive Basic $58/mo (good quality, 10-min setup), BocaLive Pro $128/mo (custom avatars), HeyGen LiveAvatar ($120-360/day overkill), self-hosted MuseTalk $0 (30+ FPS lip-sync on GPU, complex setup). HeyGen per-minute pricing ($0.50-1.50) = $120-360/day for 2-hour dual-avatar stream ($2,400-7,200/mo).

**Tier 1: PoC ($180-240/mo):** 14B local + 1x RTX 3090 + ElevenLabs Creator + BocaLive Basic. Validates concept before major investment. Sparse dialogue due to credit limits.

**Tier 2: Production ($420-520/mo — RECOMMENDED):** 70B local (Llama/Qwen) + 2x RTX 3090 or 1x A6000 + ElevenLabs Pro + BocaLive Pro. Near-frontier dialogue, unlimited streaming, professional avatars.

**Tier 3: Premium ($600+/mo):** 70B + 2x RTX 4090 + ElevenLabs Scale + self-hosted MuseTalk or BocaLive Ultra. Maximum independence and quality.

**Key insight:** API (Claude Sonnet $55/mo) dramatically cheaper than expected at stream volume. Hybrid approach: local brain for memory + research, cloud API for dialogue polish = best of both.

## Status / caveats

April 2026 pricing from Vast.ai, RunPod, ElevenLabs, BocaLive, HeyGen. Self-hosted avatar path requires engineering effort but eliminates recurring avatar costs after GPU amortizes. No show-stoppers. Regulatory note: Copy trade tier requires investment adviser registration (defer). Start with Tier 2 (PoC if unsure, then scale). BocaLive production-ready day 1; self-hosted avatar migration later.
