# spec.md — Patient Intake Triage Agent

## §1. Use Case
Automate patient intake triage for a multi-location outpatient clinic network. When a patient
contacts the clinic via phone or web portal, the system extracts clinical information, verifies
insurance, determines urgency, and schedules an appropriate appointment — reducing intake time
from 15 minutes (manual) to under 2 minutes (automated).

## §2. Workflow (Step by Step)
First, the agent extracts symptoms and clinical details from the patient's unstructured description
(free text from web form or transcribed phone call). Then, it verifies the patient's insurance
coverage and co-pay amount by querying the insurance provider's API. Then, it classifies the
clinical urgency level (routine, urgent, emergent) using standardized triage rules. Then, it
schedules an appointment with the appropriate provider type (PCP, specialist, ER) based on urgency
and availability. Finally, if the triage classification indicates likely medication need, it
notifies the pharmacy partner to begin pre-authorization — they operate their own system.

## §3. Scale & Performance
- Peak volume: 200 intakes per hour (across all clinic locations)
- Target end-to-end latency: < 30 seconds per intake
- Concurrent users: 25 intake coordinators (internal staff)
- Availability: 99.9% (clinic operating hours: 7am-9pm ET, Mon-Sat)

## §4. Data Sources
- **Patient records:** EMR system (Epic/Cerner) — existing MCP server (emr-mcp) for patient
  lookup, symptom recording, clinical history retrieval
- **Insurance data:** Payer verification API — existing MCP server (insurance-api-mcp) for
  eligibility check, co-pay calculation, coverage verification
- **Scheduling:** Clinic scheduling system — existing MCP server (scheduling-mcp) for
  appointment search, booking, confirmation

## §5. External Partners
- **Pharmacy partner (RxPartner Health):** They operate their own prescription management system.
  When a patient's triage indicates likely medication need, the agent sends a pre-authorization
  request to RxPartner's system with the patient ID, prescribing provider, and ICD-10 code.
  RxPartner returns authorization status and estimated fill time.

## §6. Non-Functional Requirements
- Latency: P95 < 30 seconds end-to-end
- Throughput: 200 intakes/hour sustained
- Availability: 99.9% during clinic hours
- RTO: 15 minutes (critical healthcare workflow)
- RPO: 0 (no data loss — patient records are transactional)
- Data classification: PHI (Protected Health Information) — HIPAA-regulated

## §7. Business Rules (Triage Classification)
IF symptom_severity = "chest_pain" OR symptom_severity = "breathing_difficulty"
  THEN urgency = "emergent" AND provider_type = "ER"

IF symptom_severity = "high_fever" AND duration_days >= 3
  THEN urgency = "urgent" AND provider_type = "PCP_same_day"

IF symptom_severity = "chronic_condition_followup"
  THEN urgency = "routine" AND provider_type = "specialist"

IF insurance_status = "inactive" OR insurance_status = "expired"
  THEN urgency = KEEP_CURRENT AND flag = "uninsured_pathway"

IF urgency IN ("emergent", "urgent") AND medication_likely = true
  THEN notify_pharmacy = true

## §8. Sensitive Data
- PHI: patient names, DOB, SSN (last 4), medical record numbers, ICD-10 codes
- PII: addresses, phone numbers, insurance member IDs
- Encryption: CMEK required for all data at rest, TLS 1.3 for all data in transit
- Access: HIPAA minimum necessary standard — each agent sees only the data it needs

## §9. Acceptance Criteria
1. GIVEN a patient with chest pain symptoms, WHEN intake is submitted, THEN urgency = "emergent"
   AND appointment type = "ER" AND latency < 10 seconds
2. GIVEN a patient with expired insurance, WHEN intake is submitted, THEN flag = "uninsured_pathway"
   AND intake continues (insurance status does not block triage)
3. GIVEN a patient with chronic follow-up + medication likely, WHEN intake completes, THEN
   pharmacy pre-authorization is sent to RxPartner AND authorization status is returned

## §10. Out of Scope
- Prescription writing (done by provider after appointment)
- Billing/claims submission
- Telehealth session scheduling
- Patient portal self-service (this is staff-initiated intake)
