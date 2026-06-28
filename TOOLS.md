# AI2X Cast — Agent Configuration

Copy this into your agent's `TOOLS.md` and fill in your credentials.

## Configuration Block

```markdown
### AI2X Display

- API Base URL: https://ai2x.link
- API Token: (paste your token here)
- Channel Index: shared/skills/ai2x/channels.json
```

## Token Scopes

| Scope | Allows |
|-------|--------|
| `pair` | Claim/revoke displays |
| `push` | Push content, interactive buttons |
| `control` | Send events (page flip, scroll) |
| `history` | Browse/restore push history |

## Getting a Token

Contact your AI2X Gateway administrator or use the admin API:

```bash
curl -X POST http://localhost:8787/v1/admin/tokens \
  -H "x-admin-token: <admin-token>" \
  -d '{"label":"my-agent","scopes":"pair+push+control+history","daily_quota":1000}'
```

## Quick Test

```bash
TOKEN=ak_xxx
BASE=https://ai2x.link

# List displays
curl -H "x-user-token: $TOKEN" $BASE/v1/devices/displays
```
