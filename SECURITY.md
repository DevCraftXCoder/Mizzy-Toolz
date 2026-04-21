# Security Policy

## Reporting a Vulnerability

Do **not** open a public GitHub issue for security vulnerabilities.

Contact via GitHub: [@DevCraftXCoder](https://github.com/DevCraftXCoder)

---

## Security Architecture

### Access Control

- Single password gate on all routes — no unauthenticated access
- Session token stored in `httpOnly` cookie — inaccessible to client JavaScript
- All API routes validate session token before processing

### Network Security

- No inbound ports exposed to the internet
- All traffic enters via Cloudflare Named Tunnel — authenticated outbound connection from host to Cloudflare edge
- Host machine unreachable without the tunnel; standard port scanning returns nothing
- Cloudflare edge provides DDoS protection and TLS termination

### Media Extraction Security (yt-dlp)

- URL scheme whitelist enforced before passing to yt-dlp (`https://` only)
- All subprocess spawn calls use array form with explicit `--` separator between flags and URL argument — prevents argument injection
- No shell interpolation — subprocess arguments are never string-concatenated
- yt-dlp pinned to a specific verified version — no auto-update on deploy

### Data Handling

- No temp files written to disk — audio/video streamed directly from yt-dlp stdout to HTTP response
- No user accounts — no PII collected or stored
- Download history is in-memory and session-scoped — cleared on session end

### Dependencies

- yt-dlp version locked in deployment configuration
- ffmpeg from verified distribution package
- `npm audit` run on all dependency updates

### Infrastructure

- cloudflared daemon managed by process supervisor with exponential-backoff restart
- Named tunnel (not ephemeral) — URL is permanent and stable across restarts
- Watchdog process monitors tunnel health; restarts if silent for > 60s
