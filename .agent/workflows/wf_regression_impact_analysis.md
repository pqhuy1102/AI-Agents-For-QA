---
description: When a requirements specification changes, diff it against its prior version, identify affected Requirement IDs, find the manual and automated test cases that reference them, classify the impact, and report it — without editing any test case itself.
---

1. Confirm scope with the user: which module's `docs/requirements/<module>-requirements-spec.md` changed. Check `docs/project-context.md` for the PROJECT/module codes and, if recorded, the automation repo path — ask only about what's missing.

2. Establish the "before" version of the spec. Prefer git history if the file is under version control (e.g. diff against the previous commit or a commit/tag the person names). If git isn't available or the file isn't tracked, ask the person to point at a saved prior version or describe what changed. Do not proceed by guessing what changed from the current version alone — a diff needs two points.

3. Diff the two versions and extract, by Requirement ID (FR-xxx / BR-xxx / VAL-xxx): which were **added**, which were **removed**, and which were **modified** (a changed Business Rule, Field Spec, Actors & Permissions entry, or State Machine transition under the same ID counts as modified, not just changed wording). Ignore purely editorial changes (typo fixes, reformatting) that don't alter meaning — note them separately as non-impacting if asked, but don't run the rest of this workflow for them.

4. Search `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` for every case whose Requirement ID matches an added, removed, or modified ID from Step 3. This is a search, not an edit — the source CSV is read-only here.

5. If an automation repo path is recorded in `docs/project-context.md`, search it too for the TC ID/Requirement ID annotations that `automate_testcases` adds (per `api_automation_testing_rule` Section 11 / `ui_automation_testing_rule` Section 12) matching the same affected IDs. Skip this step if no automation path is known, and say so in the report rather than silently omitting it.

6. Classify each match:
   - **Orphaned** — its Requirement ID was removed. The case no longer traces to anything; propose retiring it, but do not delete or edit it here.
   - **Needs Review** — its Requirement ID was modified. The case may now be stale (wrong expected result, outdated field constraint, changed state transition); propose a pass through `review_testcases` or a direct update, but do not edit it here.
   - **Coverage Gap** — a Requirement ID was added and no existing case (manual or automated) references it yet; propose running `generate_testcases_from_requirement` for that specific addition.

7. Present the Impact Report to the user (Step 8's structure) before suggesting or taking any follow-up action. This workflow only analyzes and reports — it never edits a test case, retires one, or triggers another workflow without the person choosing to.

8. Save the report to `docs/requirements/<module>-regression-impact-<date>.md`:

   ```markdown
   # Regression Impact Analysis: <module>

   Spec: docs/requirements/<module>-requirements-spec.md
   Compared: <before reference> → <after reference> Date: <date>

   ## Requirement Changes

   | Requirement ID | Change Type (Added/Removed/Modified) | Summary |

   ## Impacted Manual Test Cases

   | TC ID | Requirement ID | Classification | Suggested Action |

   ## Impacted Automated Tests

   | Test (file/name) | Requirement ID | Classification | Suggested Action |
   (or "Not checked — no automation repo path recorded")

   ## Coverage Gaps

   | Requirement ID | Suggested Action |

   ## Non-Impacting Changes

   - <editorial-only changes noted but not acted on>
   ```

9. Ask the user which suggested actions to act on now, if any — for example, running `review_testcases` on the Needs Review set, or `generate_testcases_from_requirement` for the Coverage Gap set. Do not chain into those workflows automatically; each is its own confirmed step.
