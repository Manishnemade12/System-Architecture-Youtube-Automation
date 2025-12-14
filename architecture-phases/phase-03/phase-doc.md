# Phase 03: API Gateway, BFF, and Ingress Security

## What problem this phase solves
Provides consistent public APIs, handles protocol translation for the frontend, and enforces perimeter security before jobs reach internal services.

## Why this phase is required at this stage
Frontend and integration bots depend on stable entry points with authentication/authorization enforced uniformly; we also need early filtering for malformed requests, rate limiting, and observability instrumentation.

## What exact features/components are implemented
- API gateway (e.g., AWS API Gateway, Kong, or custom Express/NestJS BFF).
- Client-facing REST endpoints (`POST /jobs`, `GET /jobs/:id/status`, `GET /channels`).
- Global rate limiting, request validation, JSON schema enforcement.
- Request tracing headers injected for observability.

## Technical design decisions
- Place a lightweight BFF that orchestrates multi-tenant context resolution, merges metadata, and forwards to queue manager.
- API gateway handles TLS termination, WAF rules (SQLi/XSS), and simple bot-blocking (e.g., CAPTCHA challenge when suspicious).
- Use HTTP/2 for lower latency on metadata fetch endpoints.

## Technologies, tools, services, and frameworks required
- Backend framework: Node.js with NestJS (modular) or FastAPI for speed and typing.
- API gateway like Kong + declarative config, or self-hosted Envoy for advanced filters.
- Use OpenTelemetry instrumentation at the gateway level.

## APIs, services, and integrations involved
- Route requests to internal job orchestration service (Phase 04).
- Validate tokens via identity service from Phase 02.
- Provide webhook endpoints for third-party integrations (optional) with HMAC verification.

## Database/storage considerations
- Gateway stores request logs in time-series store (e.g., Loki or CloudWatch) including latency buckets.
- Stores job request metadata (metadata JSON) in PostgreSQL for auditability.

## Security considerations
- Strict input validation to avoid injection.
- API key/secret pair per agency integration to limit abuse.
- Rate limits per tenant/channel defined here.
- TLS 1.3 enforced with HSTS.

## Performance & scalability considerations
- Autoscaling gateway pods behind load balancer; queue worker uses message bus to avoid head-of-line blocking.
- Caching layer for static metadata, such as channel configurations.

## Dependencies on previous phases
- Relies on Phase 01 requirements and Phase 02 identity context.

## What output/deliverable this phase produces
- Hardened ingress with documented endpoints, schemas, and security protections.
- API contract ready for frontend integration.
