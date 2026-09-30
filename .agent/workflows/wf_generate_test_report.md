---
description: End-to-end workflow that gathers execution results and defect data for a test cycle, evaluates them against the confirmed test plan's exit criteria via the test_summary_report skill, and produces a Test Summary Report for human release sign-off.
---

1. Check `docs/project-context.md` first for PROJECT code, module list, and Jira project key; ask only about what's missing. Then confirm the cycle scope with the user: which test plan (`docs/test-plans/<plan-name>-test-plan.md`) and which modules are in scope for this report.

2. Read the test plan. If its status is not `Confirmed`, tell the user and ask whether to proceed anyway (report will carry a Draft-plan caveat throughout, per `test_summary_report` Section 3) or wait until it's confirmed. Do not proceed silently against an unconfirmed plan.

3. For each in-scope module, check for `tests/manual/<module>/executions/<cycle-name>-execution.csv` (matching this report's cycle name to the one used by `track_manual_execution`). If it doesn't exist, tell the user this module's results will be marked `Not Tracked` and ask whether they can supply results directly (e.g. paste a summary) for this run, or whether to proceed with the gap noted. Never read Execution Status from the source `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` — that file holds design-time data only.

4. Check for automated test results for this cycle (from `automate_from_manual_testcase` runs or a CI report), if the user indicates automation is in play for any in-scope module. Merge these into the execution data by TC ID per `test_summary_report` Section 4 Step 2 — don't double-count a case covered both manually and by automation.

5. Gather defect data from `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` for each in-scope module (per `bug_report_rule` / `log_defect`). If a Jira connector is available and connected, it may supplement or replace this file — confirm with the user which source to trust if both exist and disagree. If neither is available, ask the user to supply the current defect list, or confirm proceeding with defect data marked `Not Available`. Do not attempt to write to Jira or to the bug list in this workflow — reading only.

6. Invoke the `test_summary_report` skill with: the confirmed (or caveated) exit criteria, the compiled execution data (including any `Not Tracked` modules), and the compiled defect data (including `Not Available` if applicable). Let the skill produce the evaluation, risk highlights, data-gap list, and recommendation exactly as its own process defines.

7. Present the draft report to the user before saving — this is a human checkpoint, especially for the Recommendation section. Ask if anything needs correcting (a wrong data source, a criterion that should be re-checked) before finalizing.

8. Save the confirmed report to `docs/reports/<cycle-name>-test-summary-report.md` (create `docs/reports/` if missing). If a report already exists for this exact cycle name, show a diff and confirm before overwriting rather than replacing it silently.

9. Report the saved path to the user, and restate plainly that the Recommendation section is a recommendation for their release decision-maker to confirm — this workflow does not mark a release as approved or blocked on its own.
