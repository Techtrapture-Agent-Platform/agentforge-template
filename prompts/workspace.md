# Workspace — Active App Directory (APP_DIR)

All slash-command **inputs and outputs** live under a single generated app folder. Do not write agent artifacts into the AgentForge template root.

## Resolve APP_DIR (every command)

1. If `.agentforge/active-app` exists at the template root, read the slug (one line, e.g. `patient-intake-triage`).
2. Set `APP_DIR = apps/<slug>/`.
3. If the file is missing or the folder does not exist:
   - Ask: "Which app slug? (folder under `apps/`, e.g. `patient-intake-triage`)"
   - Create `apps/<slug>/` if needed
   - Write `.agentforge/active-app` with that slug
4. Confirm to the developer: `Using APP_DIR=apps/<slug>/`

## Paths (always relative to APP_DIR)

| Artifact | Path |
|----------|------|
| Spec | `APP_DIR/spec.md` |
| Plan | `APP_DIR/plan.md` |
| Design contract | `APP_DIR/design-contract.md` |
| Validated JSON | `APP_DIR/design-contract.json` |
| Auth choice | `APP_DIR/auth-config.yml` |  # set in /implement Step 1.5 before code gen
| Tasks | `APP_DIR/tasks.md` |
| Diagrams | `APP_DIR/diagrams/` |
| Generated code | `APP_DIR/app/`, `APP_DIR/deployment/`, `APP_DIR/ci-cd/`, `APP_DIR/eval/`, `APP_DIR/hooks/`, `APP_DIR/config/` |
| Deploy guide | `APP_DIR/docs/DEPLOYMENT.md` |  # GCloud deploy steps (/implement)
| DR runbook | `APP_DIR/docs/DISASTER-RECOVERY.md` |  # failover/failback (/implement)
| HA/DR diagram | `APP_DIR/diagrams/hadr-lifecycle.*` |  # /design (+ .mermaid supplement at /implement if missing)
| App README / Docker | `APP_DIR/README.md`, `APP_DIR/Dockerfile`, `APP_DIR/pyproject.toml` |

## Template-root paths (read-only for generation)

| Resource | Path |
|----------|------|
| Spec/plan templates | `templates/` |
| Generation skills | `skills/` |
| Meta-skills | `meta-skills/` |
| Prompts | `prompts/` |

See `apps/README.md` for the full tree.
