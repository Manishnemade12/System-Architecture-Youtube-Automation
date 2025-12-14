# Architecture Issues and Risk Log

## 1. Reliance on third-party extraction tools (yt-dlp / ffmpeg)
- **What the issue is**: `yt-dlp` and `ffmpeg` are community-maintained and reverse-engineering platforms like YouTube, Instagram, and Facebook introduces a continuous maintenance burden.
- **Why it is a problem**: These tools can break when platforms change their delivery mechanisms or enforce stricter rate limiting; this directly impacts extraction worker stability and may require legal review.
- **Possible alternatives or workarounds**: Maintain a dedicated team to monitor upstream updates, contribute patches, regularly run compatibility tests, and implement feature toggles that can disable unsupported platforms temporarily. Evaluate licensed content providers or official APIs where available (e.g., Instagram Graph API for business accounts) to reduce dependency on scraping.

## 2. YouTube API quota limits for large multi-channel customers
- **What the issue is**: YouTube enforces quotas per project, and each `videos.insert` consumes a significant quota (~1600 units per upload). Scaling to many channels and frequent uploads can exhaust daily quotas.
- **Why it is a problem**: Once quota exhausted, uploads fail until refreshed, causing SLA breaches and poor UX. Relying solely on one Google project amplifies this risk.
- **Possible alternatives or workarounds**: Implement quota tracking per tenant, throttle uploads nearing quota limits, and support multiple Google Cloud projects (with associated OAuth credentials) or per-tenant quota reservation. Engage with Google to request quota increases and monitor usage trends with analytics (Phase 11).
