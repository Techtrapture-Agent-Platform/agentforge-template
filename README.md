# agentforge-enterprise (monorepo preset)

Enterprise AgentForge preset aligned with [AnchorOps/agentforge](https://github.com/AnchorOps/agentforge).

| Sibling folder | Purpose |
|----------------|---------|
| `../design-agent-server/` | Design Agent MCP (local: `localhost:8080/mcp`) |
| `../tf-agentforge/` | Platform Terraform (catalogs, storage, remote state) |
| `../m1-knowledge-base/` | Pattern/skill/tool catalog JSON |
| `../AgentForge-Docs/` | Architecture + developer guide |

## Layout

```
agentforge-template/
├── preset.yml                    # manifest
├── prompts/                      # /specify → /implement (+ workspace.md)
├── apps/                         # generated per-agent outputs (APP_DIR)
├── templates/
├── meta-skills/                  # MS1–MS4 (Design Agent)
├── skills/                       # S1–S6 (/implement)
├── memory/
├── schemas/
├── design-agent/system-prompt.md
├── hooks/README.md
├── samples/                      # reference examples (read-only)
├── .agentforge/active-app          # current apps/<slug>
├── .specify/enterprise-config.yml  # nonprod catalog IDs + MCP (monorepo only)
├── .claude/commands/
└── .cursor/{rules,mcp.json}
```

## App output folder

Slash commands write into **`apps/<app-slug>/`**, not the template root:

`/specify` → `spec.md` · `/plan` → `plan.md` · `/design` → `design-contract.md` + `diagrams/` · `/validate` → `design-contract.json` · `/implement` → `app/`, `deployment/`, `ci-cd/`, … (optional basic testing if user says yes)

See `apps/README.md` and `prompts/workspace.md`.

## Quick start

```bash
git clone https://github.com/Techtrapture-Agent-Platform/agentforge-template.git
cd agentforge-template

# 1. API key (get from your platform admin)
cp .env.local.example .env.local
# edit .env.local → AGENTFORGE_API_KEY=<your-key>

# 2. Open Cursor WITH the key loaded (required for /design and /validate MCP)
chmod +x scripts/cursor.sh
./scripts/cursor.sh

# 3. Workflow in Cursor chat
# /specify → /plan → /design → /validate → /implement
# (creates apps/<your-slug>/ and writes all artifacts there)
```

**Important:** `export AGENTFORGE_API_KEY=...` in a terminal is not enough if you open Cursor from the Dock.
MCP reads the key from Cursor's process environment — use `./scripts/cursor.sh` or run `cursor .` from the same terminal after `export`.

No local Design Agent server needed — MCP points at hosted Cloud Run (see `.cursor/mcp.json`).

## Docs

- `../AgentForge-Docs/agentforge-speckit-future-architecture.md`
- `../AgentForge-Docs/agentforge-speckit-future-developer-guide.md`
