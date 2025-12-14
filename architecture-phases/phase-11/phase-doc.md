# Phase 11: Metadata, Tagging, and Analytics Pipeline

## What problem this phase solves
Captures granular metadata about videos, jobs, and tenants so analytics, search, and reporting can be built on consistent, queryable data.

## Why this phase is required at this stage
Insights (e.g., extraction duration, youtube status, tenant trends) are necessary for product management, SLAs, and compliance.

## What exact features/components are implemented
- Metadata enrichment service that records codec, duration, resolution, platform, tenant, job timings.
- Tagging/labeling interface for manual overrides (e.g., marketing metadata).
- Analytics datastore optimized for queries (e.g., ClickHouse, BigQuery, Postgres + materialized views).
- Reporting dashboards and CSV export API.

## Technical design decisions
- Use event-driven ingestion (Kafka or queue) to decouple metadata capture from job flow.
- Store normalized metadata in Postgres for operational queries and replicate to OLAP store for analytics.
- Provide tenant-specific views respecting row-level security.

## Technologies, tools, services, and frameworks required
- Kafka/Redis streams for metadata events.
- ClickHouse or Amazon Redshift for analytics.
- Superset/Grafana for dashboards.

## APIs, services, and integrations involved
- Jobs publish metadata events post-extraction/upload.
- Dashboard service queries analytics store and exposes data to clients.
- Webhook API for integrations (e.g., to notify agencies or CRMs).

## Database/storage considerations
- Partitioned tables by tenant for both Postgres and analytics store.
- Materialized views for common queries (jobs by tenant, success rate, latency percentiles).

## Security considerations
- Row-level security ensures tenants see only their data.
- Mask sensitive metadata before storage (e.g., user identifiers replaced with tokens).

## Performance & scalability considerations
- Use asynchronous ingestion to keep job throughput unaffected.
- Optimize OLAP store for high-cardinality metrics (use Bloom filters, indexing strategies).

## Dependencies on previous phases
- Relies on completed job data from Phases 04-07.
- Requires tenant context from Phase 02.

## What output/deliverable this phase produces
- Analytics-ready metadata lake, dashboards, and tenant-level reporting tools.
- Search index for jobs based on tags and metadata.
