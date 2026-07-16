# AgentForge — Antigravity Workspace Rules

This file defines the project-scoped rules for Antigravity in the AgentForge template. Since Antigravity does not support custom, user-defined slash commands (like `/specify`) natively in the chat input (only built-in ones like `/goal`, `/schedule`, `/grill-me`, `/learn`), these rules instruct the agent to route user command intents to the appropriate prompt files.

## Command Routing
When the user mentions or requests any of the following commands, read the specified prompt file and follow its instructions exactly:
* **"specify" or "/specify"** -> Read [specify.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/specify.md) and follow it.
* **"plan" or "/plan"** -> Read [plan.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/plan.md) and follow it.
* **"design" or "/design"** -> Read [design.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/design.md) and follow it.
* **"validate" or "/validate"** -> Read [validate.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/validate.md) and follow it.
* **"tasks" or "/tasks"** -> Read [tasks.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/tasks.md) and follow it.
* **"implement" or "/implement"** -> Read [implement.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/implement.md) and follow it.

## Execution Rules
1. **Workspace Resolution**: Always resolve `APP_DIR` via [workspace.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/prompts/workspace.md) first (which is `apps/<slug>/`). Never write spec, plan, design, or code into the template root.
2. **Specification Layout**: Always fill `APP_DIR/spec.md` against [spec-template.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/templates/spec-template.md) (11 sections).
3. **Implementation Protocol**:
   * `/implement` **MUST** ask the user which ADK authentication mechanism to use (Step 1.5) and wait for a/b/c/d before generating any code; write `APP_DIR/auth-config.yml` then `app/auth.py`.
   * After code generation, ask: "Do you want basic testing?" — only run import smoke + hooks if the user approves.
   * Always generate `APP_DIR/docs/DEPLOYMENT.md` and `docs/DISASTER-RECOVERY.md` (GCloud deployment + DR runbook).
   * `/implement` **MUST** generate Google ADK agents only (package `google-adk`, imports from `google.adk`).
   * Follow [SKILL.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/skills/adk-agents/SKILL.md) and [adk-reference.md](file:///Users/divyanshubhati/Desktop/AgentForge/agentforge-template/memory/adk-reference.md); documentation at `https://adk.dev/llms.txt`.
   * **Never** use LangChain/LangGraph/CrewAI/AutoGen or raw LLM SDK chat loops as the agent runtime.
   * **Tools**: Use `FunctionTool` by default. Use MCP only as a client to existing remote/public MCPs (GitHub, GCP MCP, MCP Toolbox for Databases, etc.) — never build custom MCP servers.
