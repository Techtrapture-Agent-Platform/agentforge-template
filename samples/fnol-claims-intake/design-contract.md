---
template: agentforge-design-contract
version: "1.0.0"
generated_by: Design Agent MCP Server
cosign_signed: true
diagram_tool: drawio
generated_at: "2026-05-23T14:30:00Z"
---

# Design Contract — FNOL Claims Intake

## 1. Metadata
| Field | Value |
|---|---|
| Name | fnol-claims-agent |
| Version | 1.0.0 |
| Use case | First Notice of Loss claims intake automation |
| Team | claims-engineering |
| Generated at | 2026-05-23T14:30:00Z |

## 2. Pattern Composition
| Root pattern | Composed patterns |
|---|---|
| Sequential | Sequential → Parallel → Loop → HITL |

Confidence: 0.95. Rationale: spec §2 uses "First... then... in parallel... loop until... route to human" — triggering all 4 patterns in sequence.

## 3. Agent Topology — Structured Topology Language (STL)

<!-- topology:begin — machine-parseable, human-editable. Diagram tools render FROM this. validate_contract checks diagram AGAINST this. -->

### Agents
| Agent | Type | Role | Parent | Model | Tools |
|---|---|---|---|---|---|
| fnol_coordinator | SequentialAgent | Root orchestrator | (root) | — | — |
| extract_details | LlmAgent | Extract claimant details | fnol_coordinator | gemini-2.0-flash | claims-db-mcp, coverage_calculator_fn |
| parallel_enrichment | ParallelAgent | Concurrent enrichment | fnol_coordinator | — | — |
| enrich_policy | LlmAgent | Policy enrichment | parallel_enrichment | gemini-2.0-flash | policy-api-mcp |
| enrich_vehicle | LlmAgent | Vehicle enrichment | parallel_enrichment | gemini-2.0-flash | vehicle-api-mcp |
| enrich_weather | LlmAgent | Weather enrichment | parallel_enrichment | gemini-2.0-flash | weather-api-mcp |
| severity_classifier | LlmAgent | Severity classification | fnol_coordinator | gemini-2.0-flash | severity_classifier_fn, body-shop-a2a |
| human_review | LlmAgent | Human adjuster routing | fnol_coordinator | gemini-2.0-flash | review-queue-mcp |

### Tool Bindings
| Agent | Tool | Protocol | Purpose |
|---|---|---|---|
| extract_details | claims-db-mcp | MCP | Claim record CRUD |
| extract_details | coverage_calculator_fn | FunctionTool | Policy coverage validation |
| enrich_policy | policy-api-mcp | MCP | Policy lookup |
| enrich_vehicle | vehicle-api-mcp | MCP | Vehicle history lookup |
| enrich_weather | weather-api-mcp | MCP | Weather at incident location |
| severity_classifier | severity_classifier_fn | FunctionTool | IF/THEN severity rules |
| severity_classifier | body-shop-a2a | A2A | Body shop estimate delegation |
| human_review | review-queue-mcp | MCP | Route to adjuster queue |

### A2A Delegations
| Agent | Delegates To | Capabilities | Source |
|---|---|---|---|
| severity_classifier | body-shop-agent (v2.3) | estimate, schedule, parts-lookup | Agent Registry |

### Data Flows
| From | To | Data | Protocol |
|---|---|---|---|
| (external) | fnol_coordinator | inbound claim (phone/web) | HTTP |
| fnol_coordinator | extract_details | raw claim data | internal |
| extract_details | claims-db-mcp | claim_id, claimant details | MCP |
| fnol_coordinator | parallel_enrichment | claim_context | internal |
| enrich_policy | policy-api-mcp | policy_number | MCP |
| enrich_vehicle | vehicle-api-mcp | vin | MCP |
| enrich_weather | weather-api-mcp | incident_location, date | MCP |
| parallel_enrichment | fnol_coordinator | enriched_claim | internal |
| fnol_coordinator | severity_classifier | enriched_claim | internal |
| severity_classifier | body-shop-a2a | estimate_request | A2A |
| fnol_coordinator | human_review | classified_claim (if critical/high) | internal |
| fnol_coordinator | (external) | claim confirmation | HTTP |

### Sequence Summary
1. Customer submits claim → fnol_coordinator receives inbound claim
2. fnol_coordinator → extract_details extracts claimant info from claims-db-mcp
3. fnol_coordinator → parallel_enrichment fans out to 3 sources in parallel
4. enrich_policy queries policy-api-mcp | enrich_vehicle queries vehicle-api-mcp | enrich_weather queries weather-api-mcp
5. parallel_enrichment consolidates → returns enriched_claim to fnol_coordinator
6. fnol_coordinator → severity_classifier applies business rules (severity_classifier_fn)
7. IF severity ≥ critical AND fraud_score < 0.7 → severity_classifier delegates to body-shop-a2a via A2A
8. IF severity = critical OR high → fnol_coordinator routes to human_review via review-queue-mcp
9. fnol_coordinator returns claim confirmation to customer

<!-- topology:end -->

## 4. Component Diagrams
<!-- Generated from §3 STL by Draw.io (plan.md: drawio) -->
![Component Diagram](diagrams/component-topology.png)

## 5. Tool Bindings
| Tool | Type | Endpoint | Agent | Discovered via | Confidence |
|---|---|---|---|---|---|
| claims-db-mcp | MCP Server | mcp://claims-db.internal:8080 | extract_details | Vertex AI Search (tool registry) | 0.95 |
| policy-api-mcp | MCP Server | mcp://policy-api.internal:8080 | enrich_policy | Vertex AI Search (tool registry) | 0.92 |
| vehicle-api-mcp | MCP Server | mcp://vehicle-api.internal:8080 | enrich_vehicle | Vertex AI Search (tool registry) | 0.88 |
| weather-api-mcp | MCP Server | https://api.weather.gov/points | enrich_weather | Vertex AI Search (tool registry) | 0.90 |
| review-queue-mcp | MCP Server | mcp://review-queue.internal:8080 | human_review | Vertex AI Search (tool registry) | 0.85 |
| body-shop-a2a | A2A Agent | https://bodyshop.partner.com/a2a | severity_classifier | Agent Registry (v2.3, healthy) | 0.80 |
| severity_classifier_fn | FunctionTool | — | severity_classifier | spec §7 business rules | 0.95 |
| coverage_calculator_fn | FunctionTool | — | extract_details | spec §7 policy rules | 0.90 |

## 6. Skill References
| Skill | Version | SHA | Agent | use_when |
|---|---|---|---|---|
| claims-intake-skill | v2.1.0 | sha256:abc123 | extract_details | FNOL intake with structured data extraction |
| severity-classification-skill | v1.3.0 | sha256:def456 | severity_classifier | IF/THEN severity rules with enrichment |

## 7. MCP Server Configs
| MCP Server | Auth | Endpoint | Timeout | Retry |
|---|---|---|---|---|
| claims-db-mcp | mTLS | mcp://claims-db.internal:8080 | 30s | 3 |
| policy-api-mcp | OAuth 2.1 | mcp://policy-api.internal:8080 | 15s | 2 |
| vehicle-api-mcp | API Key | mcp://vehicle-api.internal:8080 | 15s | 2 |
| weather-api-mcp | None | https://api.weather.gov/points | 10s | 1 |
| review-queue-mcp | mTLS | mcp://review-queue.internal:8080 | 60s | 1 |

## 8. Infrastructure Modules
| Module | Source | Version | Component | DR aware |
|---|---|---|---|---|
| tf-agentic-pilot-cold | github.com/anchorops/tf-agentic-pilot-cold | v3.1.0 | Cloud Run + Agent Engine | yes |
| tf-cloud-sql | github.com/anchorops/tf-cloud-sql | v2.1.0 | Claims database | yes |
| tf-model-armor | github.com/anchorops/tf-model-armor | v1.3.0 | Model Armor config | no |
| tf-vpc-sc | github.com/anchorops/tf-vpc-sc | v1.2.0 | VPC-SC perimeter | yes |

## 9. Business Rules
### severity_classifier_fn
- IF injury_reported = true OR vehicle_total_loss = true THEN return "critical"
- IF damage_amount > 10000 AND injury_reported = false THEN return "high"
- IF damage_amount > 2500 AND damage_amount <= 10000 THEN return "medium"
- IF damage_amount <= 2500 THEN return "low"

### coverage_calculator_fn
- IF policy_status != "active" THEN reject claim
- IF date_of_loss > today() THEN reject with "future_date"

## 10. Non-Functional Requirements
| Category | Requirement | Target |
|---|---|---|
| Latency | End-to-end processing | < 30s (p95) |
| Throughput | Concurrent claims | ≥ 100 |
| Availability | Service uptime | 99.9% |
| Data retention | Claim data | 7 years |
| Security | PII encryption | AES-256 at rest, TLS 1.3 in transit |

## 11. Architecture Decisions Log
| Decision | Rationale | Alternatives |
|---|---|---|
| SequentialAgent as root | Spec §2 "First... then..." requires ordered steps | ParallelAgent (rejected: steps have data deps) |
| ParallelAgent for enrichment | Spec §2 "in parallel" + 3 independent sources | SequentialAgent (rejected: 3× slower) |
| A2A for body shop | Spec §5 "they operate their own" — separate domain | MCP (rejected: partner manages their agent) |
| Cloud SQL for claims | Spec §4 "transactional database" — relational CRUD | Firestore (rejected: relational data, need joins) |

## 12. Tech Stack
| Component | Technology | Version | Justification |
|---|---|---|---|
| LLM | Gemini 2.0 Flash | latest | Balance of speed + quality |
| Database | Cloud SQL (PostgreSQL) | 15 | Relational CRUD for claims |
| Agent runtime | Agent Engine | GA | GCP-native ADK runtime |
| CI/CD | Cloud Build + Cloud Deploy | — | GCP-native default |
| Observability | Cloud Monitoring + Cloud Trace | — | GCP-native default |

## 13. HA/DR Views
![HA/DR Lifecycle](diagrams/hadr-diagram.png)
DR strategy: pilot-cold. Primary: us-east1. DR: us-west1.

## 14. Sequence Diagrams
```mermaid
sequenceDiagram
    participant User
    participant Coordinator as fnol_coordinator
    participant Extract as extract_details
    participant Par as parallel_enrichment
    participant Policy as enrich_policy
    participant Vehicle as enrich_vehicle
    participant Weather as enrich_weather
    participant Severity as severity_classifier
    participant Review as human_review

    User->>Coordinator: Submit claim
    Coordinator->>Extract: Extract details
    Extract->>Coordinator: Claim record created
    Coordinator->>Par: Enrich (parallel)
    par Policy enrichment
        Par->>Policy: Lookup policy
        Policy->>Par: Policy details
    and Vehicle enrichment
        Par->>Vehicle: Lookup vehicle
        Vehicle->>Par: Vehicle details
    and Weather enrichment
        Par->>Weather: Get weather
        Weather->>Par: Weather data
    end
    Par->>Coordinator: All enrichments complete
    Coordinator->>Severity: Classify severity
    Severity->>Coordinator: severity = "medium"
    Coordinator->>Review: Route for review (if critical/high)
    Review->>Coordinator: Approved
    Coordinator->>User: Claim #554 confirmed
```

## 15. Agent Identity Config
| Agent | Capabilities | Denied | Delegation | Data |
|---|---|---|---|---|
| fnol_coordinator | — (orchestrator) | all tools | extract_details, parallel_enrichment, severity_classifier, human_review | none |
| extract_details | claims-db-mcp:read-write | policy-api, vehicle-api, weather-api, review-queue | — | pii |
| enrich_policy | policy-api-mcp:read-only | claims-db, vehicle-api, weather-api, review-queue | — | policy |
| enrich_vehicle | vehicle-api-mcp:read-only | claims-db, policy-api, weather-api, review-queue | — | none |
| enrich_weather | weather-api-mcp:read-only | claims-db, policy-api, vehicle-api, review-queue | — | none |
| severity_classifier | severity_classifier_fn | claims-db, policy-api, vehicle-api, weather-api, review-queue | body-shop-a2a | none |
| human_review | review-queue-mcp:write-only | claims-db, policy-api, vehicle-api, weather-api | — | pii |
