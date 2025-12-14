# Technical Implementation Breakdown — Phase 06

## Components & Interfaces
- **Storage Layout Manager**: Creates per-job directories with `source.mp4`, `metadata.json`, `logs.txt` and registers paths in Postgres.
- **Cleanup Daemon**: Cron job that deletes expired artifacts and updates job statuses with cleanup timestamps.
- **Object Storage Bridge**: Optional process that uploads large files to S3/MinIO when local disk is insufficient.

## Draft Folder Layout & Artifacts
```
phase-06/
├── storage-patterns/
│   └── layout.md
├── services/
│   ├── cleanup-service/
│   └── object-storage-sync/
└── implementation.md
```

## Data Flow & API Sequence
1. Extraction worker writes files to local volume → Storage Manager records path + size.
2. Cleanup job monitors TTLs → removes directories and records audit entries.
3. Upload worker reads metadata to locate file for transfer.

## Integrations & Services
- Extraction worker writes to local PV (Phase 05).
- Upload worker (Phase 07) relies on deterministic path naming.
- Monitoring (Phase 09) tracks disk usage/cleanup failures.

## Observability + Security
- Track cleanup success/failure metrics; log details for auditing.
- Enforce file ACLs so only owning tenant’s worker can read/write.

## Deliverables
- Documented storage blueprint, cleanup automation, and object storage integration plan.
