# /tasks — Implementation Task Breakdown

## Prerequisites

- Read `prompts/workspace.md` and resolve **APP_DIR**
- `APP_DIR/design-contract.md` reviewed
- Prefer `APP_DIR/design-contract.json` from `/validate` (warn if missing)

## Your job

1. Read `APP_DIR/design-contract.json` (or `APP_DIR/design-contract.md` if json missing)
2. Read `templates/tasks-template.md`
3. List every artifact `/implement` will generate under **APP_DIR** vs what developer writes manually

## Categorize

### Generated (from design contract + 6 skills)

Map each agent, tool binding, and infra module to:
- skill name (adk-agents, adk-tools, company-terraform, etc.)
- validation hook that will run in pre-commit

### Manual (developer-owned)

Typical manual work:
- Domain system prompts
- FunctionTool business logic refinement
- Golden dataset edge cases beyond seed
- Compliance audit sign-off

## Output

Save `APP_DIR/tasks.md`. Paths in the task list must be under `APP_DIR/` (e.g. `APP_DIR/app/agents/…`). Tell developer to run `/implement` when ready.
