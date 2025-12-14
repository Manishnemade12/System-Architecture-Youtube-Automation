# Phase 08: OAuth Token Management and Channel Provisioning

## What problem this phase solves
Manages Google OAuth consent flows, securely stores tokens per user/channel, and keeps tokens refreshed for upload workers.

## Why this phase is required at this stage
Upload workers need valid YouTube tokens. Without centralized token lifecycle management, uploads will fail when tokens expire or are revoked.

## What exact features/components are implemented
- OAuth consent flow triggered when adding a new channel.
- Token exchange service that stores access + refresh tokens with expiry metadata.
- Scheduled refresh job to preemptively refresh tokens and cache metadata per tenant/channel.
- Token revocation UI/API for admins.

## Technical design decisions
- Use server-side OAuth flow; do not expose refresh tokens to the browser.
- Store refresh tokens encrypted with a KMS-managed key and access tokens in a short-lived cache (Redis).
- Provide token rotation capability: if refresh fails, mark channel as needing re-consent and alert user.

## Technologies, tools, services, and frameworks required
- Use Google OAuth 2.0 endpoints (`https://accounts.google.com/o/oauth2/v2/auth` and `token`).
- Key management via AWS KMS or Vault to encrypt refresh tokens.
- Redis for caching short-lived access tokens.

## APIs, services, and integrations involved
- Gateway (Phase 03) forwards OAuth callbacks to this service.
- Upload worker (Phase 07) reads tokens via gRPC/REST API and caches them locally per session.
- Notification service (Phase 12) alerts user when re-consent required.

## Database/storage considerations
- `channels` table maintains `refresh_token_encrypted`, `access_token_cache_key`, `token_expiry`, `last_refresh`.
- Audit logs track consent time and IPs for compliance.

## Security considerations
- Rotate encryption keys per tenant; do not store tokens in plain text.
- Access control ensures only owner/listed admins can trigger refresh or revocation.
- Use proof-of-possession to ensure tokens used by legitimate workers only.

## Performance & scalability considerations
- Use refresh job shards per region to avoid hitting Google rate limits.
- Use caching to reduce repeated decryptions for uploads.

## Dependencies on previous phases
- Depends on Phase 02 identity data for channel ownership.
- Requires Phase 03 for callback routes.

## What output/deliverable this phase produces
- Token management service with secure storage, refresh automation, and audit logs.
- Channels ready for upload workers with valid credentials.
