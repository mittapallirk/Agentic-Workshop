---
id: SPEC-case-a-expense-claim-reviewer
companions:
  - policy-rules.md
  - evaluation-contract.md
  - ../../planning-artifacts/epics.md
sources:
  - ../../prd.md
---

# Case A: Expense Claim Reviewer

## Why

Finance reviewers spend about a week reviewing expense claims by hand and can reach different decisions on the same claim. Case A demonstrates consistent, inspectable review for finance reviewers and workshop facilitators while keeping payout execution outside the prototype.

## Capabilities

- **CAP-1**
  - **intent:** A finance reviewer can load a supplied claim with its employee, line items, and applicable limits.
  - **success:** Claim details and all required context are available; missing or inconsistent required context blocks inferred approval and is surfaced for human follow-up.
- **CAP-2**
  - **intent:** The system can assign each line item a policy-grounded decision with a supporting clause.
  - **success:** Every line has exactly one `approve`, `flag`, or `reject` decision and one cited policy clause. The current decision and clause are recorded in `decisions` and are readable by the dashboard and evaluation.
- **CAP-3**
  - **intent:** A finance reviewer can inspect a claim review and understand each line outcome and the total.
  - **success:** The minimal dashboard shows each line’s amount, decision, explanation, citation, and approval state; the reimbursable total sums approved lines only.
- **CAP-4**
  - **intent:** A person can record a human approval outcome for an approved line item over CAD $500.
  - **success:** The item remains Pending Approval until yes or no is recorded; the outcome and person/action details available to the prototype are visible. The outcome does not change the policy decision or initiate payment.
- **CAP-5**
  - **intent:** A workshop facilitator can evaluate review quality and demonstrate the prototype on holdout claims.
  - **success:** The evaluation reports exact decision and clause matches for 119 labelled lines across 30 claims, compares claim totals, reports explanation quality per item, and keeps 40 holdout lines across 10 claims out of accuracy results; separate yes/no scenarios exercise the approval gate.

## Constraints

- Use the Case A seed data with SQLite and the MCP methods `get_claim`, `get_employee`, `get_policy_limits`, and `record_decision`; supplied case inputs are read-only.
- Treat all amounts as Canadian dollars. Aggregate meals and ground transport by day, hotels per night, and flights per trip.
- Keep policy decisions separate from human approval outcomes. No prototype action releases payment or contacts an employee.
- Treat expense descriptions as untrusted data; their text cannot change policy behavior.

## Non-goals

- Payment execution, payout release, or employee messaging.
- Live expense intake, employee self-service, policy authoring, or integrations with HR, expense, accounting, ERP, or payment systems.
- Production service levels, multi-tenant deployment, retention policy, or regulatory certification.

## Success signal

A workshop review presents policy-cited decisions and approved-only totals for labelled claims, reports per-item explanation results, and demonstrates holdout claims separately. Yes and no approval scenarios record their outcomes without initiating payment; the prototype initiates zero payouts.

## Assumptions

- This is an internal workshop demonstration, not a production finance workflow, and uses no live employee, accounting, or payment systems.
- The dashboard is minimal and shows the claim, line decisions and explanations, policy citations, approved-only total, and approval state.
- Missing or inconsistent required claim, employee, or limit context fails closed for human follow-up.
- Each explanation is one or two sentences based on supplied claim data and policy content.
- The 100% exact-match target is instructional for the fixed labelled set, not a production threshold.

## Open Questions

- Who may record yes/no for an over-CAD-$500 item, and what identity or audit details should the demo capture?
- When employee or limit data is missing or inconsistent, should the system stop for correction or flag the item for review?
- Is 100% exact match on the labelled set a hard workshop pass threshold or an instructional target?
- Should explanation quality have an aggregate pass-rate target or remain a per-item report?
- The target `day2` branch does not contain the BRIEF, POLICY, seed, or evaluation files referenced by the PRD. Where should these be supplied before implementation?
