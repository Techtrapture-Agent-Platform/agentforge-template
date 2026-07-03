---
skill: company-observability
id: S4
type: generation
archetype: agentic
base: AgentCatalyst greenfield company-observability
---

# company-observability — /implement Generation Skill (S4)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
GCP-native defaults (Cloud Monitoring, Cloud Trace) over the same OTel + Dynatrace + Splunk patterns as the base.

> The full generation patterns are inherited from the AgentCatalyst greenfield `company-observability`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] OTel spans on every agent class
- [ ] Structured JSON logging (no print())
- [ ] Health-check endpoints present
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_otel.py`
