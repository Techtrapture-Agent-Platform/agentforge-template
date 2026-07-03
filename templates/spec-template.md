---
template: agentforge-spec
version: "2.0"
archetype: agentic
---

# Agent Specification

## 1. Purpose
<!-- What business problem does this agent solve? Who are the users? What's the current process? What's the expected outcome? -->

## 2. Workflow — Step by Step
<!-- Describe the agent's workflow using ordering words: "First... then... in parallel... loop until... route to..." -->
<!-- These are SIGNALS for the Design Agent's pattern-composition meta-skill (base patterns: Sequential, Parallel, Loop, HITL). -->

## 2a. Orchestration Style — Which Agent Design Pattern?
<!-- The bigger picture: how do the steps coordinate? Name the closest agent design pattern so pattern-composition (MS1) -->
<!-- can classify across ALL 11 patterns, not just the 4 base ones. See: https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system -->
<!-- Pick ONE and write the signal phrase: -->
<!--   Review & Critique  → "a generator produces X, then a critic reviews/approves it before use" (give the critic's pass/fail criteria + exit condition) -->
<!--   Iterative Refinement → "refine X repeatedly until <quality bar>, max <N> iterations" (give the exit condition) -->
<!--   Coordinator        → "a coordinator decides at runtime which specialist handles each request" (routing is MODEL-decided, not hardcoded) -->
<!--   Hierarchical       → "break the goal into sub-tasks across multiple levels of sub-agents" (must be ≥2 decomposition levels) -->
<!--   Swarm              → "agents collaborate/debate and hand off to whoever fits, converge on <consensus/goal>" (give the exit condition) -->
<!--   ReAct              → "ONE agent reasons → acts → observes → adapts its tool choice each step until done" (single-agent) -->
<!--   Custom Logic       → "IF <condition> do <pattern A> ELSE do <pattern B>" (name the base patterns it mixes + the branch predicate) -->
<!--   None               → "base workflow only — Sequential/Parallel/Loop/HITL as in §2" -->
<!-- Selected pattern: -->
<!-- Signal phrase / exit condition / routing / levels: -->

## 3. Actors
<!-- Who interacts with this agent? Users, upstream/downstream systems, other agents. Identifies integration points. -->

## 4. Data Systems
<!-- Databases, data warehouses, data lakes. Include: system name, read/write/both, data format, -->
<!-- and OWNERSHIP: does it already EXIST ("existing REST API / existing database — MUST use" → connect, don't provision) or is it NEW (provision)? -->

## 5. External Partners & Integrations
<!-- External APIs, partner systems, third-party services -->
<!-- Use "they operate their own" to signal A2A boundaries (triggers agent-boundary meta-skill) -->

## 6. Error Handling
<!-- Retry? Fallback? Human escalation? Timeout behavior? -->

## 7. Business Rules
<!-- IF/THEN conditions. These generate working FunctionTool code. -->
<!-- Format: IF <condition> THEN <action> [ELSE <alternative>] -->
<!-- Transformation rules belong here too. Format: TRANSFORM <source> TO <target> USING <logic> -->

## 8. Sensitive Data
<!-- What information is sensitive? PII, PHI, financial, regulated. Triggers Model Armor segmented callbacks. -->

## 9. Compliance & Regulatory Requirements
<!-- HIPAA, SOX, PCI-DSS, GDPR; data residency, audit trail, retention policies. Shapes security generation (VPC-SC, CMEK). -->

## 10. Acceptance Criteria
<!-- GIVEN/WHEN/THEN. These seed the golden dataset. -->
