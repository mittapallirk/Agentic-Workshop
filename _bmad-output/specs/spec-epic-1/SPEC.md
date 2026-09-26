---
id: SPEC-epic-1
companions: []
sources: [../../../INTENT.md]
---

> **Canonical contract.** This SPEC and the files in `companions:` are the complete, preservation-validated contract for what to build, test, and validate. Source documents listed in frontmatter are for traceability — consult them only if you need narrative rationale or prose color this contract intentionally omits.

# Epic 1: triage data and schema

## Why

The workshop needs a stable triage-decision contract and local ticket/customer data before later epics can build an agent and measure its decisions. Epic 1 establishes that foundation while keeping the seed data unchanged and the existing MCP server compatible.

## Capabilities

- **CAP-1**
  - **intent:** Systems can exchange triage decisions through a single validated JSON contract.
  - **success:** A valid decision contains category `billing`, `bug`, `access`, `performance`, or `how-to`; priority `P1`, `P2`, `P3`, or `P4`; route `billing-team`, `bug-team`, `access-team`, `performance-team`, or `how-to-team`; and a one-sentence rationale. Any decision outside this contract is rejected with a clear error.

- **CAP-2**
  - **intent:** A workshop attendee can load the supplied ticket and customer data into a local database with one command.
  - **success:** `uv run python load_seed.py` loads `seed/tickets.csv` and `seed/customers.csv` into `app.db` as SQLite tables `tickets` and `customers`, with columns matching the respective CSV files. Running the command twice produces the same database.

## Constraints

- Use Python 3.12 or newer, managed with uv.
- Files under `seed/` are read-only.
- This epic makes no network calls and uses no API keys.
- `mcp/triage_server.py` must continue to work with the database table and column names.

## Non-goals

- The triage agent.
- MCP tools.
- Evals.
- A user interface.

## Success signal

`uv run python load_seed.py` creates `app.db` with both expected tables and CSV-matching columns, and a second run leaves the database equivalent. The triage-decision contract accepts only decisions with the specified fields and values, and invalid decisions produce a clear error.

## Assumptions

- The triage-decision JSON contract is a data/schema deliverable in this epic; its use by an agent is deferred.
