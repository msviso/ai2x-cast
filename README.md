# AI2X Cast 🖥️

> **Agent-to-Screen Transport for OpenClaw.**  
> Let your AI agent push content to any display with a simple `cast` command.

AI2X Cast bridges AI agents and physical displays. No CMS, no templates, no content management — just a lightweight transport layer that lets your agent control what appears on screen, in real time.

```
User: 「Push today's calendar to the meeting room screen」
Agent: (POST /v1/display) → 🖥️ Display updates instantly
```

## Features

| Capability | Description |
|-----------|-------------|
| 🔗 **Pair** | 6-digit pair code + QR code, AirPlay-style pairing |
| 📄 **Cast** | Push documents, images, PDFs, charts, alerts, cards |
| 🔘 **Interactive** | Add buttons on screen — users tap, agent reacts |
| 📜 **History** | Browse and restore last 5 pushed items |
| 🔄 **Auto-renew** | Lease-based session with auto-extend |
| 🌐 **Multi-screen** | Manage multiple displays by nickname |
| 🔐 **Self-hosted** | You own the infrastructure (or use ai2x.link) |

## Quick Start

### 1. Install Skill

Copy `ai2x-cast/` into your OpenClaw agent's shared skills:

```bash
cp -r ai2x-cast/ /path/to/your/agent/shared/skills/ai2x/
```

### 2. Get an API Token

- **Self-hosted:** Set up the AI2X Gateway on your server, create tokens via the admin API
- **ai2x.link:** Contact your provider for a scoped API token

### 3. Get a Token

- **Self-hosted:** Your agent generates tokens from the admin API (see `SKILL.md`)
- **ai2x.link:** Email **Allan@msviso.com** for a temporary demo token

### 4. Configure

Set your token in the agent's environment:

```markdown
### AI2X Display
- API Base URL: https://ai2x.link (or your self-hosted URL)
- API Token: ak_xxx
- Channel Index: shared/skills/ai2x/channels.json
```

### 4. Cast!

```bash
# Pair a display (read the 6-digit code from ai2x.link on your screen)
curl -X POST "$BASE/v1/pair/claim" \
  -H "x-user-token: $TOKEN" \
  -d '{"pairCode":"ABC123", "nickname":"Living Room"}'

# Push content
curl -X POST "$BASE/v1/display" \
  -H "x-user-token: $TOKEN" \
  -d '{"assignmentId":"as_xxx", "content":{"title":"Hello","body":"**World!**"}}'
```

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────┐
│  AI Agent    │────▶│  AI2X Gateway │────▶│ Display │
│ (OpenClaw)   │     │  (WebSocket)  │     │ (Browser)│
└─────────────┘     └──────────────┘     └─────────┘
     │                      │
     │  POST /v1/display    │  WS push + render
     │  POST /v1/pair/*     │  Button callbacks
     └──────────────────────┘
```

1. Open `ai2x.link` on any browser (TV, tablet, monitor)
2. A 6-digit pair code appears
3. Your agent calls `POST /v1/pair/claim` with the code
4. The display is now "leased" to your agent
5. Any `POST /v1/display` pushes content instantly

## Agent Keywords

When this skill is installed, agents understand these commands:

```
cast, push to screen, show on display, display this
```

## API Reference

See [SKILL.md](./SKILL.md) for full API endpoint reference.

## Requirements

- An AI2X Gateway instance (self-hosted or ai2x.link)
- An API token with appropriate scopes

## Troubleshooting & Error Handling

### Agent Error Dictionary

When your AI agent encounters an error from the AI2X API, here's how to interpret and resolve it:

| HTTP | Error Response | Meaning | Agent Action |
|------|---------------|---------|--------------|
| 401 | `{"error":"Invalid or expired x-user-token"}` | Token invalid, expired, or daily quota (1000) exhausted | Check token validity. Retry after 1s on quota. Contact admin if persistent. |
| 404 | `{"error":"Pair code not found"}` | Pair code expired (TTL: 180s) or already claimed | Ask user to refresh the display page (ai2x.link) for a new 6-digit code. |
| 409 | `{"error":"Assignment not active"}` | Lease expired or display disconnected | Renew lease (`POST /v1/pair/renew`) or re-pair the display. |
| 409 | `{"error":"Display already assigned"}` | Another agent/claim on this display | Wait for lease to expire or ask user to release from dashboard. |
| 415 | `{"error":"Unsupported Media Type"}` | Missing `Content-Type: application/json` header | Always include `Content-Type: application/json` on POST requests. |
| 429 | `{"error":"Rate limit exceeded"}` | Too many requests per second | Wait 1-2 seconds before retrying. Token limit: 1000/day (resets UTC midnight). |
| 413 | `{"error":"Payload Too Large"}` | Content exceeds 50MB | Reduce size (especially PDF/images). Use Gateway proxy for large files. |

### Quick Agent Checklist

When your agent can't push content, check these in order:

1. **Is the token valid?** → `401` → check API key, quota remaining
2. **Is the assignment active?** → `409 Assignment not active` → lease expired, renew or re-pair
3. **Is the display online?** → display must be on `ai2x.link` with active WebSocket
4. **Is the pair code still showing?** → `404 Pair code not found` → codes expire after 180s, refresh page
5. **Is Content-Type set?** → `415` → always include `Content-Type: application/json`

### Token Best Practices

- Store the token in a config file or environment variable, never hardcode it in shell commands
- Use scoped tokens (e.g., `push` only if you don't need pair/control)
- Token quota resets at midnight UTC (1000 requests/day default)
- If rate-limited (`429`), wait 1-2 seconds before retrying

### Agent Instruction: Do NOT Self-Request Tokens

AI agents must **never** use the self-service token portal or `POST /v1/token/request` on their own.
Demo tokens are for human evaluation only. Agents should use the token already configured
in their environment by an administrator. Self-requesting tokens causes:
- Duplicate/multiple token state with no central management
- Rapid quota exhaustion (50/day per token)
- Confusion about which assignment belongs to which token

### Diagnose Endpoint (Coming Soon)

A `GET /v1/diagnose` endpoint is in development that will return a complete health snapshot:
- Token validity, label, scopes, usage today vs quota
- All assignments with active/expired status and remaining lease time
- Display online/offline status and pairing state

This lets your agent self-diagnose with one API call instead of guessing which error code applies.

---

## License

This skill is maintained by **Microsense Vision Co., Ltd.**  
For commercial use, please contact Allan@msviso.com

---

Made with 🦞 by the Microsense Vision team
