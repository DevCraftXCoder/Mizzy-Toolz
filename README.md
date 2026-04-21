# Mizzy Tools

**Private media download dashboard. Paste a link, get MP3 or MP4 — straight to your browser.**

> A password-protected, self-hosted media download tool. Supports YouTube, SoundCloud, Instagram, TikTok, and Twitter/X. Built with a custom yt-dlp streaming backend delivered through a permanent Cloudflare Named Tunnel.

---

## Architecture

```
Browser (authenticated session)
  │
  ▼
Next.js 15  (App Router · SSR · password-gated)
  │
  ▼  Cloudflare Named Tunnel (permanent URL — no port exposure)
yt-dlp Backend  (Node.js streaming proxy)
  │
  ├── yt-dlp  (media extraction engine)
  ├── ffmpeg  (audio transcoding, MP3 conversion)
  └── Cloudflare Tunnel (cloudflared daemon — inbound only, 127.0.0.1 binding)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, App Router, React |
| Backend | Node.js streaming proxy |
| Media Engine | yt-dlp (pinned version) |
| Transcoding | ffmpeg |
| Tunnel | Cloudflare Named Tunnel (cloudflared) |
| Auth | Password gate (session-based) |

---

## Features

- **Multi-platform** — YouTube, SoundCloud, Instagram, TikTok, Twitter/X
- **Format selection** — MP3 (audio-only) or MP4 (video+audio)
- **Streaming download** — no temp files on server; streams directly to client
- **Download history** — session-scoped history of recent downloads
- **Clean UI** — paste, click, done

---

## Security Design

### Access Control
- Password-gated at the application layer — no unauthenticated access to any endpoint
- Session token stored in `httpOnly` cookie; not accessible to client JavaScript
- All traffic routed through Cloudflare Named Tunnel — no ports exposed to the public internet

### Infrastructure
- Backend bound to `127.0.0.1` only; externally unreachable without the tunnel
- Cloudflare Tunnel acts as the sole ingress point — DDoS protection + TLS termination at the edge
- `cloudflared` daemon managed by process supervisor with automatic restart on failure

### Media Extraction Security
- yt-dlp pinned to a verified version — no auto-update on deploy
- All spawn calls use array form with `--` separator before URL (prevents argument injection)
- URL validation before passing to yt-dlp
- No user-controlled flags or options passed to the subprocess

### Dependency Hardening
- yt-dlp pinned: version locked in Dockerfile
- No `npm audit` vulnerabilities in production dependencies
- ffmpeg from verified distribution package

---

## Deployment

Runs as a containerized service managed by a process supervisor (PM2). The Cloudflare Named Tunnel (`cloudflared`) runs as a companion process with:

- Exponential backoff on reconnect failures
- Watchdog process that restarts the tunnel if it goes silent
- Named tunnel (not ephemeral) — URL is permanent and does not change between restarts

---

## Why Cloudflare Named Tunnels over Port Forwarding

| Approach | Public IP exposure | TLS | DDoS protection | URL stability |
|---|---|---|---|---|
| Port forwarding | Yes | Manual | No | IP-dependent |
| Cloudflare Named Tunnel | No | Automatic | Yes | Permanent |

Named tunnels assign a permanent `*.cfargotunnel.com` subdomain (or custom domain) that routes through Cloudflare's network to your local process via an outbound-only connection. The host machine never exposes a public port.

---

## License

MIT — see [LICENSE](LICENSE)
