# Policy reconciliation

## Policy extract checked

- **Decision unit and currency:** Every line item receives exactly one of approve, flag, or reject, plus its deciding clause. Items are evaluated independently; one claim may mix outcomes. All amounts are CAD.
- **General rules:** Reject an item dated more than 60 days before claim submission (1.2). Flag an item over $25 without a receipt (1.3).
- **Limits:** Limits are determined by employee level (L1–L4), expense city, and category in `seed/limits.csv`. Meals aggregate by day and every meal that day receives the day's outcome (2.1). Hotels are per night, one line per night (2.2). Flights are per trip, one line per trip (2.3). Ground transport (taxi, rideshare, train) aggregates by day under the `ground` limit, like meals (6.1). At/under limit = approve; over by at most 20% = flag; more than 20% over = reject (2.4).
- **Absolute exclusions:** Reject alcohol (3.1), personal gym/entertainment/clothing expenses (3.2), and parking tickets/traffic fines (3.3).
- **Software/equipment:** Approve only if description contains an IT approval code matching `ITA-` followed by digits; otherwise reject (4.1).
- **Duplicates:** Reject the later item when an earlier item from the same employee has the same date, merchant, and amount (5.1).
- **Precedence:** Section 3 exclusions, then duplicate 5.1, late 1.2, software/equipment 4.1, limits in sections 2 and 6, then missing receipt 1.3. If none applies, approve under the category clause (2.1 meals, 2.2 hotels, 2.3 flights, 6.1 ground transport).
- **Human approval:** An otherwise-approved item over $500 is paid only after a person says yes. The agent records the decision; a person releases payout (7.1).

## Gaps or constraints in PRD

1. **Ground daily outcome assignment is implicit.** FR-2 says ground items are summed by day before applying the limit, but unlike meals it does not explicitly require each ground-transport line that day to receive the day's outcome. Add that criterion to preserve 6.1 and the one-decision-per-line rule.
2. **Payout-release step is constrained away.** Policy 7.1 says a person releases payout after saying yes. FR-6 says the prototype never initiates/releases payment and no payout action is performed. That is a deliberate prototype scope reduction, but the PRD should make clear that any downstream payout release remains a human responsibility outside the prototype; a yes recorded in the demo is not a release.

## Disposition

FR-2 now states that every ground-transport line receives the day's aggregate outcome. FR-5 and FR-6 make clear that human approval outcomes do not change the policy Decision or Reimbursable Total, and that a recorded yes does not release payment. No policy gaps remain open.
