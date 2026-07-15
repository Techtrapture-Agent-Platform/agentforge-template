---
skill: adk-agents
id: S1
type: generation
archetype: agentic
runtime: google-adk
docs: https://adk.dev/llms.txt
---

# adk-agents — /implement Generation Skill (S1)

## Purpose
Generate **production agent code using Google Agent Development Kit (ADK) only**
from the validated `design-contract.json` `adk_agent_tree`.

**Package:** [`google-adk`](https://adk.dev/get-started/python/index.md) (`pip install google-adk`)  
**Canonical docs index:** [https://adk.dev/llms.txt](https://adk.dev/llms.txt)  
**Language:** Python 3.10+ (prefer `>=3.11` to match AgentForge apps)

## Strict runtime rule (NON-NEGOTIABLE)

| Allowed | Forbidden |
|---------|-----------|
| `google-adk` / `from google.adk…` | LangChain, LangGraph, CrewAI, AutoGen, LlamaIndex, Semantic Kernel, Haystack |
| ADK `LlmAgent` / `Agent`, `SequentialAgent`, `ParallelAgent`, `LoopAgent` | Hand-rolled chat loops, custom “agent” classes that call the LLM directly |
| Gemini (or other models via ADK model APIs) | Raw `openai.*`, `anthropic.*`, `google.generativeai` / `google.genai` chat loops outside ADK |
| Tools via ADK (`tools=[…]` on `LlmAgent`) | Direct HTTP/LLM calls inside agent modules to “act as” the agent |

If the design-contract implies another framework: **still emit Google ADK code** that realizes the same topology. Do not invent a non-ADK runtime.

Before generating, skim these official pages (via [llms.txt](https://adk.dev/llms.txt)):

- [Python quickstart](https://adk.dev/get-started/python/index.md) — install + `root_agent`
- [LlmAgent](https://adk.dev/agents/llm-agents/index.md)
- [Sequential](https://adk.dev/agents/workflow-agents/sequential-agents/index.md) /
  [Parallel](https://adk.dev/agents/workflow-agents/parallel-agents/index.md) /
  [Loop](https://adk.dev/agents/workflow-agents/loop-agents/index.md)
- [Function tools](https://adk.dev/tools-custom/function-tools/index.md)
- [MCP tools](https://adk.dev/tools-custom/mcp-tools/index.md)
- [A2A](https://adk.dev/a2a/index.md)

## AgentForge input

Read **`APP_DIR/design-contract.json`** (not `.md`):

- `adk_agent_tree[]` — each node: `name`, `type`, `role`, `parent`, `model`, `tools`
- `tool_bindings[]` — bind names in `tools` following the **tool selection policy** below (detail in `skills/adk-tools/SKILL.md`)
- Pattern invariants from `pattern_composition` (e.g. Loop `max_iterations` / exit condition)

## Tool selection policy (NON-NEGOTIABLE)

Default to **ADK `FunctionTool`**. Use **remote MCP (client only)** only for generic public
capabilities that already expose a hosted / remote MCP endpoint. **Never scaffold, invent,
or “build” a custom MCP server** in generated app code.

| Prefer | When |
|--------|------|
| **`FunctionTool`** | Business rules (IF/THEN), transforms, scoring, domain logic, wrapping a company REST/gRPC API, anything a short Python function can do |
| **Remote MCP client** (`McpToolset`) | A **public / vendor remote MCP** already exists for that capability — connect only, do not implement the server |
| **A2A** | External partner runs their own agent (own-system flag in §5) — not a substitute for MCP |

### Allowed remote MCP examples (connect as ADK client)

| Capability | Remote MCP (already available) | Docs |
|------------|--------------------------------|------|
| GitHub (issues, PRs, repos, …) | `https://api.githubcopilot.com/mcp/` | [GitHub remote MCP](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md) |
| Google Cloud products | Google Cloud MCP servers for supported products | [Supported products](https://docs.cloud.google.com/mcp/supported-products) |
| Databases (BigQuery, Cloud SQL, Postgres, Spanner, …) | [MCP Toolbox for Databases](https://mcp-toolbox.dev/documentation/introduction/) | Connect via Toolbox / ADK; do not hand-roll a DB MCP |

Same pattern applies to other **vendor-hosted** remote MCPs (Stripe, Notion, …) when the
contract names them and a public remote URL/docs exist.

### Forbidden MCP generation

- ❌ Do **not** create `mcp_connections/` wrappers for proprietary “fake” MCP URLs like `mcp://loan-origination…` when no real remote MCP exists
- ❌ Do **not** generate MCP server projects, stdio servers, or toolbox config except when the contract explicitly requires connecting to an **existing** Toolbox/GCP/GitHub (etc.) remote endpoint
- ❌ Do **not** use MCP for business rules, quality rubrics, or thin REST shims — those are **`FunctionTool`**

When `tool_bindings[]` says `type=mcp` but no real public/remote MCP exists for that
system: **reclassify to `FunctionTool`** (or A2A if partner-owned agent) and note the
reclassification in the module docstring.

Map `type` → ADK class:

| Contract `type` | ADK class | Import |
|-----------------|-----------|--------|
| `LlmAgent` | `LlmAgent` (alias `Agent`) | `from google.adk.agents import LlmAgent` |
| `SequentialAgent` | `SequentialAgent` | `from google.adk.agents import SequentialAgent` |
| `ParallelAgent` | `ParallelAgent` | `from google.adk.agents import ParallelAgent` |
| `LoopAgent` | `LoopAgent` | `from google.adk.agents import LoopAgent` |

Workflow agents (`Sequential` / `Parallel` / `Loop`) are **deterministic orchestrators** — they take `sub_agents=[…]`, not an LLM `model`. Only `LlmAgent` nodes get `model`, `instruction`, and `tools`.

## Output layout (under APP_DIR)

```
APP_DIR/app/
├── __init__.py
├── agent.py                 # configure_auth() then exports root_agent for `adk run` / `adk web`
├── auth.py                  # from APP_DIR/auth-config.yml (Step 1.5 — user-chosen)
├── agents/
│   ├── __init__.py
│   ├── {root_name}.py       # build_root_agent() / composition
│   └── {child_name}.py      # one module per adk_agent_tree node
├── tools/                   # FunctionTool (default — business rules, company APIs)
├── mcp_connections/         # ONLY clients to existing remote MCP (GitHub/GCP/Toolbox/…)
├── a2a_clients/             # A2A clients (partner-owned agents)
└── security/                # Model Armor callbacks (company-security)
```

Also ensure:
- `APP_DIR/pyproject.toml` lists **`google-adk`**
- `APP_DIR/auth-config.yml` exists (written in `/implement` Step 1.5 — **never invent auth silently**)
- `APP_DIR/.env.example` documents the required env vars for the chosen mechanism

## Auth (from user choice — Step 1.5)

`/implement` **must ask** the user which ADK auth mechanism to use (same options ADK itself
documents) and only then generate `app/auth.py`. Choices map as follows
([Gemini](https://adk.dev/agents/models/google-gemini/index.md),
[Google Cloud](https://adk.dev/get-started/google-cloud/index.md)):

| Choice | `auth-config.yml` `mechanism` | What `configure_auth()` sets |
|--------|-------------------------------|------------------------------|
| (a) Google AI Studio API key | `ai_studio_api_key` | Expect `GOOGLE_API_KEY`; do **not** set Vertex flags |
| (b) Vertex / Agent Platform — user ADC | `vertex_adc` | `GOOGLE_GENAI_USE_VERTEXAI=TRUE` (or `GOOGLE_GENAI_USE_ENTERPRISE`), project + location; unset `GOOGLE_API_KEY` if present so ADC wins |
| (c) Service account | `service_account` | Same Vertex/Enterprise flags + document `GOOGLE_APPLICATION_CREDENTIALS` (or Workload Identity on GCP — no key file) |
| (d) Express Mode | `express_mode` | `GOOGLE_GENAI_USE_ENTERPRISE=TRUE` + `GOOGLE_GENAI_API_KEY` |

Wire auth in `app/agent.py`:

```python
from app.auth import configure_auth
from app.agents.{root_module} import build_root_agent

configure_auth()  # MUST run before agents talk to Gemini
root_agent = build_root_agent()
```

Never hardcode keys or service-account JSON in generated source.

## Generation rules

### 1. LlmAgent leaves
Per [LlmAgent docs](https://adk.dev/agents/llm-agents/index.md):

```python
from google.adk.agents import LlmAgent

agent = LlmAgent(
    name="extract_details",           # required; match contract name
    model="gemini-2.0-flash",         # from contract.model
    description="…",                # from contract.role
    instruction="…",                # from §3 / spec purpose; use {state_key} for state
    tools=[…],                      # FunctionTool by default; remote McpToolset / A2A only per policy
    output_key="optional_state_key",  # when downstream agents read state
)
```

- Prefer `LlmAgent` over the `Agent` alias in generated code (clearer in multi-agent trees).
- Wire Model Armor via ADK callbacks when the agent handles PII (see `skills/company-security/SKILL.md`): `before_model_callback` / `after_model_callback`.
- **Never** call Gemini/OpenAI/Anthropic SDKs from agent modules.

### 2. SequentialAgent
Per [Sequential workflow](https://adk.dev/agents/workflow-agents/sequential-agents/index.md):

```python
from google.adk.agents import SequentialAgent

root = SequentialAgent(
    name="fnol_coordinator",
    description="…",
    sub_agents=[child_a, child_b, child_c],  # order = contract parent→child order
)
```

Pass data across steps with `output_key` on children and `{var}` in later instructions.

### 3. ParallelAgent
Per [Parallel workflow](https://adk.dev/agents/workflow-agents/parallel-agents/index.md):

```python
from google.adk.agents import ParallelAgent

enrichment = ParallelAgent(
    name="parallel_enrichment",
    description="…",
    sub_agents=[policy_agent, vehicle_agent, weather_agent],
)
```

### 4. LoopAgent
Per [Loop workflow](https://adk.dev/agents/workflow-agents/loop-agents/index.md):

```python
from google.adk.agents import LoopAgent

refinement = LoopAgent(
    name="enters_refinement_loop",
    description="…",
    sub_agents=[generator, critic],
    max_iterations=5,  # REQUIRED when pattern is Loop / Review&Critique / Iterative
)
```

Always set an exit: `max_iterations` and/or escalate from a tool/`ToolContext.actions.escalate` as in the official loop examples. Never emit an unbounded loop.

### 5. Tree assembly
1. Build leaves first, then parents, then root (respect `parent` edges in `adk_agent_tree`).
2. Export **`root_agent`** from `app/agent.py` so `adk run` / `adk web` work ([quickstart](https://adk.dev/get-started/python/index.md)):

```python
# app/agent.py
from app.auth import configure_auth
from app.agents.{root_module} import build_root_agent

configure_auth()
root_agent = build_root_agent()
```

3. Name modules after contract agent names (`drafts_credit_memo.py` → `build_drafts_credit_memo()`).

### 6. Tools (delegate detail to adk-tools)
- Attach tools only on `LlmAgent` nodes, names matching `adk_agent_tree[].tools` and `tool_bindings[]`.
- Apply **Tool selection policy** above before choosing a binding type.
- **FunctionTool (default)** → `from google.adk.tools import FunctionTool` ([docs](https://adk.dev/tools-custom/function-tools/index.md))
- **Remote MCP client only** → `McpToolset` / `MCPToolset` against an existing public URL ([MCP tools](https://adk.dev/tools-custom/mcp-tools/index.md))
- **A2A** → partner agents ([A2A](https://adk.dev/a2a/index.md))
- Never generate a custom MCP server under `APP_DIR`

## Minimal root skeleton (copy pattern)

```python
"""Root agent — built by AgentForge /implement from design-contract.json."""
from __future__ import annotations

from google.adk.agents import SequentialAgent  # or ParallelAgent / LoopAgent / LlmAgent

from app.agents.child_a import build_child_a
from app.agents.child_b import build_child_b


def build_root_agent():
    return SequentialAgent(
        name="example_coordinator",
        description="Root orchestrator from adk_agent_tree",
        sub_agents=[build_child_a(), build_child_b()],
    )
```

## Validation Hook
After generating each file for this skill, verify:
- [ ] File imports from `google.adk` only for agent constructs (no LangChain/CrewAI/etc.)
- [ ] Agent class is one of: `LlmAgent`, `SequentialAgent`, `ParallelAgent`, `LoopAgent`
- [ ] `LlmAgent` nodes have `name`, `model`, `instruction`; workflow agents use `sub_agents`
- [ ] `LoopAgent` has `max_iterations` (or documented escalate exit)
- [ ] `APP_DIR/auth-config.yml` exists (user chose auth in Step 1.5) and `app/auth.py` matches it
- [ ] `app/agent.py` calls `configure_auth()` before building `root_agent`
- [ ] Tool bindings follow FunctionTool-first / remote-MCP-only policy (no invented MCP servers)
- [ ] `mcp_connections/` only wraps real public/remote MCP URLs (GitHub, GCP MCP, Toolbox, …)
- [ ] Business rules / custom APIs use FunctionTool, not MCP
- [ ] No direct LLM SDK calls (`openai`, `anthropic`, raw `genai.Client` chat loops) in `app/agents/`
- [ ] `app/agent.py` (or package entry) exports `root_agent`
- [ ] `pyproject.toml` depends on `google-adk`
- [ ] No hardcoded API keys / SA JSON in source

If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_adk_patterns.py` — must reject non-`google.adk` agent frameworks and require ADK class constructs + `google-adk` in `pyproject.toml`.
