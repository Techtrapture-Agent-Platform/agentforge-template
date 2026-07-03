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
├── design-contract-template.md
├── prompts/                      # /specify → /implement
├── templates/
├── meta-skills/                  # MS1–MS4 (Design Agent)
├── skills/                       # S1–S6 (/implement)
├── memory/
├── schemas/
├── design-agent/system-prompt.md
├── hooks/README.md
├── samples/
├── .specify/enterprise-config.yml  # nonprod catalog IDs + MCP (monorepo only)
├── .claude/commands/
└── .cursor/{rules,mcp.json}
```

## Quick start

```bash
cd ../design-agent-server
python3 -m venv .venv && .venv/bin/pip install -e .
cp .env.example .env
.venv/bin/python -m server.app

cd ../agentforge-template
cursor .
# /specify → /plan → /design → /validate → /tasks → /implement
```

## Docs

- `../AgentForge-Docs/agentforge-speckit-future-architecture.md`
- `../AgentForge-Docs/agentforge-speckit-future-developer-guide.md`
