# Phase 07: YouTube Upload Worker and Channel Routing

## What problem this phase solves
Handles high-latency, authenticated uploads to the YouTube Data API, ensuring each job posts to the correct channel with resumable transfers.

## Why this phase is required at this stage
After extraction, the platform must push videos into YouTube without blocking other jobs. This phase closes the loop on the video-processing pipeline.

## What exact features/components are implemented
- Upload worker that reads local files and metadata.
- Resumable uploads via `videos.insert` with `mediaBody` and `part=snippet,status`.
- Channel routing logic that selects the correct OAuth token per job.
- Upload status updates and retries (exponential backoff with jitter).

## Technical design decisions
- Use YouTube API resumable upload session URLs to survive transient network errors.
- Keep uploads idempotent by storing `uploadId` per job.
- Validate metadata (title, description, tags, privacy status) before uploading.

## Technologies, tools, services, and frameworks required
- Google client libraries (python/Node) that wrap `youtube.videos.insert`.
- Worker runs in container with network access to YouTube and token store.
- Persistent state store (Postgres) for upload progress, `uploadUrl`, and etags.

## APIs, services, and integrations involved
- YouTube Data API v3 with proper quotas.
- Token management service from Phase 08.
- Optional CDN (CloudFront) for verifying video sizes before upload if needed.

## Database/storage considerations
- Table storing `jobId`, `uploadSessionUrl`, `lastByteUploaded`, `etag`, `uploadAttempts`.
- Link to storage path for the file being uploaded to ensure worker reads right file.

## Security considerations
- Only the tenant’s upload worker can access channel tokens.
- Refresh tokens stored encrypted; decrypt only when necessary in secure environment.
- Monitor for suspicious API usage (ae. repeated failures) to detect compromised credentials.

## Performance & scalability considerations
- Upload workers horizontally scale by spawning pods per region; use queue depth as scaling signal.
- Track YouTube quota usage per channel and throttle if nearing limit.
- Use chunked uploads to sub 10MB segments to avoid large retries.

## Dependencies on previous phases
- Depends on Phases 05 and 06 for extracted files.
- Uses Phase 08 token store to load channel credentials.

## What output/deliverable this phase produces
- Working upload worker with channel-specific routing and retries.
- Completed job states when uploads finalize and metadata stored for audits.
