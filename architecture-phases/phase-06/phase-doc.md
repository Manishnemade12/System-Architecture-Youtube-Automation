# Phase 06: Local, Temporary, and Persistent Storage Strategy

## What problem this phase solves
Defines where extracted videos, logs, metadata, and uploads live, and ensures storage is performant, antivirus-safe, and cleaned up after consumption.

## Why this phase is required at this stage
Extraction workers produce large files; without a disciplined storage plan, disk exhaustion, inconsistent cleanup, and upload errors become inevitable.

## What exact features/components are implemented
- Directory layout (`/tmp/video-jobs/{jobId}`) with source.mp4, metadata.json, logs.
- Local disk dance: ensure volumes mount per worker, with quotas.
- Optional intermediate object storage (S3/GCS) for long-running jobs or large teams.
- TTL cleanup service that runs after job completion/failure.

## Technical design decisions
- Prefer local disk for initial extraction to minimize transfer latency; only move to object storage when needed.
- Use immutable storage identifiers (jobId + tenantId) to prevent cross-tenant exposure.
- Provide metrics on disk usage, last access timestamp, and cleanup sweeps.

## Technologies, tools, services, and frameworks required
- Kubernetes PersistentVolumes with `ReadWriteOnce` and local SSDs where possible.
- Object storage (S3-compatible like MinIO or AWS S3) for staging if jobs exceed local capacity.
- Cleanup service as Kubernetes cronjob or AWS Lambda that deletes directories older than TTL.

## APIs, services, and integrations involved
- Extraction workers save to local pods and optionally copy to S3 via AWS SDK or MinIO client.
- Upload worker reads from storage, so consistent naming is critical.

## Database/storage considerations
- Metadata namespace stored in Postgres pointing to the storage URL/path.
- Cleanup job logs stored to track deletions for audits.

## Security considerations
- Encrypt files at rest using filesystem encryption (dm-crypt) or leverage underlying PV encryption.
- Enforce ACLs: only the owning worker and upload process can read a job directory.
- Sanitize metadata to avoid secret leakage.

## Performance & scalability considerations
- Use local SSDs for hot jobs, tier to object storage for cold data.
- Limit concurrent extractions per node to avoid I/O storms.
- Implement disk pressure alerts (node exporter) to add nodes before exhaustion.

## Dependencies on previous phases
- Depends on Phase 05 extraction workers and Phase 04 orchestration to know job locations.

## What output/deliverable this phase produces
- Documented storage layout, TTL policy, and encryption/compliance controls.
- Automated cleanup job and storage monitoring metrics.
