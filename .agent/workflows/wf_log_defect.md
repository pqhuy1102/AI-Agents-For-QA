---
description: Draft a bug report per bug_report_rule from a Failed/Blocked execution record and save it to a local per-module bug list under tests/bugs/, instead of filing it in Jira. Designed to be swapped for a Jira-filing workflow later without changing where b
---

1. Check `docs/project-context.md` for PROJECT code and module list; ask only about what's missing. Confirm which module and which Failed/Blocked TC ID(s) this run is for — normally handed off directly from `track_manual_execution` Step 8, but this workflow can also be run standalone if pointed at an execution log.

2. Read the source case's Test Steps, Expected Result, and Test Data from `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv`, and the recorded Actual Result, Execution Status, build/version, and date from `tests/manual/<module>/executions/<cycle-name>-execution.csv`. Both are read-only here.

3. Check `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` for an existing open bug (Status not `Closed`/`Won't Fix`) against the same TC ID or the same symptom, per `bug_report_rule` Section 8. If one exists, propose linking this occurrence to it instead of filing a new one, and stop unless the person confirms a new bug is warranted.

4. Draft the bug report using every field in `bug_report_rule` Section 2: concrete Steps to Reproduce (Section 3, using the actual test data from execution, not generic placeholders), Expected vs. Actual kept distinct, a proposed Severity (Section 4), and Status `New`. Do not propose a root cause. Assign the next sequential Bug ID (`[PROJECT]_[MODULE]_BUG_[NNN]`) without reusing or renumbering an existing one.

5. Mask any credential, token, or sensitive personal data in the draft or its evidence per `bug_report_rule` Section 5 — describe what was seen instead of attaching raw evidence when masking isn't enough.

6. Present the full draft to the person for confirmation before saving — this is a human checkpoint even though nothing external is being filed yet, because the report becomes the record of truth for this bug going forward. Let the person adjust Severity or propose a Priority; do not set Priority unilaterally (`bug_report_rule` Section 4).

7. Save the confirmed bug to `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` (create `tests/bugs/<module>/` if missing), using the same CSV structure as `bug_report_rule` Section 2's field list and standard CSV escaping (`manual_testcases_rule` Section 15). Append as a new row for a new bug; never overwrite or renumber an existing Bug ID — a status/evidence update to an existing bug is a row update shown as a diff before writing, not a new row.

8. Add a cross-reference back in `tests/manual/<module>/executions/<cycle-name>-execution.csv`: note the new Bug ID in that TC ID's Notes field for this cycle, so the execution log and the bug list stay linked in both directions. Do not alter any other field in that row.

9. Report the saved path and Bug ID to the person. Mention explicitly that this is a local record, not a Jira ticket — filing to Jira is intentionally deferred; when that's wanted later, a `log_defect_to_jira` workflow can read this same `tests/bugs/<module>/*.csv` file as its source and file each row as a ticket, so nothing recorded now needs to be redone.
