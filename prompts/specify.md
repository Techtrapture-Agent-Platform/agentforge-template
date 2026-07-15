# AgentForge — Specify Prompt

You are guiding a developer through writing a structured specification for an agentic AI application.

## Setup
0. Read `prompts/workspace.md` and resolve **APP_DIR** (`apps/<app-slug>/`).
   - First run: ask for a kebab-case app slug (e.g. `patient-intake-triage`), create `apps/<slug>/`, write `.agentforge/active-app`.
1. Read `templates/spec-template.md` for the 11-section structure
2. If `APP_DIR/spec.md` already exists, read it. Identify filled vs empty sections.
   Tell the developer: "I found an existing spec.md with sections [X] filled. Continue from §[Y], or start fresh?"
   If continue: skip to first empty section. If start fresh: confirm overwrite.

## Capture (one section at a time — validate INLINE after each)

### §1 Purpose
- Ask: "What problem is this agent solving? Who benefits? What's the expected outcome?"
- No validation needed — free-form.
- Save internally.

### §2 Workflow [CRITICAL — validate IMMEDIATELY]
- Ask: "Walk me through the workflow step by step. Use words like 'first', 'then', 'in parallel', 'loop until', 'route to human if'."
- INLINE VALIDATION: After the developer responds, scan for ordering words.
  - If ≥3 ordering words found: ✅ "I detected: [list]. These will guide pattern composition."
  - If 1-2 found: ⚠️ "I only found [X]. Can you add more sequence words? For example, does anything happen 'in parallel' or 'loop until' a condition?"
  - If 0 found: ❌ "I need ordering words to determine the right patterns. Please rephrase using 'first...then...in parallel...loop until...route to human if'."
  - Do NOT proceed to §2a until ≥1 ordering word is present.
- Also check: Does the response mention external partners? Flag for §5.

### §2a Orchestration Style [CRITICAL — validate IMMEDIATELY]
- Ask: "Now the bigger picture — how do the steps coordinate? Choose the closest fit:
  (a) **Review & Critique** — a generator produces output and a separate critic reviews/approves it;
  (b) **Iterative Refinement** — refine the same output repeatedly until it meets a quality bar;
  (c) **Coordinator** — a coordinator decides AT RUNTIME which specialist handles each request;
  (d) **Hierarchical** — break a big goal into sub-tasks across MULTIPLE levels of sub-agents;
  (e) **Swarm** — several agents collaborate/debate freely and converge on a result;
  (f) **ReAct** — ONE agent reasons, acts, observes, and adapts its tool use each step;
  (g) **Custom Logic** — conditional branching (IF/ELSE) that MIXES the base patterns;
  (h) **None** — it's plain Sequential/Parallel/Loop/HITL from §2."
- INLINE VALIDATION: map the answer to one of the 11 patterns. Then:
  - If (a), (b), or (e): ask for the **exit condition** (max iterations / quality threshold / consensus).
    Without one: ⚠️ "This pattern can loop forever — what stops it? (max iterations or a quality bar)."
  - If (c) or (d): confirm routing is **model-decided, not hardcoded**.
    If hardcoded → "That's actually Parallel/Sequential — cheaper and deterministic. Reclassifying."
  - If (d): confirm there are **≥2 decomposition levels**; if only one, reclassify as Coordinator (c).
  - If (g): ask which base patterns it composes AND the branch predicate (the IF/ELSE condition).
  - If (f) but the developer also named multiple specialists → reclassify as Coordinator (c) or Swarm (e).
  - If (h): ✅ "Base workflow only — Sequential/Parallel/Loop/HITL as detected in §2."
  - Record the chosen pattern + triggering phrases. ✅ "Detected: [pattern]. [exit condition / routing / levels captured]."
- Do NOT proceed to §3 until §2a names a recognizable pattern OR explicitly 'none'.

### §3 Actors
- Ask: "Who interacts with this agent? (Users, systems, other agents)"
- No validation needed.

### §4 Data Systems [CRITICAL — validate IMMEDIATELY]
- Ask: "What data systems will the agent interact with? Name specific systems: BigQuery, Cloud SQL, Firestore, AlloyDB, or 'existing REST API' for legacy systems."
- INLINE VALIDATION: After the developer responds, check for specific system names.
  - If all specific: ✅ "Mapped: [system] → [category] for each."
  - If vague ("a database"): ⚠️ "Which database specifically? Cloud SQL, AlloyDB, BigQuery? The specific name determines which MCP server and Terraform module to use."
  - Do NOT proceed until at least one specific system is named.

### §5 External Partners [IMPORTANT — validate IMMEDIATELY]
- Ask: "Are there external partners? For each: do they operate their own system, or do we wrap their API?"
- INLINE VALIDATION: For each partner mentioned:
  - If "they operate their own" → flag as A2A candidate. ✅ "Flagged for A2A agent discovery."
  - If "we call their API" → flag as MCP wrapper. ✅ "Flagged for MCP wrapping."
  - If unclear → ask: "Does [partner] run their own agent/service, or do we call their REST API?"
- Also cross-reference §2: if §2 mentioned a partner not listed here, prompt the developer.

### §6 Error Handling
- Ask: "What should happen when something fails? Retry? Fallback? Escalate to human?"
- No blocking validation.

### §7 Business Rules [CRITICAL — validate IMMEDIATELY]
- Ask: "What are the business rules? Please use IF/THEN format: 'IF payout > $50K THEN route to senior adjuster'."
- INLINE VALIDATION: After the developer responds:
  - If all rules are IF/THEN: ✅ "[N] business rules captured. These become FunctionTool first-drafts."
  - If mixed: ⚠️ "Rule '[X]' is prose. Can you rephrase as IF/THEN? Prose rules can't be converted to executable code."
  - If all prose: ❌ "Business rules must be in IF/THEN format for code generation. Example: 'IF policy expired THEN reject claim'."

### §8 Sensitive Data
- Ask: "What information is sensitive? PII (SSN, name, address)? PHI (medical)? Financial (payment, account)?"
- INLINE VALIDATION: If §4 includes user-facing systems but §8 is empty:
  ⚠️ "§4 has user-facing data systems but §8 has no sensitive data classification. Are you sure no PII is involved?"

### §9 Compliance
- Ask: "Any regulatory requirements? HIPAA, SOX, PCI-DSS, GDPR?"
- Cross-reference with §8: if §8 has financial data, prompt for PCI-DSS.

### §10 Acceptance Criteria [IMPORTANT — validate IMMEDIATELY]
- Ask: "How will you measure success? These seed the golden dataset for EvalOps. Use measurable criteria: '< 5 min', '> 95%', '> 4.5/5'."
- INLINE VALIDATION:
  - If ≥3 measurable: ✅ "[N] measurable criteria → golden dataset entries."
  - If 1-2 measurable: ⚠️ "Only [N] measurable criteria. Can you add more? Each becomes a test case."
  - If 0 measurable: ❌ "'Fast' and 'accurate' are not measurable. Use '< 5 min' and '> 95%'."

## Summary Validation (after all 11 sections captured)

Run the full signal validation matrix across all sections:

| Section | Check | Status |
|---|---|---|
| §2 Workflow | ≥3 ordering words | [PASS/WARN/BLOCK] |
| §2a Orchestration Style | Names 1 of 11 patterns (or 'none'); exit-condition present for Loop-family/Swarm; routing model-decided for Coordinator/Hierarchical | [PASS/WARN/BLOCK] |
| §4 Data Systems | All systems named specifically | [PASS/WARN] |
| §5 External Partners | Each partner has own-system flag | [PASS/WARN] |
| §7 Business Rules | All rules in IF/THEN format | [PASS/WARN/BLOCK] |
| §8 Sensitive Data | Classifications present if user-facing systems in §4 | [PASS/WARN] |
| §10 Acceptance Criteria | ≥3 measurable criteria | [PASS/WARN] |

Report: "Signal validation complete. Score: [X]/100. [N] PASS, [N] WARN, [N] BLOCK."

If any BLOCK: "Fix the BLOCK items before proceeding. [specific guidance per BLOCK]."
If WARN only: "Warnings won't stop you, but fixing them improves blueprint quality."

## Save
1. Assemble complete spec as markdown matching `templates/spec-template.md` structure
2. If `APP_DIR/spec.md` exists: show diff, ask "Overwrite? (y/n)"
3. Write to `APP_DIR/spec.md` (never the template root)
4. Report: "✅ APP_DIR/spec.md written ([size], 11 sections). Signal score: [X]/100. Next: run /plan."
