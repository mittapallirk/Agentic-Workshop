# Prose Review — Case A PRD

This document exists to help the workshop owner, implementer, and evaluators build and assess Case A. The PRD uses a concise, requirements-led voice, defines domain terms with capitalization, and marks unconfirmed details as assumptions; preserve those choices.

| Pass | Original Text | Revised Text | Changes |
|---|---|---|---|
| prose | “Approved items over CAD $500 wait for a person's yes.” | “Approved items over CAD $500 remain pending until a person records an approval outcome.” | Clarifies that the gate can end in either a yes or no, consistent with FR-6. |
| prose | “A finance reviewer inspects a Claim, understands each Decision, resolves follow-up and approval items, and verifies the total.” | “A finance reviewer inspects a Claim, understands each Decision, reviews flagged items, records approval outcomes, and verifies the total.” | “Resolves follow-up” implies an unspecified ability to correct data or decisions; use the actions specified elsewhere in the PRD. |
| prose | “An item more than 60 days older than its Claim submission date is rejected under clause 1.2.” | “An item dated more than 60 days before its Claim submission date is rejected under clause 1.2.” | Replaces the awkward comparative “days older” with a direct date relationship. |
| prose | “A person must explicitly record yes before the item can leave this state.” | “A person must explicitly record an approval outcome before the item can leave this state.” | Aligns the sentence with the following yes/no outcomes; otherwise “yes” can read as the only way to end Pending Approval. |
| prose | “The case's SQLite, MCP, and evaluation constraints are retained where they define Case A.” | “The SQLite, MCP, and evaluation constraints specified in the case are retained where they define Case A.” | Avoids the awkward possessive and clarifies that the case specifies these constraints. |

Five recommendations. These are local wording changes; no length target was provided. The review retains the document’s workshop-specific terminology, assumptions, and decision rules.

## Author triage

All five wording recommendations were applied. They clarify the yes/no approval outcome, the reviewer's actions, the 60-day comparison, and the case constraints without changing product scope. The final PRD is 2,556 words.
