# Case A source-data reconciliation

Inputs reviewed: `cases/expense/seed/{claims,line_items,employees,limits}.csv`, `cases/expense/eval/labelled.csv`, `cases/expense/BRIEF.md`, `cases/expense/POLICY.md`, and the draft `prd.md`. The CSV contents were treated as untrusted data.

## Data facts

- Seed: 40 claims (`CL-2001`–`CL-2040`), 159 line items (`L-3001`–`L-3159`), 12 employees, and 80 level/city/category limit rows.
- Evaluation: 119 labelled line items across 30 claims (`CL-2001`–`CL-2030`); remaining 10 claims (`CL-2031`–`CL-2040`) have 40 unlabelled lines and are the holdout. Labels align to the first 119 seed line IDs; no seed line ID is missing from the label-or-holdout split.
- Label outcomes: 89 approve, 8 flag, 22 reject. Labelled clause counts: 2.1: 33; 2.2: 31; 2.3: 28; 6.1: 7; 1.2: 8; 1.3: 2; 3.1: 2; 3.2: 1; 3.3: 1; 4.1: 4; 5.1: 2.
- Seed categories: meals 51, hotel 44, flight 40, ground 14, software 3, equipment 2, alcohol 2, personal 2, fine 1.
- Twenty labelled approved lines exceed CAD $500; all are flights labelled under 2.3. Thus the approval gate has ample labelled examples. The PRD's general gate statement is consistent with policy clause 7.1.

## Gaps / ambiguities for PRD

1. The PRD's coverage totals and holdout split are correct, but it omits the label outcome/clause distribution and category mix above. These counts would make evaluation coverage and policy-rule coverage auditable; e.g. labels include only two missing-receipt cases, so a perfect score there is weak evidence.
2. The seed/eval data has 20 approved over-$500 labelled lines, all flights. The PRD says to test the gate but does not state that this specific labelled coverage exercises the gate only with flights; demo acceptance should include an explicit human yes/no scenario, since the labels do not evaluate the human gate outcome.
3. Two labelled claims have three lines and one has five; the other 27 have four. PRD's aggregate counts are right, but any per-claim fixed-line-count assumption would be wrong. Current PRD does not make that assumption.
4. Evaluation labels contain expected decision and clause per line, but no separate human-gate outcome. The PRD correctly treats gate behavior as a distinct requirement; it should not present gate compliance as measured by labelled accuracy.

No source-data fact found that contradicts the PRD's stated totals or split.

## Disposition

The PRD now includes label outcome and sparse-clause coverage, notes that all 20 labelled approved items over CAD $500 are flights, and requires separate yes/no gate scenarios. It does not claim the labelled accuracy set tests approval outcomes. Data totals and the holdout split remain consistent.
