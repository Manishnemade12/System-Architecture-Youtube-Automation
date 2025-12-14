# Phase 12: Client UI and Orchestration Experience

## What problem this phase solves
Delivers the user interface for video submissions, job monitoring, channel selection, and error remediation while orchestrating backend interactions.

## Why this phase is required at this stage
All backend infrastructure needs an orchestrated UI to make the product usable; we cannot ship without a coherent experience for creators/teams.

## What exact features/components are implemented
- Web client allowing users to paste URLs, choose channels, provide metadata, and trigger jobs.
- Real-time job status view (queued, downloading, uploading, completed, failed).
- Channel management console showing OAuth status and quotas.
- Notification center for retries, errors, and approvals.

## Technical design decisions
- Build SPA using React+TypeScript with strict typing for forms (React Hook Form + Zod).
- Use WebSockets or Server-Sent Events for job status updates to reduce polling.
- Provide multi-tab workspace for agencies managing multiple channels.

## Technologies, tools, services, and frameworks required
- Frontend stack: React, Next.js for SSR, TailwindCSS for consistent styling.
- State management: Zustand or Redux Toolkit for job-state caching.
- Charting library (Recharts) for analytic summaries.

## APIs, services, and integrations involved
- Connects to Phase 03 API endpoints for job creation and status.
- Subscribes to real-time updates (e.g., via Redis Pub/Sub or SSE) from job status service.
- Integrates with notification service from Phase 11.

## Database/storage considerations
- Frontend stores user preferences locally (IndexedDB) while respecting tenant isolation in backend.
- Caches channel metadata for offline UX but refreshes periodically to reflect OAuth re-consent requirements.

## Security considerations
- Protects against CSRF/Clickjacking by implementing `SameSite` cookies and CSP.
- Input sanitization on client to prevent injection (complement to server validation).

## Performance & scalability considerations
- Lazy-load large components (job logs) and code-split per route.
- CDN for static assets to keep load times low globally.

## Dependencies on previous phases
- Needs API contracts from Phase 03 and analytics APIs from Phase 11.
- Authentication tokens from Phase 02/08.

## What output/deliverable this phase produces
- Production-ready frontend experience covering job submission, monitoring, and channel management.
- Documentation of API contracts used by the UI.
