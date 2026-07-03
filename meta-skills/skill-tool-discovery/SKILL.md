---
meta-skill: skill-tool-discovery
version: "2.0.0"
type: retrieve-then-reason
discovery_tools: [search_skills, search_tools]
---

# Skill & Tool Discovery Meta-Skill

## Purpose
For each agent node in the composed topology (from MS1), find the reusable generation
skills and tools that implement it — by SEMANTIC capability search, with transitive
dependency resolution. This skill matches by CAPABILITY, never by name.

## Hard rule
Match on capability description, not on names. Only skills/tools returned by
`search_skills` / `search_tools` exist. Do not assume a skill exists because its name
"sounds right." If a node's capability returns nothing, WARN — the node may need a
Build (coordinate with MS3) or a catalog gap report.

## Per-node procedure
For each agent node:
1. Write a natural-language description of the node's REQUIRED CAPABILITY
   (what it must do), including its archetype.
2. **Skill discovery.**
   Call `search_skills(query=<capability description>, filters={archetype:<archetype>})`.
   - Check each candidate's `do_not_use_when` BEFORE accepting it; discard violators.
   - Rank survivors by `score`; select the best fit; record alternatives in §11 ADL.
3. **Dependency closure (reasoning the corpus cannot do for you).**
   For each selected skill, read its `dependencies[]`. For every dependency not yet
   selected, resolve it (by id if exact, else `search_skills` for its capability).
   Repeat until the dependency set is CLOSED. Detect and break cycles; report any
   unresolved dependency as a BLOCK.
4. **Tool discovery for internal capabilities.**
   For capabilities not satisfied by a platform tool (MS2) or an A2A agent (MS3),
   call `search_tools(query=<capability description>)` at TOOL granularity (e.g.
   "execute a parameterized read against the claims store", not "the claims server").
   Take the `connection` block verbatim.
5. **Reconcile with MS3.** Do NOT search-to-build a capability already satisfied by a
   retrieved A2A agent or platform tool. Cross-check the boundary decisions first.

## Output (with provenance on every item)
- `skill_references[]` (§6): `{ skill_id, name, version, sha, assigned_to: <node> }`
  for every selected skill AND every resolved dependency.
- `tool_bindings[]` (§5): one entry per matched tool, `{ tool_id, version, connection }`.
- `mcp_server_configs[]` (§7): server config for each distinct server.
