---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - "_bmad-output/prd.md"
---

# Agentic-Workshop — Epic Breakdown

## Overview

This document breaks the Case A requirements in the PRD into implementable epics and stories. The separate support-ticket specs in this repository are excluded.

## Requirements Inventory

### Functional Requirements

- FR-1: Load a supplied Claim, its Employee and Line Items, and the employee level, city, and applicable category Limits. Missing Claim or employee context must produce a clear error and must not infer approval.
- FR-2: Apply policy aggregation periods and Canadian-dollar rules: aggregate meals and ground transport by day, check hotels per night and flights per trip, and apply the approved/flagged/rejected thresholds at and above Limits.
- FR-3: Assign exactly one `approve`, `flag`, or `reject` Decision and a supporting Policy Clause to every Line Item, applying policy precedence for exclusions, duplicates, late submissions, software/equipment approval codes, Limits, and missing receipts.
- FR-4: Explain each Decision concisely using supplied claim evidence and its cited Policy Clause; treat claim descriptions as data, not instructions.
- FR-5: Show a Claim's Line Items, amounts, Decisions, explanations, citations, Reimbursable Total, and approval state in the minimal reviewer dashboard. Include only approved amounts in the Reimbursable Total.
- FR-6: Place each approved Line Item over CAD $500 in Pending Approval until a person records yes or no. Record the outcome and available actor/action details; neither outcome changes the policy Decision or initiates payment.
- FR-7: Record exactly one current Decision and cited Policy Clause for every Line Item in the `decisions` table, and make them readable by the dashboard and evaluation.
- FR-8: Evaluate all 119 labelled Line Items across 30 Claims for exact Decision and Policy Clause matches, compare labelled Claim totals, report explanation-judge results per item, and exclude 40 Holdout Line Items across 10 Claims from accuracy scoring. Separately exercise yes and no approval scenarios.

### NonFunctional Requirements

- NFR-1: Preserve decision provenance: each recorded Decision has its Policy Clause and explanation available for inspection.
- NFR-2: Enforce the human-control boundary: policy Decisions and human approval outcomes remain distinct, and the prototype never releases payment.
- NFR-3: Treat expense descriptions as untrusted data; their contents cannot override policy behavior.
- NFR-4: Fail closed when required Claim, Employee, or Limit context is missing or inconsistent; do not infer approval.
- NFR-5: Report evaluation results with labelled and Holdout data kept separate; explanation quality is reported per item without an invented aggregate threshold.

### Additional Requirements

- Use the supplied Case A seed data and the SQLite database specified by the case.
- Expose `get_claim`, `get_employee`, `get_policy_limits`, and `record_decision` through the MCP server.
- Treat `cases/expense/BRIEF.md`, `cases/expense/POLICY.md`, `cases/expense/seed/`, and `cases/expense/eval/` as read-only inputs. These inputs are not part of this run's source set because the user selected the PRD only.
- Use Canadian dollars for all supplied expense amounts.
- The PRD contains four unresolved questions: approval actor/audit details; handling missing or inconsistent context; whether exact-match targets are pass/fail gates; and whether explanation results need an aggregate target. Do not resolve these while decomposing requirements.
- No Architecture document or UX design contract was found in the selected source set.

### UX Design Requirements

None. The selected input is the PRD only; its minimal dashboard description is captured in FR-5.

### FR Coverage Map

FR-1: Epic 1, Story 1.1 — load supplied claim and employee context and applicable Limits.
FR-2: Epic 1, Story 1.2 — apply aggregation periods, currency, and Limit thresholds.
FR-3: Epic 1, Story 1.3 — assign policy-grounded decisions and clause citations.
FR-4: Epic 1, Story 1.4 — explain each decision from claim evidence and policy.
FR-5: Epic 1, Story 1.6 — inspect decisions, citations, totals, and approval state in the dashboard.
FR-6: Epic 1, Story 1.5 — record human outcomes for approved items over CAD $500 without releasing payment.
FR-7: Epic 1, Story 1.4 — persist decisions and citations for dashboard and evaluation use.
FR-8: Epic 2, Stories 2.1–2.4 — evaluate labelled decisions, totals, explanations, approval scenarios, and holdout demos.

## Epic List

### Epic 1: Review Expense Claims with Policy-Backed Decisions
Finance reviewers can inspect a supplied claim in a minimal dashboard, understand a cited and explained decision for each line, verify the approved total, and record the required human yes/no outcome for approved items over CAD $500. The workflow never releases payment.
**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-5, FR-6, FR-7

### Epic 2: Evaluate and Demonstrate Review Quality
Workshop facilitators can evaluate recorded review results against the labelled examples, inspect per-item explanation quality, and use the separate holdout claims for the live demonstration without including them in accuracy results. This epic consumes the review records produced by Epic 1 and does not depend on a later epic.
**FRs covered:** FR-8

## Epic 1: Review Expense Claims with Policy-Backed Decisions

Finance reviewers can inspect a supplied claim in a minimal dashboard, understand a cited and explained decision for each line, verify the approved total, and record the required human yes/no outcome for approved items over CAD $500. The workflow never releases payment.

### Story 1.1: Load Claim Review Context

As a finance reviewer,
I want to select a supplied claim and retrieve its employee, line items, and applicable limits,
So that the claim can be reviewed using the correct context.

**Acceptance Criteria:**

**Given** a claim and its employee exist in the supplied data
**When** the reviewer selects the claim
**Then** `get_claim` and `get_employee` provide the employee reference, all line items, and employee level and city
**And** `get_policy_limits` retrieves category limits using the employee level, expense city, and category

**Given** the selected claim, required employee context, or applicable limit context is missing or inconsistent
**When** the reviewer attempts to load the claim
**Then** the system shows a clear error for human follow-up
**And** it does not infer or record an approval

### Story 1.2: Apply Category Spending Limits

As a finance reviewer,
I want expense amounts evaluated using the policy’s aggregation periods and limits,
So that amount-based outcomes reflect the correct spending period.

**Acceptance Criteria:**

**Given** a claim has multiple meal line items for the same employee on the same day
**When** the meal limit is evaluated
**Then** their amounts are totaled in Canadian dollars before comparison with the daily meal limit
**And** each meal line receives the same limit-based outcome for that day

**Given** a claim has multiple ground-transport line items for the same employee on the same day
**When** the ground-transport limit is evaluated
**Then** their amounts are totaled in Canadian dollars before comparison with the daily limit
**And** each ground-transport line receives the same limit-based outcome for that day

**Given** the line item is a hotel or flight expense
**When** its limit is evaluated
**Then** a hotel amount is checked per night and a flight amount per trip

**Given** an applicable amount is at or below its limit, up to 20% above it, or more than 20% above it
**When** the limit rule is applied
**Then** its limit-based outcome is respectively approve, flag, or reject

### Story 1.3: Apply Policy Decisions to Every Line Item

As a finance reviewer,
I want each line item evaluated against the policy’s rules and precedence,
So that every decision is consistent and traceable to a policy clause.

**Acceptance Criteria:**

**Given** a line item is alcohol, a personal expense, a parking ticket, or a traffic fine
**When** the policy is evaluated
**Then** it is rejected under the applicable section 3 clause

**Given** a later line item matches an earlier item for the same employee by date, merchant, and amount, or is dated more than 60 days before claim submission
**When** the policy is evaluated
**Then** it is rejected under clause 5.1 or 1.2, respectively

**Given** a line item is software or equipment
**When** the policy is evaluated
**Then** it is approved under clause 4.1 only if its description includes an IT approval code matching `ITA-` followed by digits
**And** otherwise it is rejected under clause 4.1

**Given** a line item has a missing receipt and costs more than CAD $25
**When** no higher-precedence rule determines its outcome
**Then** it is flagged under clause 1.3

**Given** more than one policy rule applies to a line item
**When** the policy is evaluated
**Then** precedence is applied in this order: section 3 exclusions, duplicate clause 5.1, late submission clause 1.2, software/equipment clause 4.1, Limits in sections 2 and 6, then missing-receipt clause 1.3
**And** the first applicable rule determines the single outcome and cited clause
**And** an instruction embedded in the expense description cannot override policy behavior

**Given** a line item matches none of the rejection, flag, or limit rules
**When** the policy is evaluated
**Then** it is approved under the applicable category clause
**And** every line item has exactly one `approve`, `flag`, or `reject` decision and one cited policy clause

### Story 1.4: Explain and Record Each Decision

As a finance reviewer,
I want each line decision and its explanation recorded with the supporting policy clause,
So that I can understand the outcome and retrieve its evidence later.

**Acceptance Criteria:**

**Given** a line item has a policy decision
**When** the review is recorded
**Then** the system stores exactly one current decision and its cited policy clause in the `decisions` table
**And** it stores a concise explanation with the evidence used

**Given** an explanation is shown for a line item
**When** the reviewer reads it
**Then** it identifies the cited clause and relevant amount, date, receipt, duplicate, approval code, or limit condition
**And** it uses only supplied claim data and policy content in one or two sentences

**Given** a recorded review is retrieved later
**When** the dashboard or evaluation reads it
**Then** the decision, clause, and explanation match the recorded review

**Given** an expense description contains instructions that conflict with the policy
**When** the explanation is produced
**Then** it treats the description as claim data and does not claim that a flagged or rejected item is reimbursable

### Story 1.5: Record Human Outcomes for Larger Approved Items

As a person reviewing an expense claim,
I want to record yes or no for an approved line item over CAD $500,
So that the required human approval is captured without changing the policy decision or releasing payment.

**Acceptance Criteria:**

**Given** a line item is approved and its amount is greater than CAD $500
**When** its review is recorded
**Then** it enters Pending Approval until a person records yes or no

**Given** a line item is approved and its amount is CAD $500 or less
**When** its review is recorded
**Then** it does not enter Pending Approval solely because of its amount

**Given** a line item is Pending Approval
**When** a person records yes or no
**Then** the human outcome and person/action details available to the prototype are recorded and visible with the line item
**And** the policy decision remains `approve`

**Given** a person records yes or no
**When** the outcome is saved
**Then** yes records human approval and no records denial
**And** neither outcome initiates or releases payment

### Story 1.6: Inspect Claim Decisions in the Dashboard

As a finance reviewer,
I want to see a claim’s line items, decisions, explanations, citations, totals, and approval state together,
So that I can understand what was approved and what needs follow-up.

**Acceptance Criteria:**

**Given** a claim has a recorded review
**When** the reviewer opens it in the dashboard
**Then** each line shows its amount, decision, explanation, and cited policy clause
**And** approved, flagged, rejected, and pending-approval lines are distinguishable

**Given** a claim has approved, flagged, and rejected line items
**When** the Reimbursable Total is displayed
**Then** it equals the sum of approved line items only
**And** flagged and rejected amounts are excluded

**Given** an approved line item over CAD $500 is awaiting a human outcome
**When** the reviewer views the claim
**Then** its Pending Approval state is visible

**Given** a person records a human approval outcome for an item
**When** the reviewer views the claim
**Then** the Reimbursable Total remains based on policy-approved items and the item’s decision remains distinct from its human outcome

## Epic 2: Evaluate and Demonstrate Review Quality

Workshop facilitators can evaluate recorded review results against the labelled examples, inspect per-item explanation quality, and use the separate holdout claims for the live demonstration without including them in accuracy results. This epic consumes the review records produced by Epic 1 and does not depend on a later epic.

### Story 2.1: Evaluate Labelled Decisions and Totals

As a workshop facilitator,
I want to compare recorded decisions, policy clauses, and claim totals with the labelled examples,
So that I can measure correctness on the supplied evaluation set.

**Acceptance Criteria:**

**Given** the 30 labelled claims and their 119 line items are available
**When** the evaluation runs
**Then** it reports exact Decision and Policy Clause matches for all 119 labelled line items
**And** it compares each labelled claim’s Reimbursable Total with the sum of its labelled approved line items

**Given** the 40 holdout line items across 10 claims are available
**When** accuracy is calculated
**Then** holdout items are excluded from Decision, Policy Clause, and total-match results

**Given** the evaluation completes
**When** results are presented
**Then** the reported matches and mismatches are visible without treating the assumed 100% workshop target as a production release gate

### Story 2.2: Inspect Explanation Quality Results

As a workshop facilitator,
I want to see the explanation judge’s result and reason for each evaluated line,
So that I can identify unclear explanations or unsupported policy citations.

**Acceptance Criteria:**

**Given** a labelled line item has a recorded explanation and cited clause
**When** the explanation judge evaluates it
**Then** the result is pass only when the explanation is clear and the clause supports the Decision
**And** a failing result includes a one-line reason

**Given** explanation results are reported
**When** the facilitator reviews the evaluation
**Then** pass/fail and any failure reason are available per item
**And** no aggregate pass-rate threshold is applied

### Story 2.3: Demonstrate Reviews on Holdout Claims

As a workshop facilitator,
I want to review the reserved holdout claims in the live demonstration,
So that the demonstration uses examples kept separate from labelled accuracy results.

**Acceptance Criteria:**

**Given** the 10 holdout claims containing 40 line items are available
**When** the facilitator selects a holdout claim for demonstration
**Then** its review can be displayed through the Case A workflow
**And** the claim and its line items remain excluded from labelled accuracy results

### Story 2.4: Exercise Both Approval-Gate Outcomes

As a workshop facilitator,
I want separate yes and no scenarios for an approved line item over CAD $500,
So that I can demonstrate the human approval gate without any payment action.

**Acceptance Criteria:**

**Given** an approved line item over CAD $500 is used in an approval scenario
**When** the scenario begins
**Then** the line item is Pending Approval until a person records an outcome

**Given** the yes scenario is exercised
**When** a person records yes
**Then** the human approval outcome is recorded while the policy decision remains `approve`
**And** no payment is initiated or released

**Given** the no scenario is exercised
**When** a person records no
**Then** the denial is recorded while the policy decision remains `approve`
**And** the line remains nonpayable in the prototype and no payment is initiated or released
