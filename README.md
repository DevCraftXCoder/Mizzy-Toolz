# Mizzy Tools

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=flat&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=flat&logo=cloudflare&logoColor=white)
![AI Powered](https://img.shields.io/badge/AI_Powered-D97706?style=flat&logo=anthropic&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

**Private all-in-one creator dashboard. Media downloads, creator analytics, influencer scoring, and an AI music industry learning suite.**

> Password-protected, self-hosted multi-tool dashboard for independent creators. A streaming media downloader, growth analytics engine, influencer scoring system, and AI-powered music industry flashcard quiz — all served through a permanent Cloudflare Named Tunnel with zero port exposure.

---

## 10 Integrated Tools

**Dashboard tabs:**

1. **Media Download** — Streaming downloader for YouTube, SoundCloud, Instagram, TikTok, Twitter/X
2. **Influnx Calc** — 100-point influencer scoring engine (6 categories, real-time)
3. **Growth Report** — AI-powered analytics narrative (Spotify, YouTube, Apple Music, TikTok, Instagram)
4. **AI Learn** — Music industry flashcard quiz (60 cards, 5 tracks, 16 badges)
5. **EV Betta** — Real-time sports picks board with EV calculations
6. **Finos** — Multi-platform AI finance workspace (web + desktop + API)
7. **Biggest Bro** — AI co-pilot for independent creators
8. **Co-Writer** — AI lyric and song writing assistant
9. **Multi-Agent** — AI agent orchestration and task automation
10. **Underground** — Social music platform feed integration + artist profiles

**Supporting panels:** History, Settings, Prompt Library, PDF Report Engine, API Status, Audit.

---

## Architecture

```
Browser
  │
  ▼
Next.js 15  (App Router · Cloudflare Workers)
  │
  ├── Download      ── Media backend ──▶ yt-dlp + ffmpeg (streaming, zero temp files)
  ├── Analytics     ── Growth Report SSE ──▶ LLM (streaming narratives)
  ├── AI Tools      ── Biggest Bro, Co-Writer, Learn ──▶ LLM APIs
  ├── Platforms     ── EV Betta, Finos, Underground ──▶ External APIs (integrated)
  └── Calc          ── Influnx Calc (client-side, zero network calls)
```

All external traffic routed through Cloudflare. No direct port exposure. Download backend accessible only through Cloudflare Named Tunnel.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Next.js 15 (App Router) | Cloudflare Workers via @opennextjs/cloudflare |
| Media Engine | yt-dlp (pinned version) + ffmpeg | Docker container, streamed output |
| Tunnel | Cloudflare Named Tunnel (cloudflared) | Permanent public URL, zero port exposure |
| AI | AI SDK (LLM) | SSE streaming, prompt caching |
| Auth | Web Crypto API (PBKDF2) | httpOnly cookie session — no JWT library |
| Testing | Vitest + axe-core | 8 test files, WCAG 2.1 AA compliance |

---

## Tools Breakdown

### 1. Media Download
Streaming media downloader for YouTube, SoundCloud, Instagram, TikTok, Twitter/X.

- **Zero temp files:** yt-dlp output piped directly to HTTP response — no disk buffering
- **Auto format:** Detects optimal format (MP3 for audio, MP4 for video)
- **DASH m4a priority:** Cleanest MP3 conversion from YouTube DASH streams
- **Session history:** Download log with replay links

### 2. Influnx Calc
100-point influencer scoring — client-side TypeScript, no data leaves the browser.

- 6 metric categories, configurable weights
- Per-platform engagement normalization
- Bot/bought-follower anomaly detection
- Sub-100ms evaluation

### 3. Growth Report AI
AI-generated analytics narrative from real platform data.

- Aggregates Spotify, YouTube, Apple Music, TikTok, Instagram metrics
- Week / month / quarter period comparisons
- Streaming token-by-token via SSE
- Prompt caching (5-min TTL) — ~80% cost savings on repeat queries
- CSV export, delta indicators

### 4. AI Learn — Music Industry Quiz
Flashcard quiz with spaced repetition and progress tracking.

- 5 learning tracks, 60 flashcards, 65 checklist items, 16 badges
- Topics: YouTube algorithm, streaming royalties, sync licensing, distribution, publishing splits
- Progress persists across sessions

### 5. EV Betta — Sports Picks
Real-time sports picks board with expected-value (EV) rankings.

- Aggregates odds from 6+ sources daily
- EV calculation engine (decimal odds × hit rate − 1)
- Tier-based ranking: LOCKED, STRONG, WATCH
- Per-sport filters, ET timezone-aware date navigation

### 6. Finos — AI Finance Workspace
Multi-platform workspace for personal finance + AI intelligence.

- **Platforms:** Next.js web, Tauri desktop, Hono edge API — identical UX across all three
- **AI features:** Forecast, health-score, risk-scan, tax analytics
- **Real-time sync:** Supabase auth + live state updates
- **Design:** Neon cyan glassmorphism, live-drifting demo data

### 7. Biggest Bro — AI Co-Pilot
Domain-expert AI assistant for independent creators (YouTubers, musicians).

- LLM with extended thinking (8k token budget)
- Tool use: content calendar, trend analysis, analytics summaries
- Conversation history + system prompt caching
- Fallback graceful shutdown on budget overflow

### 8. Co-Writer — AI Song Assistant
AI lyric and song writing partner.

- Chord progressions + song structure guidance
- Hook generation, rhyme scheme analysis
- Genre-specific tone matching
- Collaborative revision workflow

### 9. Multi-Agent — Agent Orchestration
AI agent orchestrator for complex task decomposition.

- Automatic agent selection (Opus/Sonnet tier-aware)
- Parallel execution + result synthesis
- Handoff protocol + session context management
- Real-time progress streaming

### 10. Underground — Social Music Feed
Direct integration with Underground, the social music platform.

- Artist profiles + track feed browsing
- Follow / like / bookmark from Mizzy UI
- Real-time notifications + activity feed
- Subscription tier visibility (Underground+ gating)

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

## Recent Additions (30 days)

- **feat(underground):** Direct Underground social platform feed integration + profile browsing
- **feat(multi-agent):** AI agent orchestration panel with task decomposition and parallel execution
- **feat(co-writer):** AI lyric and song writing assistant with genre-aware tone matching
- **feat(biggest-bro):** AI creator co-pilot with extended thinking (8k token budget)
- **feat(finos):** Multi-platform AI finance workspace (web + desktop + edge API parity)
- **feat(ev-betta):** Real-time sports picks board with EV ranking engine
- **fix(frontend):** Restore mizzy tab access gates + DFE security compliance
- **docs:** Replace npm with pnpm for supply-chain security, refresh architecture diagrams
- **chore:** Pin all dependencies to exact lockfile versions; add audit:ci script
- **feat(streaming):** AI insights streaming for growth reports + quiz engine via SSE

---

## License

MIT — see [LICENSE](LICENSE)

---

*Built by Frxncois — not open source.*
