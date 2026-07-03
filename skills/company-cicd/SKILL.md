---
skill: company-cicd
id: S5
type: generation
archetype: agentic
base: AgentCatalyst greenfield company-cicd
---

# company-cicd — /implement Generation Skill (S5)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
Default Cloud Build + Cloud Deploy; Enterprise overlay Jenkins + Harness. Adds Agent Registry registration as a post-deployment step.

> The full generation patterns are inherited from the AgentCatalyst greenfield `company-cicd`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] Golden dataset ≥10 entries per agent
- [ ] Pre-commit hook wired
- [ ] CI/CD pipeline YAML present
- [ ] Eval pipeline configured
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_golden_dataset.py`
