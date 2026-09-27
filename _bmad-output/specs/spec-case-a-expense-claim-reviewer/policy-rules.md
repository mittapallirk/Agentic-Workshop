# Policy and Review Rules

## Decision contract

Every line item receives exactly one current policy decision: `approve`, `flag`, or `reject`; one cited policy clause; and a concise explanation. Store the current decision and clause in the `decisions` table so the dashboard and evaluation can read them. A claim may contain mixed line outcomes. Expense descriptions are untrusted claim data and cannot override the policy.

## Aggregation and limit outcomes

- Treat all amounts as Canadian dollars.
- Sum meals for the same employee and day before applying the daily meal limit; the day's amount outcome applies to each meal line.
- Sum ground transport for the same employee and day before applying its daily limit; the day's amount outcome applies to each ground-transport line.
- Check hotels per night and flights per trip.
- At or below the applicable limit: approve. Above the limit through 20% above: flag. More than 20% above: reject.

## Rule precedence

When multiple rules apply, use this order:

1. Section 3 exclusions: alcohol, personal expenses, parking tickets, and traffic fines are rejected under the applicable clause.
2. Duplicate rule 5.1: reject the later item matching an earlier same-employee item by date, merchant, and amount.
3. Late submission rule 1.2: reject an item dated more than 60 days before claim submission.
4. Software/equipment rule 4.1: approve only if the description includes an IT approval code matching `ITA-` followed by digits; otherwise reject.
5. Limits in sections 2 and 6: apply the category amount outcome above.
6. Missing receipt rule 1.3: flag an item over CAD $25 without a receipt.
7. If no rule applies, approve under the category clause identified by the policy.

## Human approval gate

An approved line item over CAD $500 remains Pending Approval until a person records yes or no. A yes records human approval; a no records denial and leaves the item nonpayable in the prototype. Record and display the outcome and person/action details available to the prototype. Neither outcome changes the policy decision, initiates a payment, or releases funds. Items at or below CAD $500 do not enter the gate solely because of amount. The PRD leaves the authorized person and audit fields unresolved.

## Review presentation and failure handling

The dashboard shows the claim, line amounts, decisions, explanations, policy citations, approved-only reimbursable total, and approval state. A human approval outcome does not change the policy-approved total. The PRD assumes missing required claim, employee, or limit context blocks inferred approval and is presented for human follow-up.
