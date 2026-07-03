---
skill: adk-agents
id: S1
type: generation
archetype: agentic
base: AgentCatalyst greenfield adk-agents
---

# adk-agents — /implement Generation Skill (S1)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
Reads design-contract.json `adk_agent_tree` (cosign-signed) instead of an unsigned blueprint. Same ADK agent patterns as the AgentCatalyst greenfield base.

> The full generation patterns are inherited from the AgentCatalyst greenfield `adk-agents`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] Agent class follows the ADK pattern
- [ ] FunctionTool bindings match §3 STL
- [ ] No direct LLM calls outside the agent framework
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_adk_patterns.py`
