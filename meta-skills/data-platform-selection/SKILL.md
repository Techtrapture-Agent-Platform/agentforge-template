---
meta-skill: data-platform-selection
version: "2.0.0"
type: retrieve-then-reason
discovery_tool: search_tools
---

# Data Platform Selection Meta-Skill

## Purpose
For each data system in spec §4, select the platform tool(s) and infra module(s) by
RETRIEVING candidates from the Tool Registry and REASONING about workload fit. This
skill contains NO mapping of data-system names to MCP servers or Terraform modules.
That mapping lives in the Tool Registry corpus.

## Hard rule
NEVER name an MCP server or infra module from memory. Only the tools `search_tools`
returns exist. If a described workload returns no platform tool, emit a WARN
("no platform tool in registry for <workload>") — do not invent a server name.

## Decision procedure
1. For each data system in §4, write a natural-language description of its WORKLOAD
   characteristics — access pattern, shape, latency, volume, consistency — NOT the
   product name.
   - Example: "low-latency transactional reads and writes over normalized,
     relational claims records with strong consistency."
2. Call `search_tools(query=<workload description>, filters={category:"data-platform"})`.
3. Reason over the candidates:
   - Pick the best-fit candidate per workload by `score` AND by whether its
     `capability` / `data_systems_supported` actually match the access pattern.
   - ANTI-PATTERN CHECK: if the top candidate is an analytical store but the workload
     is transactional (or vice-versa), reject it and pick the next candidate that
     fits; record the rejection in §11 ADL.
   - OWNERSHIP CHECK: if §4 (Data Systems) or the ownership section (what we own
     vs. what we connect to) says the system already EXISTS ("existing database",
     "existing REST API"), select a CONNECT-only tool and emit NO infra module —
     do not provision.
   - CLASSIFICATION CHECK: read the chosen tool's `classification_constraints[]`; if
     the data-classification section (sensitive / regulated data) requires CMEK/VPC-SC
     and the tool supports it, propagate
     that requirement to the infra module; if the tool cannot meet it, WARN.
4. Take the `connection` block (endpoint, auth_method, timeout, retry) verbatim from
   the chosen candidate.

## Output
- `tool_bindings[]` (§5): one entry per selected platform tool, with provenance
  `{ tool_id, version }` and the connection block.
- `mcp_server_configs[]` (§7): server-level config for each distinct server.
- `infra_modules[]` (§8): only for systems being PROVISIONED (not connect-only),
  taken from the candidate's associated module reference.
