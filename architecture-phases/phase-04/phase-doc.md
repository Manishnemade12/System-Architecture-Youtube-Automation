# Phase 04: Job Queue Infrastructure and Orchestration Layer

## What problem this phase solves
Provides reliable buffering between incoming job requests and extraction/upload workers, enabling retries, ordered processing, and visibility into queue health.

## Why this phase is required at this stage
Job processing is asynchronous and must survive spikes; hooking workers directly to HTTP requests would block API responsiveness and reduce resiliency.

## What exact features/components are implemented
- Queue subsystem for extraction jobs and upload jobs (two logical queues).
- Job record creation with state machine (PENDING → DOWNLOADING → etc.).
- Retry policies (exponential backoff, dead-letter queue) and poison message handling.
- Job locking and visibility leases to prevent duplicate processing.

## Technical design decisions
- Use BullMQ (Redis Streams) or RabbitMQ with TTL/durable queues.
- Separate queues for extraction and upload to enable independent scaling.
- Store job metadata in relational DB but use queue for orchestration signals.
- Provide job status updater microservice.

## Technologies, tools, services, and frameworks required
- Redis (clustered) for BullMQ or RabbitMQ cluster (HA) with mirrored queues.
- Postgres job tables + event logs with transactional writes.
- Worker supervisor (PM2 or Kubernetes Jobs) to restart failed workers automatically.

## APIs, services, and integrations involved
- Gateway (Phase 03) posts job commands.
- Extraction worker (Phase 05) and Upload worker (Phase 07) consume jobs.
- Monitoring service (Phase 09) reads queue metrics via Redis exporter.

## Database/storage considerations
- Job table holds pointers to storage path, metadata JSON, tenant/channel IDs, and state timestamps.
- Log table for each state transition with trace IDs for observability.

## Security considerations
- Enforce tenant access via queue metadata to prevent one tenant from affecting another.
- Mutually authenticated connections between queue consumers and Redis/RabbitMQ.

## Performance & scalability considerations
- Horizontal scaling by adding workers triggered by queue depth metrics.
- Rate limiting per tenant at the queue level to prevent noisy neighbors.
- Backpressure handling: API gateway rejects new jobs when queue depth crosses threshold.

## Dependencies on previous phases
- Requires Phase 01-03 outputs (requirements, identity, API). 

## What output/deliverable this phase produces
- Production-ready job queue plumbing, state machine definitions, and governance around retries.
- Observability hooks for queue pressure.
