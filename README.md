# Mizzy Tools

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=flat&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=flat&logo=cloudflare&logoColor=white)
![Anthropic](https://img.shields.io/badge/Anthropic_Claude-D97706?style=flat&logo=anthropic&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

**Private all-in-one creator dashboard. Media downloads, creator analytics, influencer scoring, and an AI music industry learning suite.**

> Password-protected, self-hosted multi-tool dashboard for independent creators. A streaming media downloader, growth analytics engine, influencer scoring system, and AI-powered music industry flashcard quiz — all served through a permanent Cloudflare Named Tunnel with zero port exposure.

---

## Table of Contents

- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Tools](#tools)
- [Security Design](#security-design)
- [Why Cloudflare Named Tunnels](#why-cloudflare-named-tunnels-over-port-forwarding)
- [Key Engineering Details](#key-engineering-details)
- [Recent Additions](#recent-additions)
- [Running This](#running-this)

---

## Architecture

```
Browser  (password-authenticated session)
  │
  ▼
Next.js 15  (App Router · SSR · Cloudflare Workers)
  │
  ├── Tab: Download      ── Named Tunnel ──▶ yt-dlp Backend  (Node.js + ffmpeg, Docker)
  ├── Tab: Influnx Calc     Scoring Engine (TypeScript — client-side, zero network calls)
  ├── Tab: Growth Report    /api/growth-report SSE ──▶ Anthropic SDK (streaming)
  └── Tab: AI Learn         /api/ai-learn SSE ──────▶ Music industry quiz engine (Claude)
```

The download backend never exposes a public port. All traffic flows through a permanent Cloudflare Named Tunnel — the only ingress to the Docker backend is through Cloudflare's network.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Next.js 15 (App Router) | Cloudflare Workers via @opennextjs/cloudflare |
| Media Engine | yt-dlp (pinned version) + ffmpeg | Docker container, streamed output |
| Tunnel | Cloudflare Named Tunnel (cloudflared) | Permanent public URL, zero port exposure |
| AI | Anthropic SDK, Claude claude-sonnet-4-6 | SSE streaming, prompt caching |
| Auth | Web Crypto API (PBKDF2) | httpOnly cookie session — no JWT library |
| Testing | Vitest + axe-core | 8 test files, WCAG 2.1 AA compliance |

---

## Tools

### Media Downloader
Streaming download for YouTube, SoundCloud, Instagram, TikTok, and Twitter/X.

- **Zero temp files:** yt-dlp output piped directly to the HTTP response — no intermediate disk writes.
- **Auto mode:** Detects optimal format automatically. Audio-only URLs → MP3; video URLs → MP4.
- **Format priority:** Prefers DASH m4a over webm for clean MP3 conversion without quality loss.
- **Zero port exposure:** Download backend in Docker, never directly reachable — all traffic proxied through Cloudflare.
- **Download history:** Session-scoped log of recent downloads.

### Influnx Calc
100-point influencer scoring engine. Client-side TypeScript — no data leaves the browser.

- 6 metric categories with configurable weights
- Per-platform normalization (engagement rates vary significantly between platforms)
- Penalty layer for engagement anomaly detection (bot/bought-follower patterns)
- Sub-100ms calculations via memoized weight tables

### Growth Report AI
AI-generated growth analytics narrative. Real platform data + streaming Claude insights.

- Aggregates Spotify, YouTube, Apple Music, TikTok, and Instagram metrics
- Period-over-period comparison (week / month / quarter)
- Claude generates specific, data-driven narrative observations
- Extended prompt caching on system prompt + reference data (5-min TTL) — ~80% cost reduction on repeat runs

### AI Learn — Music Industry Quiz
Music industry flashcard quiz powered by Claude.

- 5 learning tracks · 60 flashcards · 65 checklist items · 16 milestone badges
- Topics: YouTube algorithm, streaming royalties, sync licensing, playlist pitching, distribution, publishing splits
- Progress persists across sessions

---

## Security Design

### Password Gate
- Dashboard password hashed with **PBKDF2** (Web Crypto API, 100,000 iterations, SHA-256) — no Node.js `crypto` module needed.
- Session stored in an httpOnly, Secure, SameSite=Strict cookie — inaccessible to JavaScript in any browser context.
- Constant-time comparison on every session validation — no timing oracle on cookie values.
- All routes protected by Next.js middleware — no client-side auth state to spoof.
- No unauthenticated endpoint exists in the application.

### Zero Port Exposure
The download backend runs in Docker and is completely inaccessible from the public internet:

```
Browser → Cloudflare network → Named Tunnel daemon → Docker backend
```

The tunnel daemon initiates **outbound-only** connections to Cloudflare. No inbound ports open on the host. No firewall rules required — the host is not reachable directly.

### yt-dlp Hardening
- yt-dlp binary pinned to a specific version — no auto-updates that could introduce regressions.
- URLs validated server-side against an allowlist before being passed to yt-dlp.
- `--` separator between flags and URL arguments — prevents argument injection.
- SSRF protection: only known media platform domains are accepted.

### AI API Security
- API credentials stored server-side in environment variables — never exposed to client JavaScript.
- User input passed to AI as structured data, never interpolated directly into prompts.
- Prompt injection mitigation: user-provided strings are treated as `user` role content, not `system` role instructions.
- Output sanitized before rendering in the chat interface.

### Edge Compatibility
- Fully edge-runtime compatible: all Node.js `crypto` and `Buffer` usage replaced with Web Crypto API.
- Deploy preflight gate blocks build if any of 8 CF Workers incompatibility checks fail (crypto, Buffer, `runtime = 'edge'` annotations, Turbopack, missing wrangler config).

---

## Why Cloudflare Named Tunnels over Port Forwarding

| Approach | Public IP exposed | TLS | DDoS protection | URL stability |
|---|---|---|---|---|
| Port forwarding | Yes | Manual | No | IP-dependent |
| Cloudflare Named Tunnel | **No** | **Automatic** | **Yes** | **Permanent** |

Named tunnels assign a permanent subdomain (or custom domain) routing through Cloudflare's network to the local process via an outbound-only connection. The host machine never opens a public port.

---

## Key Engineering Details

- **Streaming downloads:** yt-dlp output piped directly to the HTTP response stream — memory usage is constant regardless of file size, no buffering.
- **Web Crypto auth:** PBKDF2 password hashing runs natively on Cloudflare Workers edge — no Node.js dependencies needed.
- **SSE for AI:** Reports and quiz responses stream token-by-token via Server-Sent Events — content appears as it's generated.
- **Prompt caching:** System prompt for growth reports and quiz engine cached at 5-minute TTL — repeat requests cost ~80% less.
- **Accessibility:** axe-core integrated in tests — all interactive elements verified for WCAG 2.1 AA compliance.

---

## Recent Additions

- CF Workers migration — migrated from Vercel to Cloudflare Workers via @opennextjs/cloudflare; all Node.js crypto replaced with Web Crypto API
- Integrated tool suite — Influnx Calc, Growth Report AI, and AI Learn as dashboard tabs
- Music industry quiz — 5 tracks, 60 cards, 65 checklist items, 16 milestones
- QA harness — Vitest + axe-core (8 files), WCAG 2.1 SC 1.3.1 compliance
- Deploy preflight gate — blocks CF Workers deploy on 8 incompatibility checks

---

## Running This

```bash
npm install

npm run dev          # dev server
npm run typecheck    # type check
npm run test         # Vitest + axe-core

# Docker backend (media download engine)
docker compose up -d

# Production build + deploy
npm run build
```

See `.env.example` for required environment variables.

---

## License

MIT — see [LICENSE](LICENSE)
