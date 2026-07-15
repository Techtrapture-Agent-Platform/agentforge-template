---
skill: adk-tools
id: S2
type: generation
archetype: agentic
runtime: google-adk
docs: https://adk.dev/llms.txt
---

# adk-tools — /implement Generation Skill (S2)

## Purpose
Generate **Google ADK tool bindings** from `design-contract.json` `tool_bindings[]`
for attachment to ADK `LlmAgent.tools`.

**Package:** `google-adk`  
**Docs:** [llms.txt](https://adk.dev/llms.txt) · [Function tools](https://adk.dev/tools-custom/function-tools/index.md) · [MCP tools](https://adk.dev/tools-custom/mcp-tools/index.md) · [A2A](https://adk.dev/a2a/index.md)

## Decision tree (NON-NEGOTIABLE)

```
For each tool_binding:
  1. Is it a business rule / transform / score / domain IF-THEN?
       → FunctionTool   (NEVER MCP)
  2. Does an existing remote/public MCP already cover this capability?
       (GitHub / Google Cloud MCP / MCP Toolbox for Databases / other vendor remote MCP)
       → McpToolset CLIENT only — connect to the published URL; do NOT build a server
  3. Is the partner an external agent they operate themselves (§5 own-system)?
       → A2A client
  4. Otherwise (company REST, internal API, custom logic, no public MCP)?
       → FunctionTool wrapping the API / logic
```

**Rule of thumb:** If a Python function (or thin HTTP client inside `FunctionTool`) can do
the job, **do not create MCP**. MCP is only for **generic public tools** where a remote
MCP server is **already available**.

## Strict runtime rule
Use **only** ADK tool APIs (`from google.adk.tools…`). Do not invent custom tool runners
or LangChain tools. HTTP is allowed **inside** a `FunctionTool` body when wrapping a
legacy REST endpoint — still registered via ADK `FunctionTool`.

**Never generate an MCP server** (no FastMCP apps, no stdio MCP projects, no fake
`mcp://…` endpoints) under `APP_DIR`. Only generate **MCP clients** to real remotes.

## AgentForge input
- `tool_bindings[]` from `APP_DIR/design-contract.json`
- Names referenced by `adk_agent_tree[].tools`
- If contract labels something `type=mcp` but no real remote MCP exists → **reclassify to FunctionTool**

## Generation patterns

### 1. FunctionTool (DEFAULT — most bindings)

Use for: business rules, quality rubrics, classification helpers, company REST/gRPC
wrappers, anything not covered by a public remote MCP.

```python
from google.adk.tools import FunctionTool

def apply_credit_rules(loan_amount_cents: int, dti: float) -> dict:
    """Apply IF/THEN credit committee rules from the design contract."""
    # first-draft from §7 business_rules
    return {"approved": dti < 0.43}

credit_rules_tool = FunctionTool(func=apply_credit_rules)
```

Place under `APP_DIR/app/tools/{name}.py`.

REST brownfield example (still FunctionTool — **not** MCP):

```python
from google.adk.tools import FunctionTool
import httpx, os

async def verify_policy(policy_id: str) -> dict:
    """Call existing company REST — FunctionTool, not MCP."""
    token = os.environ["POLICY_API_TOKEN"]  # from Secret Manager
    async with httpx.AsyncClient() as client:
        r = await client.get(
            f"https://claims-api.internal/api/v2/policies/{policy_id}",
            headers={"Authorization": f"Bearer {token}"},
        )
        r.raise_for_status()
        return r.json()

verify_policy_tool = FunctionTool(func=verify_policy)
```

### 2. Remote MCP client ONLY (existing public / vendor servers)

Generate `APP_DIR/app/mcp_connections/{binding_name}.py` **only** when connecting to a
documented remote MCP. Auth via Secret Manager / env — never hardcode tokens.

#### GitHub remote MCP
Docs: [remote-server.md](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)  
URL: `https://api.githubcopilot.com/mcp/` (toolset URLs under `/mcp/x/…`)

```python
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPConnectionParams
import os

def get_github_mcp_toolset() -> McpToolset:
    return McpToolset(
        connection_params=StreamableHTTPConnectionParams(
            url="https://api.githubcopilot.com/mcp/",
            headers={"Authorization": f"Bearer {os.environ['GITHUB_MCP_TOKEN']}"},
        )
    )
```

#### Google Cloud MCP servers
Docs: [Supported products](https://docs.cloud.google.com/mcp/supported-products)  
Connect to the product’s published MCP endpoint for that GCP service — client only.

#### Databases — MCP Toolbox for Databases
Docs: [MCP Toolbox introduction](https://mcp-toolbox.dev/documentation/introduction/)  
Use Toolbox as the remote MCP (or ADK Toolbox SDK) for BigQuery / Cloud SQL / Postgres /
Spanner / etc. Do **not** invent a hand-rolled database MCP server in the app.

```python
# Connect to an already-deployed Toolbox MCP endpoint (URL from contract / env)
def get_db_toolbox_toolset() -> McpToolset:
    return McpToolset(
        connection_params=StreamableHTTPConnectionParams(
            url=os.environ["MCP_TOOLBOX_URL"],  # existing Toolbox deployment
            headers=auth_headers_from_secret_manager(),
        )
    )
```

Other vendor remote MCPs (when named in the contract and publicly documented) follow the
same pattern: **URL from vendor docs + ADK `McpToolset`**.

### 3. A2A
Generate clients under `APP_DIR/app/a2a_clients/` using ADK A2A patterns
([A2A intro](https://adk.dev/a2a/intro/index.md), [Python consume](https://adk.dev/a2a/quickstart-consuming/index.md))
when the partner operates their own agent. Do not invent a non-ADK RPC stack. Do not use
A2A as a substitute for FunctionTool or remote MCP.

## What NOT to generate

| Do not generate | Do instead |
|-----------------|------------|
| Custom MCP server for loan-origination / memo-store / review-queue / etc. | `FunctionTool` (or REST wrap) |
| Fake `mcp://….internal` connections | `FunctionTool` + real HTTPS API |
| MCP just because the contract string ends in `_mcp` | Reclassify to FunctionTool unless a real remote MCP exists |
| Duplicate MCP for pure IF/THEN rules | `FunctionTool` from business_rules |

## Validation Hook
After generating each file for this skill, verify:
- [ ] Imports are from `google.adk.tools` (or `google.adk.tools.mcp_tool…`)
- [ ] Business rules / custom company APIs are `FunctionTool`, not MCP
- [ ] Every `mcp_connections/` module targets a **real public/vendor remote MCP URL** (GitHub, GCP MCP, Toolbox, …) — not invented `mcp://` hosts
- [ ] No MCP **server** code was generated under APP_DIR
- [ ] Auth pulled from Secret Manager / env only (no hardcoded credentials)
If ANY check fails: delete the file and regenerate.

`/implement` derives the following enforcement script(s) from this hook section:
`hooks/check_tool_connections.py` — reject invented MCP URLs / MCP-for-business-rules;
require FunctionTool-first and remote-MCP-only clients.
