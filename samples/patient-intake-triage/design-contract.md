# Design Contract — Patient Intake Triage Agent (Golden Reference Outline)

This is the golden reference — what the Design Agent SHOULD produce. TechTrapture compares the Design Agent's actual output against this.

> **Note:** The full 15-section golden design-contract.md for App 1 is provided as a separate deliverable (`golden-design-contract-app1.md`). It includes:
> - §1 Solution Overview (from spec §1-§3 + plan)
> - §2 Pattern Composition (Sequential, confidence 0.95, adjacency validated)
> - §3 Agent Topology with full STL (5 sub-tables: Agents, Tool Bindings, A2A Delegations, Data Flows with 12 edges, Sequence Summary with 9 steps)
> - §4 Tool Bindings (3 MCP + 1 A2A + 1 FunctionTool with endpoints, auth, discovered_via)
> - §5 FunctionTool Specification (triage_classifier_fn Python implementation)
> - §6 Infrastructure Modules (4 TF module repos)
> - §7 Security Configuration (per-agent Model Armor, Workload Identity SAs)
> - §8 Agent Identity Configuration (least-privilege: capabilities, denied, delegation per agent)
> - §9 Business Rules (5 IF/THEN rules from spec §7)
> - §10 Observability (Dynatrace + Splunk + OTel + dashboards + alerting)
> - §11 CI/CD Pipeline (Cloud Build + Cloud Deploy, 3-phase EvalOps)
> - §12 Gateway Routes (5 routes: 1 external + 3 MCP + 1 A2A)
> - §13 NFR Targets (12-row table with latency, throughput, availability, RTO/RPO, HIPAA)
> - §14 A2A Agent Card Reference (pharmacy-agent v1.2, capabilities, SLA)
> - §15 Golden Dataset Seed (10 entries covering 5 acceptance criteria)

> NOTE: The full 15-section golden `design-contract.md` for App 1 is delivered separately as `golden-design-contract-app1.md` (per the M2-MVP doc). This file captures its expected structure/outline for repo completeness.
