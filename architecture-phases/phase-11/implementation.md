# Technical Implementation Breakdown — Phase 11

## Components & Interfaces
- **Metadata Enricher**: Listens to job events and extracts codec, duration, platform info, storage footprint.
- **Analytics OLAP Layer**: ClickHouse/BigQuery tables with tenant-partitioned views.
- **Tag & Label Service**: Allows manual metadata overrides via API.

## Draft Folder Layout & Artifacts
```
phase-11/
├── pipelines/
│   └── metadata-ingest/
├── analytics/
│   └── dashboards/
└── implementation.md
```

## Data Flow & API Sequence
1. Job state service emits event → metadata enricher consumes via Kafka.
2. Normalized record written to Postgres and replicated to ClickHouse.
3. APIs serve aggregated metrics to dashboards and export endpoints.

## Integrations & Services
- Event bus (Kafka/Redis) for metadata propagation.
- Analytics dashboards (Superset/Grafana) for tenant analytics.
- Notification service (Phase 12) uses metrics to inform users.

## Observability + Security
- Use row-level security in analytics store.
- Mask user IDs before exporting data to third parties.

## Deliverables
- Metadata pipeline, analytics dataset, and tagging service documentation for product teams.
