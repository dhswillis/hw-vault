---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Downloads/Video_Ingestion_Pipeline_Guide.docx
related:
  - wiki/entities/tempo-methodology.md
  - wiki/summaries/tempo-rules-v3.md
  - wiki/maps/automation-moc.md
tags: [summary, pipeline, video-extraction, educators]
---

# Video Ingestion Pipeline Guide — Summary

> Two-phase hybrid approach: manual Claude Code extraction for first 2–3 strategy brains, then automated video AI (Twelve Labs / Gemini API) for scaling to hundreds of hours.

## Key findings

Outlines the pipeline for extracting trading strategies from five educators (Tempo, Lanto, Carmine, Omega, Flipping Markets) and converting them into codified strategy brains. Content comes from Discord and YouTube, flows through local download, AI extraction, and produces codified strategy specifications.

Three methods for building the video URL list: manual collection (best for <30 videos), Discord Chat Exporter (for large Discord libraries), and YouTube API/playlist scraping. The recommended approach starts manual with Claude Code for fast iteration, then scales to automated pipelines once the format is proven.

## Status / caveats

Pipeline guide from Feb 2026. The Tempo extraction (281 transcripts) is already complete — see [[wiki/summaries/tempo-rules-v3|Tempo Rules V3]]. The other four educators have not yet been extracted.
