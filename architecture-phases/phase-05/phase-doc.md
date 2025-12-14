# Phase 05: Extraction Worker Infrastructure

## What problem this phase solves
Handles the actual download/extraction of media assets, transforming URLs (direct, playlist, platforms) into local video files ready for upload.

## Why this phase is required at this stage
The heart of the product is extracting videos; without a scalable worker layer, the platform cannot perform the core operation.

## What exact features/components are implemented
- Extraction worker service that processes queue messages.
- URL classifier (direct file, HLS/DASH playlist, platform link).
- Integration with `yt-dlp`/`ffmpeg` plus fallback strategies.
- Logging of extraction progress and output metadata.

## Technical design decisions
- Containerized workers that mount storage volumes and run `yt-dlp` via wrappers for predictable behavior.
- Use `yt-dlp` for platform URLs and custom ffmpeg scripts for streaming playlists.
- Worker emits checkpoints (downloaded, merged) so resumable extraction is possible.

## Technologies, tools, services, and frameworks required
- Docker/Kubernetes job pods with CPU/memory limits.
- `yt-dlp`, `ffmpeg`, `aria2c` for segmented downloads.
- Shared libraries for sanitization/parsing of URLs and user metadata.

## APIs, services, and integrations involved
- Consumes extraction queue from Phase 04.
- Emits status updates via job status service to Postgres/Redis.
- Optionally calls third-party CAPTCHA solving? (if required for some platforms, document risk in issues file).

## Database/storage considerations
- Writes `source.mp4` and logs to tenant-specific local directories (`/tmp/video-jobs/{jobId}/`).
- Stores metadata JSON (streams info, codecs, size) in Postgres for auditing.

## Security considerations
- Run workers in sandboxed containers, drop privileges, run in separate namespaces.
- Limit outbound network access to known CDNs; egress firewall ensures only supported hosts.
- Sanitize file names to avoid path traversal.

## Performance & scalability considerations
- Workers autoscale based on queue depth and CPU/memory usage.
- Segment downloads performed with parallel threads to reduce latency.
- Worker caching of frequently used certificates/FFmpeg configs helps speed.

## Dependencies on previous phases
- Extraction queue from Phase 04 and storage layout from Phase 06.
- Requires identity and job metadata from Phases 01-03.

## What output/deliverable this phase produces
- Reliable extraction worker ready for diverse URL types.
- Metadata about extracted assets and versioned logs for troubleshooting.
