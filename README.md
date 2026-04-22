# Mizzy Tools

**Private all-in-one creator dashboard. Media downloads, creator analytics, influencer scoring, and an AI-powered music industry learning suite.**

> A password-protected, self-hosted multi-tool dashboard for independent creators. Includes a streaming media downloader, growth analytics, influencer scoring engine, and a music industry flashcard quiz — all served through a permanent Cloudflare Named Tunnel with zero port exposure.

---

## Architecture

```
Browser (authenticated session)
  │
  ▼
Next.js 15  (App Router · SSR · password-gated · Cloudflare Workers)
  │
  ├── Tab: Download     ── Cloudflare Named Tunnel ──▶ yt-dlp Backend (Node.js + ffmpeg)
  ├── Tab: Influnx Calc ── Scoring Engine (TypeScript, client-side)
  ├── Tab: Growth Report── /api/growth-report SSE ──▶ Anthropic SDK (streaming)
  └── Tab: AI Learn     ── /api/ai-learn SSE ──────▶ Music industry flashcard engine
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, App Router, React, Cloudflare Workers (via @opennextjs/cloudflare) |
| Media Engine | yt-dlp (pinned version), ffmpeg |
| Tunnel | Cloudflare Named Tunnel (cloudflared) |
| AI | Anthropic SDK, Claude claude-sonnet-4-6, Server-Sent Events |
| Auth | Web Crypto API — password gate (no JWT library) |
| Testing | Vitest, axe-core (8 test files, accessibility-compliant) |

---

## Tools

### Media Downloader
- **Multi-platform** — YouTube, SoundCloud, Instagram, TikTok, Twitter/X
- **Format selection** — MP3 (audio-only) or MP4 (video+audio); auto mode detects optimal format
- **Streaming download** — no temp files on server; streams directly to client (piped via ffmpeg)
- **Download history** — session-scoped log of recent downloads
- yt-dlp pinned to verified version; spawn calls use `--` separator (prevents argument injection)

### Influnx Calc
- 100-point influencer scoring engine with 6 configurable metric categories
- Platform-specific normalization (engagement rates differ by platform)
- Penalty layer detects bought followers (engagement/follower ratio outliers)
- Pure TypeScript functions — sub-100ms per calculation, ±2-point accuracy

### Growth Report AI
- Streaming AI-generated growth narrative via SSE (report lines arrive as Claude generates them)
- Period-over-period comparison (week/month/quarter) across Spotify, YouTube, Apple Music, TikTok, IG
- Extended prompt caching on system prompt + reference data (5-min TTL)
- Retry with exponential backoff on Anthropic API errors

### AI Learn — Music Industry Quiz
- Flashcard and checklist quiz covering the music industry
- 5 learning tracks · 60 flashcards · 65 checklist items · 16 milestone badges
- Progress persists across sessions

---

## Security Design

### Access Control
- Password-gated at the application layer — no unauthenticated access to any endpoint
- Session token derived via Web Crypto API (`crypto.subtle.digest`) — no Node.js `crypto` module
- Session token stored in `httpOnly` cookie; inaccessible to client JavaScript
- All traffic routed through Cloudflare Named Tunnel — no ports exposed to the public internet

### Infrastructure
- Backend processes only accessible via the Named Tunnel — no direct internet exposure
- Cloudflare Tunnel acts as the sole ingress — DDoS protection + TLS termination at edge
- `cloudflared` daemon managed by process supervisor with automatic restart on failure

### Edge Compatibility
- Fully edge-runtime compatible: all Node.js crypto/Buffer replaced with Web Crypto API equivalents
- No `runtime = 'edge'` on individual routes — @opennextjs/cloudflare manages the edge context
- Webpack build (not Turbopack) — Turbopack chunks are incompatible with CF Workers deployment
- Preflight gate blocks deploy if any of 8 incompatibility checks fail (crypto, Buffer, runtime=edge flags, Turbopack, missing wrangler config)

---

## Deployment

Built with `@opennextjs/cloudflare` and deployed to Cloudflare Workers — same deploy pattern as the main landing site. Runs globally at the edge with zero cold starts.

Locally, the yt-dlp backend runs as a Docker container accessed via Cloudflare Named Tunnel:
- Named tunnel assigns a permanent URL — does not change between restarts
- Watchdog process restarts the tunnel on silence
- Exponential backoff on reconnect failures

---

## Recent Additions

- **CF Workers migration** — migrated from Vercel to Cloudflare Workers via @opennextjs/cloudflare; all Node.js crypto replaced with Web Crypto API
- **Integrated tool suite** — added Influnx Calc, Growth Report AI, and AI Learn as dashboard tabs
- **Music industry quiz** — 5 tracks, 60 cards, 65 checklist items, 16 milestones (replaced previous chat interface)
- **QA harness** — Vitest + axe-core test suite (8 files), WCAG 2.1 SC 1.3.1 accessibility compliance
- **Deploy preflight gate** — blocks CF Workers deploy on 8 incompatibility checks

---

## Why Cloudflare Named Tunnels over Port Forwarding

| Approach | Public IP exposure | TLS | DDoS protection | URL stability |
|---|---|---|---|---|
| Port forwarding | Yes | Manual | No | IP-dependent |
| Cloudflare Named Tunnel | No | Automatic | Yes | Permanent |

Named tunnels assign a permanent subdomain (or custom domain) that routes through Cloudflare's network to your local process via an outbound-only connection. The host machine never exposes a public port.

---

## License

MIT — see [LICENSE](LICENSE)
