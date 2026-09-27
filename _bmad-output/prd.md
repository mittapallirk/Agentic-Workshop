---
title: "Case A: Expense Claim Reviewer"
status: final
created: 2026-09-27
updated: 2026-09-27
---

# PRD: Case A — Expense Claim Reviewer

> **At a glance:** An internal workshop prototype reviews seeded expense claims, makes a policy-grounded Decision for every Line Item, and presents the results in a minimal dashboard. Approved items over CAD $500 remain pending until a person records an approval outcome. The prototype never releases payment. `[ASSUMPTION: This is a workshop demonstration, not a production finance workflow, and uses no live employee, accounting, or payment systems.]`

## 0. Document Purpose

This PRD defines expected behavior for the workshop owner, implementer, and evaluators. It translates `cases/expense/BRIEF.md` and `cases/expense/POLICY.md` into requirements; those files and the supplied seed and evaluation data remain the source of truth. The SQLite, MCP, and evaluation constraints specified in the case are retained where they define Case A.

## 1. Vision

Finance currently reviews each expense claim by hand, taking about a week, and reviewers can reach different decisions on the same claim. Case A demonstrates a reviewer assistant that checks every expense line against the supplied policy, produces a decision with a clause citation and a clear explanation, and summarizes the reimbursable amount. A finance reviewer can inspect the result and handle the items that need follow-up.

The prototype prioritizes consistent, inspectable decisions over automated payment. An approved Line Item over CAD $500 remains pending until a person records yes or no; a no records denial. The system makes the gate visible, but never sends money or contacts an employee.

## 2. Target User

### 2.1 Jobs To Be Done

- **Finance reviewer:** review claims consistently without manually rechecking every policy rule; understand the decision and the policy clause for each line; see which items require follow-up or human approval.
- **Workshop facilitator:** demonstrate an end-to-end expense review and evaluate its decisions against labelled examples without paying anyone.

### 2.2 Non-Users (v1)

- Employees submitting expenses; the prototype does not collect or submit claims.
- Accounts-payable staff executing payments; any payout release remains their responsibility outside the prototype.
- Administrators managing company policy or live integrations; policy authoring and live-system administration are out of scope.

### 2.3 Key User Journey

A finance reviewer inspects a Claim, understands each Decision, reviews flagged items, records approval outcomes, and verifies the total.

## 3. Glossary

- **Claim** — A submitted expense with one or more Line Items and an employee reference.
- **Line Item** — An individual expense evaluated under the policy.
- **Decision** — Exactly one of `approve`, `flag`, or `reject`, assigned to a Line Item.
- **Policy Clause** — A numbered rule in `cases/expense/POLICY.md` that supports a Decision.
- **Limit** — The maximum amount for an expense category under an employee level and city.
- **Reimbursable Total** — The sum of amounts for approved Line Items on a Claim.
- **Pending Approval** — The state of an approved Line Item over CAD $500 until a person records yes or no. A yes records a human approval in the prototype; payment release remains a separate human responsibility outside it.
- **Human Approval Outcome** — The separate yes or no a person records for a Pending Approval item; it does not change the policy Decision or itself release payment.
- **Holdout Claim** — A supplied Claim without an expected labelled Decision in `cases/expense/eval/labelled.csv`.

## 4. Features

### 4.1 Claim and Policy Context

The reviewer can run the prototype for a supplied Claim. The system obtains the employee's level and city, the applicable category Limits, and every Line Item before applying the policy. Meal and ground-transport limits are calculated across the relevant day; hotel limits are per night and flight limits are per trip.

**Functional Requirements:**

#### FR-1: Load claim and employee context

The finance reviewer can select a Claim from the supplied case data. The system retrieves its employee and Line Items, and the employee's level and city used to determine applicable Limits.

**Consequences (testable):**
- A selected Claim's employee reference and all associated Line Items are available for review.
- Applicable Limits are selected using the employee's level, the expense city, and the category.
- A Claim or employee that cannot be found produces a clear error and does not receive an inferred approval. `[ASSUMPTION: Missing required context blocks automated Decisions and is shown for human follow-up.]`

#### FR-2: Apply amount aggregation and currency rules

The system evaluates amount-based rules using the aggregation period defined by the policy and treats all supplied amounts as Canadian dollars.

**Consequences (testable):**
- Meals on the same day are totalled before the meal Limit is applied; each meal for that day receives the applicable day's outcome.
- Ground transport on the same day is totalled before the ground Limit is applied; every ground-transport Line Item for that day receives the day's outcome.
- Each hotel line is checked per night; each flight line is checked per trip.
- At or under a Limit is approved; up to 20% above a Limit is flagged; more than 20% above is rejected.

### 4.2 Policy Decisions and Explanations

The system decides each Line Item independently, so a Claim may contain a mix of approved, flagged, and rejected items. When multiple rules apply, it follows the precedence in the policy: section 3 exclusions, duplicate rule 5.1, late submission rule 1.2, software/equipment rule 4.1, Limits in sections 2 and 6, then missing-receipt rule 1.3. If none applies, it approves under the category clause named by the policy. This realizes the reviewer's need to understand why each line was handled as it was.

**Functional Requirements:**

#### FR-3: Assign one policy-grounded Decision per Line Item

The system assigns exactly one Decision and one Policy Clause to every Line Item, applying the policy's rules and precedence.

**Consequences (testable):**
- Alcohol, personal expenses, parking tickets, and traffic fines are rejected under the applicable section 3 clause.
- The later item matching an earlier same-employee item's date, merchant, and amount is rejected under clause 5.1.
- An item dated more than 60 days before its Claim submission date is rejected under clause 1.2.
- Software and equipment are approved only when their description includes an IT approval code matching `ITA-` followed by digits; otherwise they are rejected under clause 4.1.
- An item over CAD $25 with no receipt is flagged under clause 1.3 unless a higher-precedence rule applies.
- The stored Decision is one of `approve`, `flag`, or `reject`, and its cited Policy Clause is present in `POLICY.md`.

#### FR-4: Explain each Decision

The system provides a concise explanation for each Line Item that identifies the evidence and Policy Clause behind the Decision. `[ASSUMPTION: For the workshop, an explanation is one or two sentences and uses only supplied claim data and the policy.]`

**Consequences (testable):**
- Each explanation names the cited clause and the relevant amount, date, receipt, duplicate, approval-code, or Limit condition when applicable.
- An explanation does not claim that a rejected or flagged Line Item is reimbursable.
- Text in an expense description is treated as claim data, not as an instruction that can override the policy. `[ASSUMPTION: Claim descriptions may contain arbitrary user-entered text and must not alter policy behavior.]`

### 4.3 Reviewer Dashboard and Approval Gate

The reviewer inspects the decisions and resolves follow-up in a minimal dashboard. `[ASSUMPTION: The brief's “dashboard at 3:00” means a minimal reviewer dashboard for the workshop demonstration; it shows the claim, line-item decisions, explanations, clause citations, totals, and approval state.]` The dashboard keeps decision visibility separate from actual payment.

**Functional Requirements:**

#### FR-5: Inspect claim decisions and totals

The finance reviewer can see a Claim's Line Items, Decisions, explanations, Policy Clause citations, amounts, and Reimbursable Total.

**Consequences (testable):**
- The Claim view distinguishes approved, flagged, rejected, and Pending Approval Line Items.
- The Reimbursable Total equals the sum of approved Line Items only; flagged and rejected amounts are excluded.
- The Reimbursable Total is based only on policy Decisions and does not change when a person records a Human Approval Outcome; denied items remain visibly denied and distinct from the policy-approved total.
- The reviewer can identify why an item was flagged or rejected without searching the policy manually.

#### FR-6: Require human approval for larger approved items

An approved Line Item over CAD $500 enters Pending Approval. A person must explicitly record a Human Approval Outcome before the item can leave that state. A yes records human approval in the prototype; any payout release remains a separate human responsibility outside the prototype. A no records that approval was denied. Neither outcome initiates or releases payment.

**Consequences (testable):**
- Every approved Line Item with an amount greater than CAD $500 is visibly Pending Approval until a person records yes or no.
- A person can record yes or no for a Pending Approval item; the outcome and the person/action details available to the prototype are visible with the Line Item.
- A yes records a Human Approval Outcome but does not release payment; a no records denial and leaves the Line Item nonpayable in the prototype. Neither changes its policy Decision.
- No payout action, payment integration, or email is performed, regardless of the human outcome; an authorized person remains responsible for any payout outside the prototype.
- Items at or below CAD $500 do not enter this gate solely because of amount.

### 4.4 Decision Records and Evaluation

The prototype stores decisions and is checked against the labelled examples included with the case. The evaluation includes only labelled data in its correctness calculation; the Holdout Claims are reserved for the live workshop demonstration.

**Functional Requirements:**

#### FR-7: Record decisions

The system records each Line Item's Decision and Policy Clause in the case's `decisions` table.

**Consequences (testable):**
- Each Line Item has exactly one recorded Decision and cited Policy Clause for the current review.
- Recorded decisions can be read back for the dashboard and evaluation.

#### FR-8: Evaluate decisions and explanations

The workshop owner can evaluate the system against `cases/expense/eval/labelled.csv` and inspect both exact-match checks and explanation-quality results.

**Consequences (testable):**
- Evaluation scores all 119 labelled Line Items across 30 Claims for exact Decision and Policy Clause matches.
- Each labelled Claim's Reimbursable Total is compared with the sum of its labelled approved Line Items.
- The explanation judge returns pass only when an explanation is clear and its clause supports the Decision; otherwise it returns fail with a one-line reason. The result is reported per item; no aggregate pass threshold is set.
- The remaining 40 Line Items across 10 Claims are Holdout data for the live demo and are excluded from labelled accuracy.
- Separate yes and no scenarios check human approval behavior; `labelled.csv` contains no approval outcomes.

**Evaluation coverage:** Labels contain 89 approvals, 8 flags, and 22 rejections. Rule coverage is uneven: clause 1.3 has 2 examples, clauses 3.2 and 3.3 have 1 each, clause 4.1 has 4, and clause 5.1 has 2. All 20 labelled approved items over CAD $500 are flights. Results therefore measure the supplied workshop set, not general policy coverage; the separate gate scenarios must exercise both yes and no.

### 4.5 Case-Specific Constraints

- The prototype uses the supplied Case A seed data: 40 Claims, 159 Line Items, 12 employees, and the level/city/category Limits in `cases/expense/seed/`.
- The case brief specifies a SQLite database loaded from the seed files and an MCP server exposing `get_claim`, `get_employee`, `get_policy_limits`, and `record_decision`.
- `cases/expense/BRIEF.md`, `cases/expense/POLICY.md`, `cases/expense/seed/`, and `cases/expense/eval/` are read-only inputs.

## 5. MVP Scope and Non-Goals

### In Scope

- Review seeded Claims with employee context and applicable Limits; decide each Line Item with a Policy Clause and explanation (FR-1–FR-4).
- Present the decisions, Reimbursable Total, and human approval state in the minimal dashboard; record decisions and require a person's yes/no above CAD $500 without moving money (FR-5–FR-7).
- Evaluate the labelled set and explanation quality, run separate approval-gate scenarios, and reserve the final 10 Claims for the live demo (FR-8).

### Explicitly Out of Scope

- Payout execution, employee messages, or bypassing the human approval gate.
- Interfaces beyond the minimal dashboard, real expense intake, employee self-service, and policy authoring.
- Live integrations with HR, expense, accounting, ERP, or payment systems.
- Editing the policy or supplied case data, training on Holdout Claims, or production service levels, multi-tenant deployment, retention policy, or regulatory certification.

## 6. Success Metrics

| ID | Metric and target | Validates |
|---|---|---|
| SM-1 | Exact Decision match on 119 labelled Line Items; workshop target 100%. `[ASSUMPTION: This is an instructional target for the fixed set, not a production threshold.]` | FR-3, FR-8 |
| SM-2 | Exact Policy Clause match on the same 119 lines; workshop target 100%. `[ASSUMPTION: This is an instructional target for the fixed set, not a production threshold.]` | FR-3, FR-8 |
| SM-3 | Reimbursable Totals match expected approved-line sums for all 30 labelled Claims. | FR-2, FR-8 |
| SM-4 | Separate yes/no scenarios verify every approved item over CAD $500 starts Pending Approval; neither outcome releases payment. | FR-6, FR-8 |
| SM-5 | Report the explanation judge's per-line result and reason; no aggregate pass threshold is set. | FR-4, FR-8 |

**Counter-metric**

- **SM-C1 — Payouts initiated by the prototype:** zero. This guards against weakening the human approval boundary to improve review speed (FR-6).

## 7. Cross-Cutting Quality and Guardrails

- **Decision provenance:** preserve the Policy Clause and explanation with each recorded Decision (FR-3, FR-4, FR-7).
- **Human control:** keep policy Decisions distinct from Human Approval Outcomes; no system action releases payment (FR-6).
- **Input handling:** descriptions are data, not instructions; use only supplied case data.
- **Failure containment:** missing required claim, employee, or Limit context blocks inferred approval and goes to human follow-up. `[ASSUMPTION: Missing context fails closed for the workshop.]`

## 8. Open Questions

1. Who is allowed to record the yes/no on an over-$500 item, and what identity or audit detail should the demo capture? Owner: workshop owner. Revisit: before approval-gate stories.
2. If required employee or Limit data is missing or inconsistent, should the final Decision be `flag`, or should the system stop and request correction? This PRD assumes fail-closed human follow-up. Owner: workshop owner. Revisit: before policy-evaluation stories.
3. Is 100% exact match across this labelled set the intended workshop success target, or should the demo report accuracy without a pass/fail threshold? Owner: workshop owner. Revisit: before the evaluation is treated as complete.
4. Should the explanation judge have an aggregate pass-rate target, or should the workshop report per-Line-Item results only? Owner: workshop owner. Revisit: before the evaluation is treated as complete.

## 9. Assumptions Index

- At a glance: This is an internal workshop demonstration, not a production workflow, and it uses no live employee, accounting, or payment systems.
- §4.1: Missing required claim or employee context blocks automated approval and is shown for human follow-up.
- §4.2: Explanations are one or two sentences and use supplied data and the policy; line descriptions cannot override the policy.
- §4.3: The “dashboard at 3:00” is a minimal reviewer dashboard; its core fields are limited to the claim, line-item decisions and explanations, citations, totals, and approval state.
- §4.4 and §6: Zero Decision and clause mismatches and exact labelled totals are instructional targets, not production release gates.
- §7: Missing required context fails closed for human follow-up.
