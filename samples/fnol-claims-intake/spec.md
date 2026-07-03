---
template: agentforge-spec
version: "2.0"
archetype: agentic
use_case: fnol-claims-intake
---

# Agent Specification — FNOL Claims Intake

## 1. Business Context
Our auto insurance company processes approximately 2,000 FNOL claims per day. Currently handled by call center agents with 45-minute average handling time. We want an AI agent that automates intake to under 5 minutes.

## 2. Workflow — Step by Step
First, the customer calls or submits a claim. The system extracts claimant details (name, policy number, date of loss, description).

Then, in parallel, it enriches from three sources: policy details from the policy management system, vehicle details from the vehicle registry, weather conditions from weather.gov.

After enrichment, the system classifies the claim severity (critical, high, medium, low).

Loop until the quality score exceeds 0.85: validate all enriched data for completeness.

Route high-severity and critical claims to a human adjuster. Low and medium severity claims are auto-approved.

## 3. Regulatory Requirements
All claim data retained 7 years. PII encrypted at rest, masked in logs. Audit trail for every status change. US-only data residency.

## 4. Data Systems
Claims database (Cloud SQL) — read/write — claim records, status, notes.
Policy management system — read only — existing REST API at policy-api.internal.
Vehicle registry — read only — existing REST API at vehicle-api.internal.

## 5. External Partners & Integrations
Body shop network — they operate their own system. We send severity + vehicle details, they return estimates.
Weather.gov — public API, no auth.

## 6. What We Own vs What We Connect To
We OWN: Claims database, claim processing logic, severity rules.
We CONNECT TO: Policy system (EXISTING REST API), Vehicle registry (EXISTING REST API), Body shop (THEIR system), Weather.gov (public).

## 7. Business Rules
IF injury_reported = true OR vehicle_total_loss = true THEN severity = "critical"
IF damage_amount > 10000 AND injury_reported = false THEN severity = "high"
IF damage_amount > 2500 AND damage_amount <= 10000 THEN severity = "medium"
IF damage_amount <= 2500 THEN severity = "low"
IF severity IN ["critical", "high"] THEN route_to = "human_adjuster"
IF policy_status != "active" THEN REJECT claim WITH reason = "inactive_policy"

## 8. Transformation Rules
TRANSFORM claimant_phone TO E.164 format
TRANSFORM date_of_loss TO ISO 8601
TRANSFORM policy_number TO uppercase zero-padded (12 digits)

## 9. Error Handling
IF policy-api returns 503: retry 3x with backoff, then escalate to human.
IF vehicle-api returns 404: proceed without vehicle enrichment, flag for manual lookup.
IF weather.gov timeout: proceed without weather, set weather_enriched = false.

## 10. Acceptance Criteria
GIVEN policy POL-000123456, $3200 damage, no injuries WHEN processed THEN severity = "medium", auto-approved
GIVEN injury_reported = true WHEN classified THEN severity = "critical", routed to human
GIVEN inactive policy WHEN submitted THEN rejected with "inactive_policy"
GIVEN weather.gov timeout WHEN processing THEN completes with weather_enriched = false
GIVEN 100 concurrent claims WHEN submitted THEN all complete within 60s, zero corruption
