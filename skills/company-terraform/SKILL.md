---
skill: company-terraform
id: S3
type: generation
archetype: agentic
base: AgentCatalyst greenfield company-terraform
---

# company-terraform — /implement Generation Skill (S3)

## Purpose
Generation skill invoked by `/implement` to produce production-ready code/IaC for this
concern from the cosign-signed `design-contract.json`.

## AgentForge-specific difference (vs. AgentCatalyst base)
Generates Agent Gateway proxy routes (not Apigee), Agent Identity tokens (not Workload Identity), and an Agent Registry entry (not API Hub). Uses company-terraform modules from the GitHub MCP Server.

**Multi-region / DR:** Read `plan.md` primary + DR region and DR strategy. Terraform must declare:
- Primary region modules in `var.region` (primary)
- DR region replica / standby flags per strategy (Warm Standby → active DR Cloud Run min instances; Pilot Light → minimal DR footprint)
- Outputs: `primary_service_url`, `dr_service_url` (when applicable) for `docs/DEPLOYMENT.md` and `docs/DISASTER-RECOVERY.md`

> The full generation patterns are inherited from the AgentCatalyst greenfield `company-terraform`
> skill. Apply the AgentForge difference above. (See: AgentCatalyst Architecture Appendix
> S1–S6.) This file is the AgentForge overlay; keep the base patterns in sync via the
> `base:` reference in the front-matter.

## Validation Hook
After generating each file for this skill, verify:
- [ ] No `resource "google_*"` blocks (company `module "company-*"` only)
- [ ] CMEK + VPC-SC present
- [ ] `tags` block on every module
- [ ] Agent Identity least-privilege
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_tf_modules.py, hooks/check_identity.py`
