# Technical Implementation Breakdown — Phase 04

## Components & Interfaces
- **Queue Manager**: BullMQ/RabbitMQ client abstractions for pushing extraction/upload jobs.
- **Job State Service**: Manages job lifecycle (PENDING → DOWNLOADING → ...) and emits events for monitoring.
- **Retry & Dead Letter Handler**: Tracks attempts, backoff timers, and stores poisoned jobs.

## Draft Folder Layout & Artifacts
```
phase-04/
├── queues/
│   ├── extraction/
│   └── upload/
├── services/
│   ├── job-state/
│   └── retry/
├── scripts/
│   └── sweep-deadletter.js
└── implementation.md
```

## Data Flow & API Sequence
1. BFF service issues job create command → stored in Postgres + queue message.
2. Job state service updates `jobs` table and writes state transition log.
3. Worker leases job via Redis lock with TTL; failure releases lock and requeues.

## Integrations & Services
- Gateway for receiving commands (Phase 03).
- Extraction and upload workers (Phases 05 & 07) subscribe to respective queues.
- Monitoring (Phase 09) reads queue metrics via Redis exporter.

## Observability + Security
- Instrument queue depth, processing time, failure rate with Prometheus.
- Tenant metadata used to tag metrics; queue access controlled by mutual TLS.

## Deliverables
- Queue infrastructure, job lifecycle API, retry policies, and configuration for worker scaling.
