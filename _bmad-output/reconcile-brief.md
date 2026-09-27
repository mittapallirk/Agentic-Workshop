# Case A brief reconciliation

**Inputs compared:** `cases/expense/BRIEF.md` and `_bmad-output/planning-artifacts/prds/prd-Agentic-Workshop-2026-09-27/prd.md`. No `addendum.md` is present.

## Material gaps / decisions

1. **Dashboard at 3:00 — intentionally unresolved by the brief.** The PRD marks a minimal reviewer dashboard and its contents as an assumption, aligned with the user's direction to assume a minimal dashboard. The brief itself specifies only that dashboard as the interface boundary; it does not specify a display format or fields. The PRD's claim, line decisions, explanations, clause citations, totals, and approval state are a reasonable elaboration, but should remain labeled as an assumption until confirmed.
2. **Approval gate — requirement preserved, with one elaboration.** The brief requires every approved item over $500 to wait for a person's yes before payment. The PRD preserves the threshold, pending state, and prohibition on prototype payouts, and makes the pending state visible. It additionally specifies that a person may record either yes or no. That no option is not explicit in the brief, but is consistent with the human gate. Keep clear that a policy `approve` decision is not sufficient to release funds; payment remains out of scope.

No material source requirement, non-goal, or qualitative intent was found omitted, softened, or contradicted. The PRD carries through per-line approve/flag/reject decisions with policy clauses, decision-table recording, the supplied SQLite/MCP tools and case data, labelled-set checks and judge, holdout use in live demos, and the exclusions on payment, employee email, and additional interfaces.

## Disposition

The user confirmed the minimal-dashboard assumption. The PRD specifies its review fields and keeps human approval separate from payment. No brief gaps remain open.
