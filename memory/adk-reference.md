# ADK framework reference (Google Agent Development Kit)

> Loaded by the coding agent during `/implement`. **Runtime is Google ADK only** (`google-adk`).

## Official docs
- Index for LLMs / coding agents: https://adk.dev/llms.txt
- Python install + quickstart: https://adk.dev/get-started/python/index.md → `pip install google-adk`
- Agents: https://adk.dev/agents/llm-agents/index.md
- Workflows: Sequential / Parallel / Loop under https://adk.dev/agents/workflow-agents/
- Tools: https://adk.dev/tools-custom/function-tools/index.md · https://adk.dev/tools-custom/mcp-tools/index.md
- A2A: https://adk.dev/a2a/index.md

## Install / dependency
```toml
# APP_DIR/pyproject.toml
dependencies = [
  "google-adk>=1.0.0",
]
```

```bash
pip install google-adk
adk run APP_DIR/app   # or package dir that exports root_agent
adk web --port 8000
```

## Core imports (Python)
```python
from google.adk.agents import LlmAgent, SequentialAgent, ParallelAgent, LoopAgent
# LlmAgent is also aliased as Agent in some docs:
# from google.adk.agents.llm_agent import Agent

from google.adk.tools import FunctionTool
from google.adk.tools.mcp_tool import McpToolset  # or MCPToolset depending on version
from google.adk.tools.mcp_tool.mcp_session_manager import (
    StreamableHTTPConnectionParams,
    SseConnectionParams,
    StdioConnectionParams,
)
```

## Auth (ask at /implement Step 1.5 — same options ADK documents)

Before generating code, `/implement` **asks the user** and writes `APP_DIR/auth-config.yml`.

| User choice | Mechanism | Env / setup |
|-------------|-----------|-------------|
| (a) AI Studio API key | `ai_studio_api_key` | `GOOGLE_API_KEY=…` |
| (b) Vertex / Agent Platform — ADC | `vertex_adc` | `gcloud auth application-default login`; `GOOGLE_GENAI_USE_VERTEXAI=TRUE`; `GOOGLE_CLOUD_PROJECT`; `GOOGLE_CLOUD_LOCATION` |
| (c) Service account | `service_account` | Same Vertex flags + `GOOGLE_APPLICATION_CREDENTIALS=/path/key.json` (or Workload Identity on GCP) |
| (d) Express Mode | `express_mode` | `GOOGLE_GENAI_USE_ENTERPRISE=TRUE`; `GOOGLE_GENAI_API_KEY=…` |

Docs: https://adk.dev/agents/models/google-gemini/index.md · https://adk.dev/get-started/google-cloud/index.md

`app/agent.py` must call `configure_auth()` before building agents. Never hardcode secrets.

## Tools — FunctionTool first; remote MCP only

| Binding | Use when | Do not |
|---------|----------|--------|
| `FunctionTool` | Business rules, transforms, company REST/API, anything a function can do | — (this is the default) |
| `McpToolset` client | Public/vendor **remote MCP already exists** | Build/scaffold an MCP server |
| A2A | Partner operates their own agent | Substitute for FunctionTool/MCP |

**Remote MCP examples (client only):**
- GitHub: https://api.githubcopilot.com/mcp/ — [docs](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- Google Cloud products: [supported MCP products](https://docs.cloud.google.com/mcp/supported-products)
- Databases: [MCP Toolbox for Databases](https://mcp-toolbox.dev/documentation/introduction/)

If `tool_bindings` says MCP but no real remote exists → emit **FunctionTool** instead.

## Topology → ADK
| design-contract.json | ADK construct |
|----------------------|---------------|
| LlmAgent | `LlmAgent(name=, model=, instruction=, tools=, output_key=)` |
| SequentialAgent | `SequentialAgent(name=, sub_agents=[…])` |
| ParallelAgent | `ParallelAgent(name=, sub_agents=[…])` |
| LoopAgent | `LoopAgent(name=, sub_agents=[…], max_iterations=N)` |

Export `root_agent` from `app/agent.py` for the ADK CLI.

## Forbidden
- LangChain / LangGraph / CrewAI / AutoGen / LlamaIndex as the agent runtime
- Direct OpenAI / Anthropic / raw Gemini chat loops in place of `LlmAgent`
- Omitting `google-adk` from `pyproject.toml`
- Inventing MCP servers or fake `mcp://….internal` connections when FunctionTool works

## AgentForge note
AgentForge `/implement` always materializes `adk_agent_tree` as **Google ADK** Python code under `APP_DIR/app/`.
