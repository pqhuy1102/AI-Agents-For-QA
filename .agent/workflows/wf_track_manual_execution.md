---
description: Record execution results for a manual test cycle into a per-cycle execution log, without modifying the source test case CSV, using the shared result vocabulary.
---

1. Check `docs/project-context.md` for PROJECT code and module list; ask only about what's missing. Confirm scope: which module, and which cycle/run this execution belongs to (match the cycle name used in a confirmed test plan, `docs/test-plans/<plan-name>-test-plan.md`, when one exists — ask for a cycle name if not).

2. Ask which mode this run is: **(a)** the agent executes the test steps itself (via available browser/API tools) and records the outcome, or **(b)** the person already ran the tests and is reporting results. If unclear, ask rather than assume.

3. Read `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` for the case list. This file is read-only in this workflow — never write Execution Status or Actual Result into it.

4. Check whether `tests/manual/<module>/executions/<cycle-name>-execution.csv` already exists. If it does, read it — this may be a partial run being continued, or a re-test after a fix. If not, this is a new execution log for the cycle.

5. For each case in scope:
   - **Mode (a):** follow the case's Test Steps using available tools, compare the actual outcome to Expected Result, and determine the status. Before any destructive or irreversible step, confirm authorization per Global Rule Section 4.1 — do not execute it silently just because the script calls for it.
   - **Mode (b):** ask the person for the Execution Status and Actual Result per case — batched, not one case per message, matching the grouped-question style used in `project_onboarding`.

6. Record, per case: TC ID, Requirement ID and Risk Level (carried from the source CSV for quick reference — never edited here), Execution Status (Global Rule Section 9 vocabulary only: Passed / Failed / Skipped / Blocked / Not Executed / Unknown — never a different label), Actual Result (required for Failed/Blocked, optional but encouraged otherwise), Executed By, Execution Date, Build/Version if known, and Notes.

7. If a record for a TC ID already exists in this cycle's execution log (a re-test), show the old vs. new values and confirm before overwriting — never silently replace a prior result. A genuinely new attempt in the same cycle is an update to that TC ID's row, not a duplicate row.

8. Immediately after recording a Failed or Blocked result, ask whether to draft a bug report for it now (via the bug-logging workflow, once available) or defer — don't force the detour, but don't let a Failed result pass unmentioned either.

9. Save the execution log to `tests/manual/<module>/executions/<cycle-name>-execution.csv` (create the `executions/` folder if missing), using standard CSV escaping (per `manual_testcases_rule` Section 15) for any field containing commas, quotes, or line breaks.

10. Report a summary to the person: cases executed this run vs. total in scope, counts per status, and the saved file path. This file is what `test_summary_report` reads for execution data — mention that a report generated now will pick up exactly what was just recorded.
