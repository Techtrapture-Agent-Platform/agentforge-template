# plan.md — Patient Intake Triage Agent

## Runtime
- Framework: Google ADK (Python)
- Runtime: Cloud Run (GA)
- Region (primary): us-east4
- Model: gemini-2.0-flash (low latency, cost-effective for structured extraction)

## Security
- VPC-SC: Required (PHI)
- CMEK: Required (HIPAA)
- Model Armor: Standard (prompt + response screening for PHI leakage)
- Network: Private VPC, no public endpoints

## Gateway
- Apigee Runtime Gateway (GA)
- OAuth 2.1 for MCP server auth
- mTLS for service-to-service

## CI/CD
- Default: Cloud Build + Cloud Deploy
- Pipeline: Non-Prod → Pre-Prod (canary 10%) → Production
- EvalOps: 3-phase (pre-commit, AutoSxS, HITL review by clinical staff)

## DR Strategy
- Strategy: Warm Standby (healthcare = Tier-1 criticality)
- DR Region: us-central1
- RTO target: 15 minutes
- RPO target: 0 (synchronous replication)

## Observability
- APM: Dynatrace
- SIEM: Splunk (HIPAA-compliant log forwarding)
- Tracing: OTel → Cloud Trace
- Dashboards: Dynatrace (intake volume, latency P50/P95/P99, error rates per agent)

## EvalOps
- Evaluation frequency: pre-commit
- Golden dataset: 50 entries (10 per acceptance criterion × 5 criteria)
- HITL reviewers: 2 (clinical staff)

## Diagram Tool
- Tool: drawio
- Rationale: Draw.io — free, works offline, HIPAA-safe (no cloud rendering of PHI diagrams)
