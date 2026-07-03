---
template: agentforge-plan
version: "2.0"
archetype: agentic
---

# Technical Plan

## Infrastructure
- **Primary GCP region:** <!-- e.g., us-east1 -->
- **DR region:** <!-- e.g., us-west1 -->
- **DR strategy:** <!-- active-active | hot-standby | pilot-cold -->

## Model Selection
- **Primary LLM:** <!-- e.g., gemini-2.0-flash -->
- **Fallback LLM:** <!-- e.g., gemini-2.0-flash-lite -->

## CI/CD
- **Default (GCP-native):** Cloud Build + Cloud Deploy
- **Enterprise overlay (if applicable):** Jenkins + Harness
- **IaC module source:** <!-- e.g., github.com/anchorops/terraform-modules -->

## Security
- **Data classification:** <!-- public | internal | confidential | restricted -->
- **PII handling:** <!-- mask | tokenize | encrypt -->

## Observability
- **Default (GCP-native):** Cloud Monitoring + Cloud Logging + Cloud Trace
- **Enterprise overlay (if applicable):** Dynatrace + Splunk + Arize Phoenix

## EvalOps
- **Evaluation frequency:** <!-- pre-commit | nightly | weekly -->
- **HITL reviewers:** <!-- number of human reviewers for phase 3 -->

## Diagram Tool
- **Tool:** <!-- drawio | eraserio | svg -->
- **Options:** Draw.io (recommended — free, works offline in VSCode), Eraser.io (visual DSL, requires account), SVG (Canva/Figma import)
- **Note:** Only ONE diagram format will be generated. Choose the tool you'll use to edit architecture diagrams.
