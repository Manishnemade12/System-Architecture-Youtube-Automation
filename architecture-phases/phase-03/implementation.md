# Technical Implementation Breakdown — Phase 03

## Components & Interfaces
- **API Gateway/NestJS BFF**: Handles TLS termination, validation, rate limits, and tenant context injection.
- **Schema Validation Service**: JSON schema definitions for `/jobs`, `/channels`, and webhook payloads.
- **Security Layer**: WAF rules, bot detection, API key verification.

## Draft Folder Layout & Artifacts
```
phase-03/
├── gateway-config/
│   ├── kong/
│   └── envoy/
├── policies/
│   └── json-schema/
├── services/
│   └── bff/
├── certs/
└── implementation.md
```

## Data Flow & API Sequence
1. Client requests `POST /jobs` → gateway validates JWT + tenant quotas.
2. Gateway transforms request into job command and publishes to queue service.
3. Response includes jobId + queue metadata; request traced via OpenTelemetry.

## Integrations & Services
- Identity service (Phase 02) for token validation.
- Queue service (Phase 04) for command forwarding.
- Webhook/event endpoints for third-party integrations secured by HMAC.

## Observability + Security
- WAF logs piped to observability stack (Phase 09).
- Rate limiting per tenant enforced by `ngx_lua` or gateway plugin with per-tenant buckets.

## Deliverables
- Gateway configuration, schema definitions, OpenTelemetry instrumentation, and API documentation for frontend/backends.
