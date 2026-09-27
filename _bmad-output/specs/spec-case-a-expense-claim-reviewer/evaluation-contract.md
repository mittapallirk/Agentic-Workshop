# Evaluation Contract

## Data sets

- Supplied Case A seed: 40 claims, 159 line items, and 12 employees. The SQLite data and source files are read-only.
- Labelled evaluation: 30 claims and 119 line items with 89 approvals, 8 flags, and 22 rejections.
- Holdout demonstration: 10 claims and 40 line items, reserved for the live workshop demo and excluded from accuracy scoring.

## Required reports

- Report exact decision and policy-clause matches across all 119 labelled line items.
- Compare each of the 30 labelled claim reimbursable totals with the sum of its labelled approved lines.
- Run the explanation judge per labelled item. Pass only when the explanation is clear and its clause supports the decision; otherwise return fail and a one-line reason. No aggregate pass-rate threshold is set.
- Keep the 40 holdout line items out of labelled accuracy results and use them only for the live demo.
- Exercise separate yes and no scenarios for an approved item over CAD $500. Neither scenario initiates or releases payment.

## Interpretation limits

The PRD sets 100% exact decision and clause match as an instructional target for the fixed workshop set, not as a production release threshold; whether it should be a hard pass/fail criterion remains open. Coverage is uneven: clause 1.3 has 2 labelled examples; clauses 3.2 and 3.3 have 1 each; clause 4.1 has 4; and clause 5.1 has 2. All 20 labelled approved items over CAD $500 are flights. Results describe this workshop sample, not general policy coverage; the yes/no gate scenarios provide separate coverage.
