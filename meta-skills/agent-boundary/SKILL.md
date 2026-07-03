---
meta-skill: agent-boundary
version: "2.0.0"
type: retrieve-then-reason
discovery_tools: [search_agents, search_tools]
priority: A2A > MCP > Build
---

# Agent Boundary Meta-Skill

## Purpose
For each integration point, decide A2A (reuse a deployed agent), MCP (connect to an
existing tool), or Build (create new). The decision is made by RETRIEVING candidates
from each surface in PRIORITY ORDER and reasoning about ownership and fit. This skill
contains NO signal→decision table.

## Hard rule
NEVER assert that a partner agent exists from memory. A2A is only valid if
`search_agents` returns a HEALTHY agent whose Agent Card SEMANTICALLY matches the
needed capability. If nothing matches at a higher priority, fall through to the next.

## Decision procedure (priority order — stop at the first that fits)
For each integration point:
1. Describe the CAPABILITY needed in natural language (what the integration must do),
   plus the OWNERSHIP signal from §5 (External Partners) and the ownership section
   ("they operate their own", "external team
   manages", "existing API — must use", "we need to build").

2. **A2A first — Agent Card semantic search.**
   Call `search_agents(query=<capability description>, filters={status:"healthy"})`.
   This runs a SEMANTIC search over deployed agents' Agent Cards
   (`description` + `skills[].description`), not an exact name/capability match.
   - If a candidate's Agent Card semantically matches AND it is `healthy` AND its
     `sla` meets the integration's need → recommend **A2A**. Capture provenance
     `{ agent_name, version, agent_card_url, status }`.
   - HIGH-STAKES OVERRIDE: if the capability involves high-value financial
     transactions or regulated PII movement, route the A2A call THROUGH Agent Gateway
     (not direct peer-to-peer) and note it.

3. **MCP second.**
   If no agent fits, call `search_tools(query=<capability description>)`. If an
   existing tool covers the capability → recommend **MCP** (consume, don't own).
   Capture `{ tool_id, version, connection }`.

4. **Build last.**
   If neither surface returns a fit → recommend **Build** (new FunctionTool or
   LlmAgent). Record in §11 ADL WHY nothing was reusable (this is the signal the
   catalog/registry has a gap).

## Reasoning rules (not in the corpus)
- Ownership beats convenience: "they operate their own / separate lifecycle" pushes
  toward A2A even when an MCP wrapper would be technically possible.
- Never recommend Build for a capability a healthy A2A agent already provides — that
  breaks the reuse flywheel.

## Output
- For each integration point: the decision (A2A/MCP/Build), the surface it came from,
  and full provenance. Feeds §3 (A2A delegations), §5 (tool_bindings), and the
  Gateway/A2A route derivation in Layer 3.
