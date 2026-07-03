# spec.md — Retail Order Issue Resolution

## §1. Use Case
Our e-commerce business handles about 8,000 post-purchase support contacts per day. We want a single AI agent that figures out step by step what to check and what to do next.

## §2. Workflow (Step by Step)
One agent handles the whole interaction. It reasons about the customer's message, then decides which single tool to call next based on what it has learned so far.

It keeps looping thought → action → observation until the issue is resolved or it must escalate to a human.

## §2a. Orchestration Style
**ReAct (Reason + Act).** A SINGLE agent loops reason → act → observe with dynamic tool selection.

## §4. Data Sources
- **Order management system:** Cloud SQL — read/write — orders, line items, status, RMA records
- **Inventory service:** read only — existing REST API at inventory-api.internal
- **Payment/refund gateway:** read/write — existing REST API at payments-api.internal

## §5. External Partners
- **Reverse-logistics partner:** They operate their own system. We request a return label / pickup.
- **Shipping carrier tracking:** Public API, no auth — last-mile tracking events.

## §7. Business Rules
IF refund_amount > 200 THEN require human escalation

## §9. Acceptance Criteria
1. GIVEN a shipped order WHEN handled THEN agent dynamically checks carrier tracking
