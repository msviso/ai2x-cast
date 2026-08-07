# AI2X Cast 🖥️

> **Your AI agent pushes content to any screen with a browser.**  
> Text, images, PDFs, charts, buttons — on the TV, tablet, or monitor next to you, in real time.

AI2X Cast is a skill for AI agents (OpenClaw, Claude, ChatGPT, …). Your agent uses it to show
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

## Why AI2X Cast?

Screens are everywhere — but getting content *onto* them is still painful.

Think about a typical day:

- Your calendar, dashboards, and documents live on your laptop or phone.
- The TV in the meeting room, the tablet in the kitchen, the monitor in the workshop — they just sit there.
- Showing something on them means **cables, casting apps, file transfers**, or building a custom app.

Meanwhile, your AI agent can already *generate* anything — summaries, charts, PDFs, slides, alerts. But it has no way to *show* it on the screen in front of you. That last meter is missing.

**AI2X Cast is that last meter.** It turns any screen with a browser into an AI output device:

- No app to install on the display, no HDMI, no dedicated hardware.
- Any browser — smart TV, tablet, Raspberry Pi, old monitor — becomes a live screen for your agent.
- Think of it as **AirPlay for AI agents**: your agent pushes, the screen shows, in real time.

### What it feels like

```
Morning, 5 minutes before the meeting.

You:   「把今天的議程和上週的銷售圖表投到會議室螢幕」
Agent: (pairs the meeting-room display, pushes agenda + chart) → 🖥️ screen updates instantly

You:   「第三頁那張圖，幫我標出下滑的區域」
Agent: (redraws the chart, pushes the updated version) → 🖥️ updates again
```

No cables. No searching for files. No "can everyone see my screen?"

### Where you'd use it

| Scenario | What the agent pushes |
|----------|----------------------|
| **Meeting room** | Agenda, charts, slides — spoken summary on the TV |
| **Factory / workshop** | Work orders, SOPs, live alerts on the shop-floor monitor |
| **Home** | Recipe on the kitchen tablet, family calendar on the living-room TV |
| **Retail / signage** | Promotions, queue numbers — with interactive buttons on screen |
| **Front desk** | Waiting screens, call numbers, interactive Q&A |

And because displays are **two-way** (buttons on screen → your agent reacts), it works for things like confirmations, ordering, and surveys — not just one-way casting.

### From earbuds to the big screen

Your AI agent can already talk to you anywhere — through earbuds or a smart-glasses mic (STT/TTS), hands-free, no screen required. Great for quick answers on the move.

But some moments need more than a voice in your ear: a chart to explain, a presentation to give, a complex dashboard to review together. That's when a screen matters — and AI2X Cast is how your agent takes over that screen: it pushes the content, updates it live, and lets people on the other side interact (tap, confirm, navigate).

**Earbuds for the conversation. A display for the moment that matters.**

## Try It Now (5 minutes)

1. Open `https://ai2x.link` on any screen (TV, tablet, monitor) → a **6-digit pair code** appears
2. Get a free **demo token** at `https://ai2x.link/token` (human only — see Token section)
3. Give your agent the one-prompt below → it installs the skill and guides you through pairing

## Install with One Prompt

Copy this message and paste it to your agent:

> I'd like to install the AI2X Cast skill so you can push content to my displays.
> The skill is at https://github.com/msviso/ai2x-cast — please install it,
> then tell me when it's ready.

After it installs, paste this second message:

> Now check that it works and explain how I get an API token
> (do not request one yourself). I want to start testing soon.

（中文版：直接把上面兩段訊息複製貼給你的 agent 即可）

The agent will install the skill, check compatibility and availability, and guide you through
asking for a token. **Tokens are issued to you, the user — your agent must not request tokens on its own.**

Or install manually with the steps below.

## Quick Start

### 1. Install the Skill

Get the skill and copy it into your agent's skills folder:

```bash
git clone https://github.com/msviso/ai2x-cast.git
cp -r ai2x-cast/ <your-agent>/shared/skills/ai2x/
```

Don't know where your agent's `shared/skills/` folder is? Just ask your agent:
*"Where is your shared/skills folder?"* — it will tell you.

### 2. Get an API Token (you, the human — not your agent)

- **ai2x.link (hosted):** get a free demo token instantly at **https://ai2x.link/token**
  (demo tokens: 50 requests/day, `pair+push` scopes). For production use, email Allan@msviso.com.
- **Self-hosted:** create tokens on your own gateway via its admin API (see your gateway docs).

> ⚠️ **Your agent must NOT request tokens on its own.** Agents use the token already
> configured by an administrator. Self-requesting tokens breaks token management and
> exhausts quotas. This applies to demo tokens too.

### 3. Configure

Set the token in your agent's environment:

```markdown
### AI2X Display
- API Base URL: https://ai2x.link (or your self-hosted URL)
- API Token: <your token>
- Channel Index: shared/skills/ai2x/channels.json
```

Make sure `channels.json` exists (start with an empty one):

```json
{"channels": []}
```

### 4. Cast!

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

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────┐
│  AI Agent    │────▶│  AI2X Gateway │────▶│ Display │
│ (OpenClaw)   │     │  (WebSocket)  │     │ (Browser)│
└─────────────┘     └──────────────┘     └─────────┘
```

1. Open `ai2x.link` in any browser → a **6-digit pair code** appears (valid 3 minutes)
2. Your agent calls `POST /v1/pair/claim` with the code → the display is now paired ("leased") to your agent
3. Any `POST /v1/display` pushes content to that screen instantly
4. The pairing auto-renews while active, and clears when done

## Features

| Capability | Description |
|-----------|-------------|
| 🔗 **Pair** | 6-digit pair code, AirPlay-style pairing |
| 📄 **Cast** | Documents, images, PDFs, charts, alerts, cards |
| 🗣️ **Speech** | Agent pushes text → the display speaks it (browser TTS) |
| 🔘 **Interactive** | Buttons on screen — users tap, agent reacts |
| 📜 **History** | Browse and restore the last 5 pushed items |
| 🌐 **Multi-screen** | Manage multiple displays by nickname |
| 🔐 **Self-hosted** | Run your own gateway, or use ai2x.link |

## Agent Keywords

When this skill is installed, agents understand:

```
cast, push to screen, show on display, display this, 投, 投上去, 上螢幕
```

## API Reference

See [SKILL.md](./SKILL.md) for the full API reference (pair, push, events, history, assets, mixed content).

## Troubleshooting

### Quick Checklist (when a cast fails)

1. **Token valid?** → `401` → check the token and its daily quota
2. **Pairing still active?** → `409 Assignment not active` → renew the lease or re-pair
3. **Display online?** → the screen must be open on `ai2x.link` with a live connection
4. **Pair code fresh?** → `404 Pair code not found` → codes expire after 3 minutes, refresh the page
5. **`Content-Type` set?** → `415` → always send `Content-Type: application/json`

### Error Dictionary

| HTTP | Error | Meaning | Action |
|------|-------|---------|--------|
| 401 | Invalid or expired x-user-token | Token bad, expired, or quota exhausted | Check token; retry after 1s; contact admin if persistent |
| 404 | Pair code not found | Code expired (3 min) or already used | Refresh the display page for a new code |
| 409 | Assignment not active | Lease expired / display offline | Renew (`POST /v1/pair/renew`) or re-pair |
| 409 | Display already assigned | Another agent holds this display | Wait for lease to expire, or release from dashboard |
| 415 | Unsupported Media Type | Missing `Content-Type: application/json` | Always set it on POST requests |
| 429 | Rate limit exceeded | Too many requests | Wait 1–2 s. Demo token: 50/day. Production token: 1000/day (resets UTC midnight) |
| 413 | Payload Too Large | Content over 50 MB | Reduce size (PDF/images); use the gateway proxy for large files |

### Token Best Practices

- Store the token in a config file or environment variable — never hardcode it in shell commands
- Use scoped tokens (e.g. `push` only) when you don't need pair/control
- Quota resets at midnight UTC (demo: 50/day · production: 1000/day)

---

## License

Maintained by **Microsense Vision Co., Ltd.**  
For commercial use, contact Allan@msviso.com

---

Made with 🦞 by the Microsense Vision team
