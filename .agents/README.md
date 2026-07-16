# Antigravity MCP config

([Antigravity MCP docs](https://antigravity.google/docs/mcp))

## Auto-load (like Cursor)

Antigravity automatically reads MCP from:

| Scope | Path |
|-------|------|
| **This repo** | `.agents/mcp_config.json` |
| Global (all projects) | `~/.gemini/config/mcp_config.json` |

**Important:** open Antigravity on the **`agentforge-template`** folder (where `.agents/` lives), not the parent `AgentForge` monorepo. Same idea as Cursor needing `.cursor/mcp.json` in the workspace root.

No MCP Store install needed — file present → servers show up after open / Refresh.

## Why `mcp-remote` (not direct `serverUrl`)

Cloud Run `/health` can be fine while Antigravity still fails with:

```text
sending "notifications/initialized": rejected by transport:
Post ".../mcp": context deadline exceeded
```

Antigravity’s HTTP MCP client opens a Streamable-HTTP GET/SSE stream during
handshake and waits for response headers before sending
`notifications/initialized`. That path often hangs against remote Streamable
HTTP (Cloud Run, gateways), so the client times out even when the service is up.

**Fix:** connect via local stdio + `mcp-remote` (Node/`npx`), which talks to
Cloud Run on your behalf.

## Setup

1. Node.js installed (`node -v`, `npx -v`)
2. Put the Cloud Run API key in `env.AGENTFORGE_API_KEY` (same value as
   Cloud Run `API_KEY` / `AGENTFORGE_API_KEY`)
3. Antigravity → Manage MCP Servers → Refresh

## Quick checks

```bash
# Service up (you already verified this in Postman)
curl -sS "https://design-agent-mcp-kri6xyjbwa-uk.a.run.app/health"

# Local proxy smoke (optional)
npx -y mcp-remote https://design-agent-mcp-kri6xyjbwa-uk.a.run.app/mcp \
  --header "Authorization: Bearer YOUR_KEY"
```
