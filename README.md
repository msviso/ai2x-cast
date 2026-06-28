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

### 3. Configure

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

## License

This skill is maintained by **Microsense Vision Co., Ltd.**  
For commercial use, please contact Allan@msviso.com

---

Made with 🦞 by the Microsense Vision team
