# Technical Implementation Breakdown — Phase 05

## Components & Interfaces
- **Extraction Worker CLI**: Orchestrates URL classification, `yt-dlp`, `ffmpeg`, and auto-retry logic.
- **Asset Manager**: Writes outputs (`source.mp4`, metadata.json) and logs to tenant-scoped directories.
- **Health Reporter**: Sends heartbeat/status updates to Job State Service.

## Draft Folder Layout & Artifacts
```
phase-05/
├── workers/
│   ├── extractor/
│   └── classifiers/
├── scripts/
│   └── url-classifier.py
├── configs/
│   └── yt-dlp/
└── implementation.md
```

## Data Flow & API Sequence
1. Queue message consumed → worker validates job metadata.
2. URL classifier determines path (direct, playlist, platform) and selects extractor.
3. Asset saved to `/tmp/video-jobs/{jobId}/` and metadata recorded; status posted to job service.

## Integrations & Services
- `yt-dlp`, `ffmpeg`, `aria2c` binaries invoked via Node/Python wrappers.
- Job state service updates (Phase 04) and monitoring (Phase 09) telemetry ingestion.

## Observability + Security
- Extraction logs shipped to Loki/ELK (Phase 09) with tenant/job tags.
- Workers run in sandboxed containers with limited network egress (Phase 10).

## Deliverables
- Production extraction worker images, configuration repo, and documentation for scaling new extractor types.
