# /tasks — Implementation Task Breakdown

## Prerequisites

- `design-contract.md` reviewed
- Prefer `design-contract.json` from `/validate` (warn if missing)

## Your job

1. Read `design-contract.json` (or `.md` if json missing)
2. Read `templates/tasks-template.md`
3. List every artifact `/implement` will generate vs what developer writes manually

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

Save `tasks.md`. Tell developer to run `/implement` when ready.
