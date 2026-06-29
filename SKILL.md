# AI2X Cast — Agent Skill

> Remote display control for AI agents. Push content, flip pages, collect button clicks (v1.2).
> Requires an API token. Auth header: `x-user-token`.

## Agent Keywords

```yaml
command: cast
synonyms:
  - push to screen
  - show on display
  - display this
  - screen cast
  - 投
  - 投上去
  - 顯示
  - 上螢幕
  - projizieren
  - proyección

use cases:
  - "Push today's calendar to the meeting room screen"
  - "把這份規格書投上去"
  - "Show the latest metrics on the office display"
  - "把股價圖 cast 上大螢幕"
```

### Agent Response Examples

> User: Push today's calendar to the meeting room screen
> Agent:
> 1. Look up channels.json → find nickname "meeting room" → get assignmentId
> 2. POST /v1/display { assignmentId, content: { title, body } }
> 3. POST /v1/pair/renew { assignmentId }
> 4. Report: "✅ Pushed calendar to meeting room screen"

> User: Show this on the display
> Agent:
> 1. Look up channels.json → find most recent active display
> 2. If not found → "Please open ai2x.link on the display and give me the pair code"

## Quick Install

1. Copy this folder to your agent workspace (`shared/skills/ai2x/`)
2. Set your API token in configuration
3. Ensure `channels.json` exists (empty `{"channels":[]}`)
4. Done — all operations are direct HTTP calls

## Prerequisites

- **API Base URL:** Set to your AI2X Gateway address (e.g. `https://ai2x.link`)
- **Auth:** Every request needs header `x-user-token: <your-token>`
- **Token scopes:** pair | push | control | history (default: all four)

## API Endpoints

> All requests use `curl -H "x-user-token: $TOKEN" <endpoint>`.
> Replace `$TOKEN` with your API key, `$BASE` with the API URL.

### 1. Pair Display

Claim a display using its 6-digit pair code (shown on screen).

```
POST /v1/pair/claim
```

```json
{
  "pairCode": "ABC123",
  "nickname": "Office Display",
  "leaseMs": 300000
}
```

Response: `{"ok":true, "assignmentId":"as_xxx", "deviceId":"disp_xxx", "leaseExpiresAt":"<ISO>"}`

### 2. Push Content

Push content to a paired display.

```
POST /v1/display
```

```json
{
  "assignmentId": "as_xxx",
  "content": {
    "title": "Hello",
    "body": "**Markdown** content"
  }
}
```

Response: `{"ok":true, "delivered":true, "templateId":"document.v2", "viewId":"vw_xxx"}`

| Content Field | Type | Required | Description |
|--------------|------|----------|-------------|
| `title` | string | optional | Content title |
| `body` | string | optional | Markdown body (document template) |
| `url` | string | optional | Image/PDF/YouTube URL |
| `items` | array | optional | `[{title, description?, meta?}]` — card list |
| `data` | object | optional | `{type, labels, datasets}` — chart data |
| `buttons` | array | optional | `[{id, label, style?}]` — interactive buttons |
| `level` | string | optional | `info` \| `warning` \| `critical` — alert level |
| `sections` | array | optional | `[{type, body?, url?, items?, ...}]` — mixed multi-section (auto-routes to `mixed.v2` when set) |
| `type` | string | optional | Force template: `document` \| `mixed` \| `image` \| `pdf` \| `chart` \| `cards` \| `interactive` |

### 3. Renew Lease

Extend display lease before it expires.

```
POST /v1/pair/renew
```

```json
{ "assignmentId": "as_xxx", "duration": 300000 }
```

### 4. Revoke Pairing

Unpair display and clear all content.

```
POST /v1/pair/revoke
```

```json
{ "assignmentId": "as_xxx" }
```

### 5. Check Status

```
GET /v1/pair/status?assignmentId=as_xxx
```

### 6. List Displays

```
GET /v1/devices/displays
```

### 7. Send Event (v1.2)

Remote control: page flip, scroll, zoom, refresh.

```
POST /v1/devices/event
```

```json
{ "assignmentId": "as_xxx", "event": "next_page" }
```

Events: `next_page` | `prev_page` | `scroll_up` | `scroll_down` | `refresh` | `zoom_in` | `zoom_out` | `goto_page`

**Jump to page (PDF only):**

```json
{
  "assignmentId": "as_xxx",
  "event": "goto_page",
  "data": {
    "page": 8
  }
}
```

### 8. Poll Buttons (v1.2)

Check for button tap events on interactive displays.

```
GET /v1/devices/disp_xxx/input?since=<timestamp>
```

### 9. View History (v1.2)

Get last 5 pushed items for a display.

```
GET /v1/devices/disp_xxx/history
```

### 10. Restore History (v1.2)

Restore a display to a previous push.

```
POST /v1/display/restore
```

```json
{ "assignmentId": "as_xxx", "viewId": "vw_xxx" }
```

### 11. Mixed Content (v1.2)

Push multi-section content with text, images, cards, and tables in a single view. Use the explicit template endpoint for maximum control, or smart routing with `content.type: "mixed"` for simplicity.

#### Explicit Template (POST /v1/push/content)

```
POST /v1/push/content
```

```json
{
  "assignmentId": "as_xxx",
  "templateId": "mixed.v2",
  "data": {
    "title": "Dashboard",
    "sections": [
      {
        "type": "text",
        "body": "## Progress\n\n- ✅ Review complete\n- 🚧 Testing in progress"
      },
      {
        "type": "image",
        "url": "https://example.com/chart.png",
        "caption": "Temp trend"
      },
      {
        "type": "cards",
        "items": [
          { "title": "✅ PCB", "description": "Done", "meta": "DONE" },
          { "title": "🔄 FW", "description": "Testing", "meta": "WIP" }
        ]
      },
      {
        "type": "table",
        "headers": ["Item", "Status", "Owner"],
        "rows": [["PCB Rev 1.0", "✅ Done", "Alice"], ["FW v1.2", "🚧 WIP", "Bob"]]
      }
    ]
  }
}
```

#### Smart Routing (POST /v1/display)

Set `content.type: "mixed"` with `content.sections` — the gateway automatically routes to `mixed.v2`:

```json
{
  "assignmentId": "as_xxx",
  "content": {
    "title": "Dashboard",
    "type": "mixed",
    "sections": [
      { "type": "text", "body": "# Text block" },
      { "type": "image", "url": "...", "caption": "..." },
      { "type": "cards", "items": [...] }
    ]
  }
}
```

**Supported section types:**

| Type | Required Fields | Optional Fields |
|------|----------------|----------------|
| `text` | `body` (markdown) | — |
| `image` | `url` | `caption` |
| `cards` | `items` [{title, description?, meta?}] | — |
| `table` | `headers` + `rows` | — |

## Endpoint Summary

| Purpose | Method | Endpoint | Auth |
|---------|--------|----------|------|
| Pair | POST | `/v1/pair/claim` | `x-user-token` |
| Push | POST | `/v1/display` | `x-user-token` |
| Push (precise) | POST | `/v1/push/content` | `x-user-token` |
| Renew | POST | `/v1/pair/renew` | `x-user-token` |
| Revoke | POST | `/v1/pair/revoke` | `x-user-token` |
| Status | GET | `/v1/pair/status?assignmentId=` | `x-user-token` |
| List | GET | `/v1/devices/displays` | `x-user-token` |
| Event | POST | `/v1/devices/event` | `x-user-token` |
| Jump to Page | POST | `/v1/devices/event` (with `data.page`) | `x-user-token` |
| Poll | GET | `/v1/devices/:deviceId/input` | `x-user-token` |
| History | GET | `/v1/devices/:deviceId/history` | `x-user-token` |
| Restore | POST | `/v1/display/restore` | `x-user-token` |

## Channel Index (Multi-Display Management)

Each agent maintains `channels.json` to map display nicknames to assignment IDs.

### Schema

```json
{
  "channels": [
    {
      "nickname": "Office Display",
      "deviceId": "disp_xxx",
      "assignmentId": "as_xxx",
      "status": "online",
      "pairedAt": "2026-06-27T14:00:00.000Z",
      "leaseExpiresAt": "2026-06-27T14:10:00.000Z",
      "lastRenewedAt": "2026-06-27T14:05:00.000Z"
    }
  ]
}
```

### Lookup Rules

1. User says display name → look up by `nickname` (exact match, then fuzzy)
2. If nickname not found → list all displays
3. If no active channel → guide user through pairing

## Performance Principle: Optimistic Push

**Never check status before pushing.** Push first, handle errors reactively.

```
✅ Optimistic Push:
POST /v1/display → 200 OK → done
                → 401/expired → guide re-pair
```

Rationale: Active leases are the common case (>99%). Status checks waste context tokens and API calls.

## Workflow: Standard Session

```
User: "Cast today's schedule to the reception display"

Agent:
1. Look up channels.json → find "reception" → get assignmentId
2. POST /v1/display { assignmentId, content: { title, body } }
3. POST /v1/pair/renew { assignmentId }
4. Report: "✅ Cast to reception display"
```

### First-time Pairing

```
1. User opens the display URL, reads 6-digit pair code
2. Agent: POST /v1/pair/claim { pairCode, nickname }
3. Agent: save { deviceId, assignmentId } to channels.json
4. Agent: POST /v1/display → content appears on screen
```

## Content Templates

| Type | Content Fields | Renders As |
|------|---------------|------------|
| Document | `body` (markdown) | Formatted document |
| Image | `url` | Full-width image |
| PDF | `url` (.pdf) | PDF viewer with page nav |
| YouTube | `url` | Embedded player |
| Cards | `items` [{title,description,meta}] | Card list |
| Chart | `data` {type,labels,datasets} | Line or pie chart |
| Alert | `level` (info/warning/critical) | Colored alert |
| Interactive | `buttons` [{id,label,style}] | Clickable buttons |
| Gallery | `items` [{url,caption}] | Image gallery |
| Mixed | `sections` | Multi-section layout |
| Form | `fields` [{label,value,type}] | Key-value display |
| List | `items` [{title,description}] | Bullet list |

## Getting an API Token

### ai2x.link (Hosted Trial)

For quick evaluation on ai2x.link, request a **temporary demo token** by emailing **Allan@msviso.com**. Demo tokens have limited quotas and may be rotated without notice.

> ⚠️ **Demo tokens are temporary.** They may be revoked or rotated at any time.
> For production use, please request a permanent token via email.

### Self-Service Token Portal ✅

Get an evaluation token instantly at **[ai2x.link/token](https://ai2x.link/token)**.

```bash
# Or from the command line:
curl -X POST https://ai2x.link/v1/token/request \
  -H "Content-Type: application/json" \
  -d '{"name":"Your Name","email":"you@example.com"}'

# Response:
# {
#   "ok": true,
#   "token": "ak_...",
#   "scopes": "pair+push",
#   "dailyQuota": 50,
#   "source": "web",
#   "info": "Evaluation token — limited to 50 requests/day..."
# }
```

> ⚠️ Demo tokens are limited to **50 requests/day** with `pair+push` scopes.
> Rate-limited to **3 requests per IP per hour**.
> For production use, email **Allan@msviso.com** for a permanent token.

## 🔒 Security Note

### Token Exposure

All API examples use `$TOKEN` as an environment variable. This is the recommended pattern:

```bash
# ✅ Safe — token in env var, never in shell history
TOKEN=ak_xxx
curl -H "x-user-token: $TOKEN" ...

# ❌ Unsafe — token hardcoded in command line, visible in history/process list
curl -H "x-user-token: ak_xxx" ...
```

### Agent Context

When an AI agent executes these commands, the token is **read from its secure configuration** (`TOOLS.md` or similar) and used as an environment variable. The full command is never displayed to the end user — agents report success/failure only.

### Best Practices

- Store your token in a config file or environment variable, never hardcode it
- Use scoped tokens (limit to `push` only if you don't need pair/control)
- Rotate tokens periodically via the admin API
- Self-hosted users: keep your `ADMIN_TOKEN` in a `.env` file, never in git

## Quick Reference

```bash
# Claim
curl -X POST "$BASE/v1/pair/claim" \
  -H "x-user-token: $TOKEN" \
  -d '{"pairCode":"ABC123", "nickname":"Office"}'

# Push
curl -X POST "$BASE/v1/display" \
  -H "x-user-token: $TOKEN" \
  -d '{"assignmentId":"as_xxx", "content":{"title":"Alert","body":"**Message**"}}'

# Renew
curl -X POST "$BASE/v1/pair/renew" \
  -H "x-user-token: $TOKEN" \
  -d '{"assignmentId":"as_xxx"}'

# Event (next/prev page)
curl -X POST "$BASE/v1/devices/event" \
  -H "x-user-token: $TOKEN" \
  -d '{"assignmentId":"as_xxx", "event":"next_page"}'

# Jump to page (PDF only)
curl -X POST "$BASE/v1/devices/event" \
  -H "x-user-token: $TOKEN" \
  -d '{"assignmentId":"as_xxx", "event":"goto_page", "data":{"page":8}}'

# Mixed content (text + image + cards)
curl -X POST "$BASE/v1/push/content" \
  -H "x-user-token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "assignmentId": "as_xxx",
    "templateId": "mixed.v2",
    "data": {
      "title": "Dashboard",
      "sections": [
        {"type": "text", "body": "# Overview\n- Task A: done"},
        {"type": "image", "url": "https://example.com/chart.png", "caption": "Chart"},
        {"type": "cards", "items": [{"title": "Task A", "meta": "✅"}]}
      ]
    }
  }'
```
