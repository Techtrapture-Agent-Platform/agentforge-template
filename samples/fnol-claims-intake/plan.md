---
template: agentforge-plan
version: "2.0"
use_case: fnol-claims-intake
---

# Technical Plan — FNOL Claims Intake

## Infrastructure
- Primary GCP region: us-east1
- DR region: us-west1
- DR strategy: pilot-cold

## Model Selection
- Primary LLM: gemini-2.0-flash
- Fallback LLM: gemini-2.0-flash-lite

## CI/CD
- Default: Cloud Build + Cloud Deploy
- Enterprise overlay: Jenkins + Harness (if applicable)
- IaC module source: github.com/anchorops/terraform-modules

## Security
- Data classification: confidential (PII present)
- PII handling: encrypt at rest, mask in logs

## Observability
- Default: Cloud Monitoring + Cloud Logging + Cloud Trace
- Enterprise overlay: Dynatrace + Splunk (if applicable)

## EvalOps
- Evaluation frequency: pre-commit
- HITL reviewers: 3

## Diagram Tool
- Tool: drawio
- Rationale: Draw.io — free, works offline in VSCode extension, no account required
