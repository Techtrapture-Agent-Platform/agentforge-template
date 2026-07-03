---
skill: adk-tools
id: S2
type: generation
archetype: agentic
base: AgentCatalyst greenfield adk-tools
---

# adk-tools — /implement Generation Skill (S2)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
Reads `tool_bindings[]` from design-contract.json. Same tool patterns as the AgentCatalyst base.

> The full generation patterns are inherited from the AgentCatalyst greenfield `adk-tools`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] Tool connections use MCP / A2A / FunctionTool (never raw HTTP)
- [ ] Auth pulled from Secret Manager only
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_tool_connections.py`
