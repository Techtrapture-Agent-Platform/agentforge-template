# Generated agent apps

Each slash-command run writes into **one app folder** under `apps/`. The AgentForge template (prompts, skills, meta-skills) stays at the repo root; your agent artifacts do not pollute it.

## Layout

```
apps/<app-slug>/
├── spec.md                 # /specify
├── plan.md                 # /plan
├── design-contract.md      # /design
├── design-contract.json    # /validate
├── auth-config.yml         # /implement Step 1.5 — user-chosen ADK auth
├── tasks.md                # /tasks
├── diagrams/               # /design
│   ├── component-topology.*
│   └── hadr-lifecycle.*    # HA/DR — 4 scenarios from plan.md DR strategy
├── app/                    # /implement — agent code (incl. auth.py)
├── deployment/terraform/   # /implement — IaC
├── ci-cd/                  # /implement
├── eval/                   # /implement
├── hooks/                  # /implement
├── config/                 # /implement
├── docs/
│   ├── DEPLOYMENT.md       # /implement — deploy to Google Cloud
│   └── DISASTER-RECOVERY.md
├── README.md
├── Dockerfile
└── pyproject.toml
```

## Active app

`/specify` asks for an `app-slug` (e.g. `patient-intake-triage`) and creates `apps/<app-slug>/`.

It also writes `.agentforge/active-app` at the template root with that slug. Later commands (`/plan`, `/design`, …) read that file and use `apps/<slug>/` as **APP_DIR**.

To switch apps: edit `.agentforge/active-app` or say which slug to use when starting a command.

## Do not write to template root

Never put `spec.md`, `design-contract.md`, or generated `app/` code in the AgentForge template root. Always use `APP_DIR`.
