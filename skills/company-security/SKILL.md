---
skill: company-security
id: S6
type: generation
archetype: agentic
base: AgentCatalyst greenfield company-security
---

# company-security — /implement Generation Skill (S6)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
Same Model Armor segmented callbacks. Generates per-agent Agent Identity from §15 `agent_identity_config{}` with explicit denied lists.

> The full generation patterns are inherited from the AgentCatalyst greenfield `company-security`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] Model Armor callbacks on every LlmAgent
- [ ] No hardcoded secrets (regex scan)
- [ ] Agent Identity with non-empty `denied[]`
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_model_armor.py, hooks/check_secrets.py`
