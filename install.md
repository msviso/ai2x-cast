# Installing AI2X Cast for OpenClaw

## Prerequisites

- OpenClaw (any version with agent skill support)
- AI2X Gateway (self-hosted or use ai2x.link)
- An API token from your AI2X provider

## Installation

### Step 1: Copy the skill

```bash
# From your OpenClaw workspace root:
cp -r ai2x-cast/ shared/skills/ai2x/
```

Or clone directly into your workspace:

```bash
cd /path/to/your/openclaw/workspace
git clone https://github.com/msviso/ai2x-cast.git shared/skills/ai2x
```

### Step 2: Configure token

Add to your agent's configuration (e.g., `TOOLS.md`):

```markdown
### AI2X Display
- API Base URL: https://ai2x.link
- API Token: ak_xxx
- Channel Index: shared/skills/ai2x/channels.json
```

### Step 3: Verify

```bash
# Check your token works
curl -s -H "x-user-token: $TOKEN" https://ai2x.link/v1/devices/displays
# Should return {"ok":true, "displays":[...]}
```

## Getting an API Token

### Self-hosted Gateway

If you run the AI2X Gateway yourself:

```bash
# On your gateway server, get the admin token
curl -X POST http://localhost:8787/v1/admin/tokens \
  -H "x-admin-token: <admin-token>" \
  -H "Content-Type: application/json" \
  -d '{"label":"my-agent","scopes":"pair+push+control+history"}'
```

### ai2x.link (hosted)

Contact the provider at Allan@msviso.com to request an API key.

## Multi-Agent Setup

Each agent needs its own `channels.json` and API token. This allows independent display management.

```bash
# Agent A
shared/skills/ai2x/channels.json → controls "Meeting Room"
# Agent B
shared/skills/ai2x/channels.json → controls "Lobby Display"
```

## Uninstall

```bash
rm -rf shared/skills/ai2x/
```

Remove the configuration from your agent's `TOOLS.md`.
