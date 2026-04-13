---
created: 2026-04-12
updated: 2026-04-12
type: summary
sources:
  - raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/TEMPO_RULES_IMPLEMENTATION.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/QC_SETUP_GUIDE.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/README_QUANTCONNECT_IMPLEMENTATION.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/DATA_SETUP_GUIDE.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_pipeline/upload_report.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_summary.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_extracted/tempo_rules_from_videos.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_extracted/V2_IMPLEMENTATION_SUMMARY.md
  - raw-sources/imports/2026-04-11/Downloads/tempo_extracted/README_V2.md
related:
  - wiki/summaries/tempo-rules-v3.md
  - wiki/summaries/video-ingestion-pipeline-guide.md
  - wiki/entities/quantconnect.md
  - wiki/maps/tempo-moc.md
tags: [summary, cluster, tempo-pipeline, implementation, qc]
---

# Tempo Pipeline & Extracted Rules — Cluster Summary

> Implementation pipeline for turning Tempo video extractions into QuantConnect strategies. Rules implementation, QC setup guides, data setup, and the V2 extraction intermediate outputs.

## Contents

Two groups of related documents:

**Pipeline docs (tempo_pipeline/):** TEMPO_RULES_IMPLEMENTATION.md is the codified version of Tempo's rules for QC implementation. QC_SETUP_GUIDE.md and README_QUANTCONNECT_IMPLEMENTATION.md cover QuantConnect project setup. DATA_SETUP_GUIDE.md handles data feeds. upload_report.md logs a specific upload session.

**Extraction intermediates (tempo_extracted/):** tempo_rules_summary.md and tempo_rules_from_videos.md are intermediate extraction outputs from the video ingestion pipeline. V2_IMPLEMENTATION_SUMMARY.md and README_V2.md document the V2 extraction iteration.

## Status / caveats

These are the intermediate artifacts that produced [[wiki/summaries/tempo-rules-v3|Tempo Rules V3]]. The V2 extraction intermediates are superseded by V3. The QC setup guides are still operationally relevant for the [[wiki/entities/quantconnect|QuantConnect]] project.
