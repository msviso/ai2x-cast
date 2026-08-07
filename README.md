# AI2X Cast 🖥️

> **Your AI agent pushes content to any screen with a browser.**
> Text, images, PDFs, charts, buttons — on the TV, tablet, or monitor next to you, in real time.

**AI2X Cast is a skill for AI agents** (OpenClaw, Claude, ChatGPT, …). Your agent uses it to show
content on any display. No app to install, no HDMI cable — just open a URL in a browser.

```
You say:  「把今天的行事曆投到會議室螢幕」
Agent:    (pushes to the display) → 🖥️ Screen updates instantly
```

**What you need to start:**

- An AI agent (OpenClaw or similar)
- A screen with a browser open at `https://ai2x.link`
- An API token (free demo token available — see below)

**It works in about 5 minutes.** No server setup required if you use `ai2x.link`.

---

## Why AI2X Cast?

> *AI should not be trapped inside a chat window.*

Your AI agent can already *think, talk, search, and create*. But when it wants to *show* you
something — a chart, a presentation, a status update — it has nowhere to put it. The TV in the
meeting room, the tablet in the kitchen, the monitor in the workshop — they just sit there.

**AI2X Cast is the missing last meter.** It turns any screen with a browser into an AI output device:

- No app to install on the display, no HDMI, no dedicated hardware.
- Any browser — smart TV, tablet, Raspberry Pi, old monitor — becomes a live screen for your agent.
- Think of it as **AirPlay for AI agents**: your agent pushes, the screen shows, in real time.

### Earbuds for the conversation. A display for the moment that matters.

Your AI agent can already talk to you anywhere — through earbuds or a smart-glasses mic, hands-free.
Great for quick answers on the move.

But some moments need more than a voice in your ear: a chart to explain, a presentation to give, a
dashboard to review together. That's when a screen matters — and AI2X Cast is how your agent takes
over that screen: it pushes the content, updates it live, and lets people on the other side interact
(tap, confirm, navigate).

### Where you'd use it

| Scenario | What the agent pushes |
|----------|----------------------|
| **Meeting room** | Agenda, charts, slides — spoken summary on the TV |
| **Factory / workshop** | Work orders, SOPs, live alerts on the shop-floor monitor |
| **Home** | Recipe on the kitchen tablet, family calendar on the living-room TV |
| **Retail / signage** | Promotions, queue numbers — with interactive buttons on screen |
| **Front desk** | Waiting screens, call numbers, interactive Q&A |

And because displays are **two-way** (buttons on screen → your agent reacts), it works for things
like confirmations, ordering, and surveys — not just one-way casting.

---

## Quick Start (5 minutes)

1. **Open `https://ai2x.link`** on any screen (TV, tablet, monitor) → a **6-digit pair code** appears.
2. **Get a demo token** at `https://ai2x.link/token` (50 requests/day, `pair+push` scopes).
3. **Give your agent this one-prompt** (copy-paste):

> I'd like to install the AI2X Cast skill so you can push content to my displays.
> The skill is at https://github.com/msviso/ai2x-cast — please install it,
> then tell me when it's ready.

After it installs, paste this:

> Now check that it works and explain how I get an API token
> (do not request one yourself). I want to start testing soon.

The agent will install the skill, check compatibility, and guide you through asking for a token.

> ⚠️ **Your agent must NOT request tokens on its own.** Agents use the token already configured by
> an administrator. Self-requesting tokens breaks token management and exhausts quotas — this applies
> to demo tokens too.

### Manual install

```bash
git clone https://github.com/msviso/ai2x-cast.git
cp -r ai2x-cast/ <your-agent>/shared/skills/ai2x/
```

Don't know where your agent's `shared/skills/` folder is? Ask your agent: *"Where is your shared/skills folder?"*

### Configure

```markdown
### AI2X Display
- API Base URL: https://ai2x.link (or your self-hosted URL)
- API Token: <your token>
- Channel Index: shared/skills/ai2x/channels.json
```

Start with an empty channels index:

```json
{"channels": []}
```

### First cast

```bash
BASE=https://ai2x.link
export TOKEN=<your-token>

# Pair a display — use the 6-digit code shown on your screen
# (the code expires after 3 minutes — refresh the page for a new one)
curl -X POST "$BASE/v1/pair/claim" \
  -H "x-user-token: $TOKEN" \
  -d '{"pairCode":"ABC123", "nickname":"Living Room"}'

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
| 🗣️ **Speech** | Agent pushes text → the display speaks it (browser TTS, audio stays local) |
| 🔘 **Interactive** | Buttons on screen — users tap, agent reacts |
| 📜 **History** | Browse and restore the last 5 pushed items |
| 🌐 **Multi-screen** | Manage multiple displays by nickname |
| 🔐 **Self-hosted** | Run your own gateway, or use the hosted `ai2x.link` |

## Agent Keywords

When this skill is installed, agents understand:

```
cast, push to screen, show on display, display this, 投, 投上去, 上螢幕
```

---

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────┐
│  AI Agent    │────▶│  AI2X Gateway │────▶│ Display │
│ (OpenClaw)   │     │  (WebSocket)  │     │ (Browser│
└─────────────┘     │   Fastify +   │     │  Static │
                    │    SQLite)    │     └─────────┘
                    └──────────────┘
```

1. Open `ai2x.link` in any browser → a **6-digit pair code** appears (valid 3 minutes)
2. Your agent calls `POST /v1/pair/claim` with the code → the display is now **leased** to your agent
3. Any `POST /v1/display` pushes content to that screen instantly (WebSocket fan-out)
4. The pairing auto-renews while active, and clears when done
5. Optional: `POST /v1/devices/event` sends button taps back to your agent

The Gateway is a single Fastify process serving REST + WebSocket on one port, persisting state in
SQLite. The Display is a static browser page that opens a WebSocket and renders whatever the
Gateway pushes. No browser extensions, no installs — just `ai2x.link`.

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

That difference is fundamental to the architecture.

---

## API Reference

Full API: **[SKILL.md](./SKILL.md)** — `pair`, `push`, `events`, `history`, `assets`, mixed content,
template selection, error dictionary.

### Error Dictionary

| HTTP | Error | Meaning | Action |
|------|-------|---------|--------|
| 401 | Invalid or expired x-user-token | Token bad, expired, or quota exhausted | Check token; retry after 1s; contact admin if persistent |
| 404 | Pair code not found | Code expired (3 min) or already used | Refresh the display page for a new code |
| 409 | Assignment not active | Lease expired / display offline | Renew (`POST /v1/pair/renew`) or re-pair |
| 409 | Display already assigned | Another agent holds this display | Wait for lease to expire, or release from dashboard |
| 415 | Unsupported Media Type | Missing `Content-Type: application/json` | Always set it on POST requests |
| 429 | Rate limit exceeded | Too many requests | Wait 1–2 s. Demo: 50/day. Production: 1000/day (resets UTC midnight) |
| 413 | Payload Too Large | Content over 50 MB | Reduce size; use the gateway proxy for large files |

---

## Self-Hosting

Want your own Gateway instead of `ai2x.link`? AI2X is one Fastify process + one SQLite file.

```bash
git clone https://github.com/msviso/ai2x-cast.git
cd ai2x-cast/gateway
npm install
cp ../.env.example .env   # set ADMIN_TOKEN, DATABASE_URL, etc.
npm start                  # serves REST + WebSocket + static Display UI
```

For production: front with Caddy for TLS, point a domain at it, create user tokens via the admin
API. See `docs/ai2x-v1/08-deployment.md` for VPS sizing (2-core / 4 GB RAM is enough to start).

---

## Troubleshooting

### Quick Checklist (when a cast fails)

1. **Token valid?** → `401` → check the token and its daily quota
2. **Pairing still active?** → `409 Assignment not active` → renew the lease or re-pair
3. **Display online?** → the screen must be open on `ai2x.link` with a live connection
4. **Pair code fresh?** → `404 Pair code not found` → codes expire after 3 minutes, refresh the page
5. **`Content-Type` set?** → `415` → always send `Content-Type: application/json`

### Token Best Practices

- Store the token in a config file or environment variable — never hardcode it in shell commands
- Use scoped tokens (e.g. `push` only) when you don't need pair/control
- Quota resets at midnight UTC (demo: 50/day · production: 1000/day)

---

## Roadmap

- ✅ v1.0 — REST + WebSocket Gateway, SQLite store, 6-digit pair codes, 50 MB proxy, simple templates
- ✅ v1.2 — Interactive templates (`interactive.v1`), `todo.v1` checkbox state, browser TTS, push history
- 🚧 v1.3 — Richer template gallery, multi-asset bundles, asset cache pre-warming
- 🔮 v2.x — Multi-tenant isolation, pluggable storage backend, voice round-trip (agent hears display-side audio)

---

## Contributing

We welcome contributions — bug reports, template additions, gateway improvements, docs.

1. Open an issue describing the change
2. Fork + branch (`feature/your-thing` or `fix/your-thing`)
3. Keep changes scoped; match existing style
4. PR with a clear description and any test commands

For larger ideas (new template families, gateway protocol changes), please open an issue first so we
can discuss direction before code.

---

## License & Maintainer

Maintained by **Microsense Vision Co., Ltd.**  
For commercial use, contact Allan@msviso.com

---

Made with 🦞 by the Microsense Vision team