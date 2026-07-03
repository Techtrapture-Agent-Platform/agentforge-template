---
meta-skill: pattern-composition
version: "2.0.0"
type: retrieve-then-reason
discovery_tool: search_patterns
progressive_disclosure: L1 → L2 → L3
---

# Pattern Composition Meta-Skill

## Purpose
Determine the agent design pattern(s) the spec implies, validate their composition,
and emit the topology. This skill does NOT contain a catalog of patterns. The catalog
lives in the Pattern Catalog data store. This skill describes the workflow's
coordination BEHAVIOR in natural language, retrieves candidate patterns via
`search_patterns`, and REASONS over the candidates' `required_invariants` and
`adjacency_constraints`. Retrieval supplies the candidates; this skill supplies the
architecture.

## Hard rule
NEVER name a pattern from memory. The only patterns that exist are the ones
`search_patterns` returns. If retrieval is empty for a described behavior, emit a
WARN ("no catalog pattern matches the described coordination") — do not invent one.

## L1 — Behavior extraction and retrieval (always loaded)
1. Read spec §2 (Workflow), §2a (Orchestration Style), the Error Handling
   section, and §7 (Business Rules). Refer to sections by their HEADING, not a
   fixed number — the spec's section numbering varies by capture path.
2. For each distinct coordination behavior you can identify, write a ONE-SENTENCE
   natural-language description of *how the work is coordinated* — NOT a keyword.
   - Describe the actors, the hand-offs, the decision points, and any stop condition.
   - Example description (not a lookup): "a generator produces a draft and a separate
     critic approves or rejects it against fixed criteria, repeating until approved
     or a max iteration count is hit."
3. Call `search_patterns(query=<that description>)` once per distinct behavior.
4. Keep the top candidates with their `required_invariants`, `adk_realization`, and
   `adjacency_constraints`. These fields come FROM the retrieved documents — treat
   them as authoritative, even for patterns this skill has never seen.

## L2 — Legality reasoning over retrieved candidates (loaded when ≥1 candidate)
For each candidate pattern, verify the spec satisfies the candidate's
`required_invariants`. Do NOT hardcode which patterns need what — read it from the
candidate:
- If the candidate lists `exit_condition` and the spec gives no stop condition → BLOCK
  that candidate with the disambiguating question.
- If the candidate lists `model_routing` and the spec's routing is actually fixed →
  DOWN-CLASSIFY: re-query `search_patterns` for a deterministic alternative
  (e.g. "independent sub-tasks dispatched concurrently with no model decision") and
  prefer the cheaper deterministic candidate. Record the down-classification rationale
  in §11 ADL.
- If the candidate lists `min_decomposition_levels>=2` and the spec has one level →
  re-query for the single-level equivalent.
Attach a confidence score and the triggering signals to each surviving candidate.

## L3 — Composition (loaded when >1 surviving pattern)
1. When the workflow combines patterns, call
   `search_patterns(query=<composition description>)` (describe the nesting/
   adjacency relationship; no family filter — composition and adjacency templates
   are retrieved semantically)
   to retrieve composition/adjacency templates.
2. Validate the proposed nesting against every candidate's `adjacency_constraints`
   (e.g. a constraint forbidding a loop nested directly inside a parallel branch).
   Reject illegal nestings and explain in §11 ADL.
3. Resolve to the concrete ADK realization from each candidate's `adk_realization`.

## Output (design-contract.md §2 Pattern Composition + §3 STL seed)
- The selected pattern(s) and their composition.
- Per selection: confidence, the spec signals that drove the query, and provenance
  `{ pattern_id, version }` of the retrieved candidate.
- WARN entries for ambiguous/empty retrieval, surfaced for the developer to resolve
  at /validate.
