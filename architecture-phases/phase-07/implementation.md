# Technical Implementation Breakdown — Phase 07

## Components & Interfaces
- **Upload Worker**: Handles resumable YouTube uploads with chunked streams and metadata validation.
- **Channel Router**: Maps job -> channel-token pair, verifies quotas, and selects OAuth credentials.
- **Upload Tracker**: Persists `uploadUrl`, last byte uploaded, token refresh status.

## Draft Folder Layout & Artifacts
```
phase-07/
├── workers/
│   └── uploader/
├── libs/
│   └── youtube-client/
├── configs/
│   ├── oauth/
│   └── upload-throttling/
└── implementation.md
```

## Data Flow & API Sequence
1. Worker reads job metadata (storage path, channelId) → loads token via token service (Phase 08).
2. Initiates resumable `videos.insert` session → saves `uploadUrl` & bytes.
3. Upon completion, updates job state to `COMPLETED` and pushes metadata to analytics (Phase 11).

## Integrations & Services
- YouTube Data API v3 for uploads.
- OAuth token management service (Phase 08) for tokens.
- Monitoring (Phase 09) for upload metrics.

## Observability + Security
- Emit upload progress and failure details; detect token expiry events.
- Restrict Google API scopes to `youtube.upload` and `youtube.readonly`. 

## Deliverables
- Upload worker container, resumable upload logic, token refresh hooks, and performance instrumentation.
