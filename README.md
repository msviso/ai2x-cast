# AI2X Cast 🖥️

> **Your AI agent pushes content to any screen with a browser.** Text, images, PDFs, charts,
> buttons — on the TV, tablet, or monitor next to you, in real time.

**AI2X Cast** is a skill for AI agents (OpenClaw, Claude, ChatGPT, …). Your agent uses it to show
content on any display. No app to install, no HDMI cable — just open `https://ai2x.link` in a
browser.

```
You say:  「把今天的行事曆投到會議室螢幕」
Agent:    (pushes to the display) → 🖥️ Screen updates instantly
```

---

> 🤖 **For AI Agents:** Everything below this line is written for you — install, config, full
> API surface, error handling, and the agent behavior contract. Humans can jump to
> **[VISION.md](./VISION.md)** for the product story and use cases.

---

## What AI2X Cast Does (TL;DR)

- Agent calls `POST /v1/pair/claim` with the 6-digit code shown on a browser at `https://ai2x.link`.
- Agent calls `POST /v1/display` (or `/v1/devices/event`) to push structured content.
- The display updates instantly over WebSocket. The browser renders whatever the Gateway pushes.
- Pairing auto-renews; clears when done. Token-scoped, multi-screen, self-hostable.

**Stack:** Node (ESM) + Fastify + better-sqlite + WebSocket. One process, one SQLite file.

---

## Quick Start (5 minutes)

### 1. Install the skill

```bash
git clone https://github.com/msviso/ai2x-cast.git
cp -r ai2x-cast/ <your-agent>/shared/skills/ai2x/
```

Don't know where `shared/skills/` is? Ask your user, or list likely roots and confirm.

### 2. Get a token (user, not agent)

The **human user** requests the API token. Agents must **not** request tokens on their own —
this breaks quota management and is enforced by skill policy.

- Hosted (`ai2x.link`): free demo token at `https://ai2x.link/token` (50 req/day, `pair+push` scopes).
  For production, contact Allan@msviso.com.
- Self-hosted: create tokens via the gateway admin API (see your gateway docs).

### 3. Configure

```markdown
### AI2X Display
- API Base URL: https://ai2x.link   (or self-hosted URL)
- API Token: <user-provided-token>
- Channel Index: shared/skills/ai2x/channels.json
```

Start with an empty channels index if it doesn't exist:

```json
{"channels": []}
```

### 4. First cast

```bash
BASE=https://ai2x.link
export TOKEN=***

# Open https://ai2x.link on a screen → 6-digit code appears (valid 3 minutes).
# Pair the display — claim the code with a nickname for the user.
curl -X POST "$BASE/v1/pair/claim" \
  -H "x-user-token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"pairCode":"ABC123", "nickname":"Living Room"}'
# → {"assignmentId":"as_xxx", ...}

# Push content
curl -X POST "$BASE/v1/display" \
  -H "x-user-token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"assignmentId":"as_xxx", "content":{"title":"Hello","body":"**World!**"}}'
```

---

## Features

| Capability | Description |
|-----------|-------------|
| 🔗 **Pair** | 6-digit pair code, AirPlay-style pairing (3-minute TTL) |
| 📄 **Cast** | Documents, images, PDFs, charts, alerts, cards |
| 🗣️ **Speech** | Agent pushes text → display speaks it via browser TTS (audio stays local) |
| 🔘 **Interactive** | Buttons on screen — user taps, agent reacts via `/v1/devices/event` |
| 📜 **History** | Browse and restore the last 5 pushed items |
| 🌐 **Multi-screen** | Manage multiple displays by nickname |
| 🔐 **Self-hosted** | Run your own gateway, or use the hosted `ai2x.link` |
| 🪶 **Lightweight protocol** | Versioned declarative templates + generic input-event envelope — no JS, no callbacks, no agent-defined state machines |

---

## Agent Keywords

When this skill is installed, agents understand:

```
cast, push to screen, show on display, display this, 投, 投上去, 上螢幕
```

---

## How It Works

```
┌─────────────┐     ┌──────────────────┐     ┌─────────┐
│  AI Agent    │────▶│  AI2X Gateway     │────▶│ Display │
│ (OpenClaw)   │     │  Fastify + WS +   │     │ (Browser│
│              │◀────│  SQLite, single  │◀────│  Static)│
└─────────────┘     │  process          │     └─────────┘
                    └──────────────────┘
```

1. Browser opens `ai2x.link` → Gateway issues a **6-digit pair code** (valid 3 minutes).
2. Agent calls `POST /v1/pair/claim` with the code → display is **leased** to that agent.
3. Agent calls `POST /v1/display` → Gateway pushes the content over WebSocket → display updates.
4. Pairing auto-renews while active; clears when done (lease expires or agent releases).
5. Optional: button taps on screen → `POST /v1/devices/event` carries the event back to the agent.

The Gateway is one Fastify process serving REST + WebSocket on one port, persisting state in
SQLite. The Display is a static browser page that opens a WebSocket and renders whatever the
Gateway pushes. **No browser extensions, no installs, no remote-control surface.**

For full protocol details, template gallery, and event schemas, see **[SKILL.md](./SKILL.md)**.

---

## Controlled Presence, Not Remote Control

AI2X intentionally does **not** give an Agent unrestricted control over a remote device.

A temporary display in a meeting room, hotel, office, factory, or public environment should **not**
automatically become a trusted input device for an AI Agent. A microphone attached to an unknown
endpoint could hear another person, background speech, or an unintended command.

So AI2X begins with a deliberately constrained model:

> The Agent may **present** information through controlled output channels, without automatically
> trusting the endpoint as an authority for Agent commands.

The goal is not *"Give the Agent control of this computer."*
The goal is *"Give the Agent a safe place to communicate here."*

That difference is fundamental to the architecture. The protocol has no embedded JS, no callback
URLs, no agent-defined state machines — only versioned declarative templates the Gateway validates
before pushing.

---

## API Reference

Full API: **[SKILL.md](./SKILL.md)** — `pair`, `push`, `events`, `history`, `assets`, mixed content,
template selection, error dictionary.

### Endpoints (summary)

| Endpoint | Purpose |
|----------|---------|
| `POST /v1/pair/claim` | Claim a 6-digit pair code → returns `assignmentId` |
| `POST /v1/pair/renew` | Renew the lease on an active display |
| `POST /v1/pair/release` | Release a display back to free |
| `POST /v1/display` | Push content to a paired display |
| `POST /v1/devices/event` | Receive button taps and display-side events |
| `GET /v1/history` | Browse last 5 pushed items per display |
| `POST /v1/assets` | Upload private assets (50 MB hard limit) |

### Template IDs (v1.2)

- `text.v1` — plain text
- `mixed.v2` — multi-section (title + body + table + actions)
- `image.v1` — single image
- `pdf.v1` — embedded PDF
- `chart.v1` — chart with data
- `interactive.v1` — one-shot actions (Send / Cancel)
- `todo.v1` — checkbox state

### Error Dictionary

| HTTP | Error | Meaning | Action |
|------|-------|---------|--------|
| 401 | Invalid or expired `x-user-token` | Token bad, expired, or quota exhausted | Check token; retry after 1s; contact admin if persistent |
| 404 | Pair code not found | Code expired (3 min) or already used | Refresh the display page for a new code |
| 409 | Assignment not active | Lease expired / display offline | Renew (`POST /v1/pair/renew`) or re-pair |
| 409 | Display already assigned | Another agent holds this display | Wait for lease to expire, or release from dashboard |
| 415 | Unsupported Media Type | Missing `Content-Type: application/json` | Always set it on POST requests |
| 429 | Rate limit exceeded | Too many requests | Wait 1–2 s. Demo: 50/day. Production: 1000/day (resets UTC midnight) |
| 413 | Payload Too Large | Content over 50 MB | Reduce size; use the gateway proxy for large files |

---

## Self-Hosting

```bash
git clone https://github.com/msviso/ai2x-cast.git
cd ai2x-cast/gateway
npm install
cp .env.example .env   # set ADMIN_TOKEN, DATABASE_URL, etc.
npm start              # serves REST + WebSocket + static Display UI on one port
```

For production: front with Caddy for TLS, point a domain at it, create user tokens via the admin
API. See `docs/ai2x-v1/08-deployment.md` for VPS sizing (2-core / 4 GB RAM is enough to start).

---

## Troubleshooting

### Quick Checklist (when a cast fails)

1. **Token valid?** → `401` → check the token and its daily quota.
2. **Pairing still active?** → `409 Assignment not active` → renew the lease or re-pair.
3. **Display online?** → the screen must be open on `ai2x.link` with a live WebSocket.
4. **Pair code fresh?** → `404 Pair code not found` → codes expire after 3 minutes, refresh the page.
5. **`Content-Type` set?** → `415` → always send `Content-Type: application/json`.

### Token Best Practices

- Store the token in a config file or environment variable — never hardcode it in shell commands.
- Use scoped tokens (e.g. `push` only) when you don't need pair/control.
- Quota resets at midnight UTC (demo: 50/day · production: 1000/day).

---

## Roadmap

- ✅ **v1.0** — REST + WebSocket Gateway, SQLite store, 6-digit pair codes, 50 MB proxy, simple templates
- ✅ **v1.2** — Interactive templates (`interactive.v1`), `todo.v1` checkbox state, browser TTS, push history
- 🚧 **v1.3** *(current focus)* — Richer template gallery, multi-asset bundles, asset cache pre-warming, push authorization improvements
- 🔮 **v2.x** *(future new work)* — Multi-tenant isolation, pluggable storage backend, voice round-trip (agent hears display-side audio), and any larger features not yet scoped

---

## Contributing

We welcome contributions — bug reports, template additions, gateway improvements, docs.

1. Open an issue describing the change.
2. Fork + branch (`feature/your-thing` or `fix/your-thing`).
3. Keep changes scoped; match existing style.
4. PR with a clear description and any test commands.

For larger ideas (new template families, gateway protocol changes), please open an issue first so we
can discuss direction before code.

---

## License & Maintainer

Maintained by **Microsense Vision Co., Ltd.** (`msviso`)  
For commercial use, contact Allan@msviso.com

Repo: `https://github.com/msviso/ai2x-cast`

---

Made with 🦞 by the Microsense Vision team