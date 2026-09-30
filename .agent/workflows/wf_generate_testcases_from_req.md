---
description: End-to-end workflow that analyzes a page/module/feature and produces a formatted manual test case suite, by chaining requirements_analyzer → generate_testcases → manual_testcases_rule in sequence.
---

1. Before asking the user anything, check whether `docs/project-context.md` exists (created by the `project_onboarding` workflow). If it does, read it and treat its values as established project context: PROJECT code, module list (slug and code), Requirement ID convention, environments, and supported browsers/devices. Ask only about what is `Not Provided` in that file or specific to this run. If the file does not exist, continue without it and mention once that running `project_onboarding` would avoid repeated questions; do not block on it.
   - If the person's instruction for this run differs from the context file (for example, a different module code), follow the person's instruction, point out the difference, and suggest re-running `project_onboarding` to update the file. This workflow never edits `docs/project-context.md` itself.
   - If the module being analyzed is not in the context file's module list, ask whether it is a new module rather than inventing its slug or code.

   Then clarify the target and scope with the user: which page/module/feature, which actors are relevant, and what evidence is available (live app to inspect, DOM/HTML, screenshots, API behavior, existing docs, existing test cases). If the target is ambiguous, ask before proceeding.

2. Confirm which evidence sources are actually accessible in this session (browser tools, provided files/screenshots, API access, existing requirements docs). Do not assume access to evidence that has not been confirmed. When `docs/project-context.md` lists environments, use them as the candidate places to inspect the live application, and use the credentials-location names it records rather than asking for credentials in chat.

3. Run the `requirements_analyzer` skill against the confirmed evidence to produce a Requirements Specification. Ensure it includes, when applicable to the feature's complexity: Functional Requirements, Business Rules, Field Specifications, Actors & Permissions, State Machine, Workflows (with decision points), Validation & Error Handling, and Open Questions — per that skill's own complexity-scaling rule (its Section 2.5).

   Save it to `docs/requirements/<module>-requirements-spec.md` by default (`<module>` = the module slug from `docs/project-context.md` when available, otherwise the identifier confirmed in Step 1, kebab-case). If the context file defines a Requirement ID convention, apply it to the FR/BR/VAL identifiers in the specification. Do not ask for confirmation on a first-time save.
   - If the file already exists: do not overwrite silently. Show a brief diff/summary of what changed, and ask the user to confirm before overwriting, appending a dated addendum, or saving as a new version.

4. Present a short summary of the Requirements Specification to the user, highlighting: complexity signals observed (multi-actor? multi-state? multi-step?), and any Open Questions.

5. If any Open Question would materially change test coverage or expected results (e.g. an undocumented permission rule, an unconfirmed state transition, an unknown boundary value), pause and ask the user to either supply the missing information or explicitly accept proceeding with it marked as an assumption. Do not silently proceed past a coverage-blocking gap.

6. Run the `generate_testcases` skill using the Requirements Specification from Step 3 as its primary input (per that skill's Section 3, which prefers structured `requirements_analyzer` sections over re-deriving evidence). Specifically:
   - Feed Functional Requirements / Business Rules / Field Specifications into Equivalence Partitioning, Boundary Value Analysis, and Validation coverage.
   - Feed the Actors & Permissions section (if present) into Decision Table design per its Section 7.1.
   - Feed the State Machine section (if present) into State Transition design per its Section 8.1.
   - Feed multi-step Workflows (with decision points) into scenario design, applying the split-vs-E2E heuristic referenced in its Section 9.1.

7. Confirm the `generate_testcases` output uses the shared vocabulary before proceeding: Risk Level on the High/Medium/Low scale, and Requirement ID values matching the FR-xxx/BR-xxx/VAL-xxx identifiers from Step 3's specification (its Section 13 traceability requirement).

8. Present the Test Design Summary (main risks, techniques used, coverage areas, assumptions/gaps) and the candidate test case list to the user before final formatting, so design-level issues are caught early rather than after formatting.

9. Format the final test cases according to `manual_testcases_rule`:
   - Use its Test Case Structure (Section 14): TC ID, Module, Requirement ID, Risk Level, Test Scenario, Pre-Condition, Test Steps, Test Data, Expected Result, Priority, Environment, Review Status.
   - Assign TC IDs per its Section 13 convention `[PROJECT]_[MODULE]_TC_[NNN]`, taking the PROJECT code and the module code from `docs/project-context.md` when available. If that file defines a different TC ID format under Project-Specific Overrides, use it for this project. If the PROJECT or module code is still unknown, ask the user rather than inventing one.
   - Set the Environment field using the supported browsers/devices from `docs/project-context.md` where execution differs by environment; use `Any` otherwise, and never invent a browser or device that is not listed.
   - Apply its Test Data Rules (Section 5) and Test Case Design rules (Section 4), including the invalid-data rationale requirement.
   - Apply its split-vs-E2E heuristic (Section 4.1) as the final word on case granularity if it conflicts with a design-time draft from Step 6.

10. If the user requests CSV output (or the consuming tool requires it), generate it per `manual_testcases_rule` Section 15 — full header when Requirement ID/Environment/Review Status apply, reduced header with an explicit note otherwise. Use standards-compliant CSV escaping; never produce visually-formatted pseudo-CSV.

11. Run the Final Quality Checks defined in `manual_testcases_rule` Section 19 (testability, data, coverage, quality, format) before returning results. Fix any check that fails rather than reporting it as a caveat.

12. Deliver the final package to the user:
    - Requirements Specification (or a link/reference to it if already shared separately).
    - Test Design Summary.
    - Final formatted test cases (table or CSV, per Step 10).
    - Coverage Summary (happy path, negative, boundary, edge cases, security/permission incl. actor×action, state transitions, validation, business-rule combinations — only the dimensions actually applicable).
    - A single consolidated Open Questions list merging gaps identified in Step 3 (requirements-level) and Step 6 (design-level), de-duplicated.

13. Save the formatted test cases to `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` by default (`<PROJECT>`/`<MODULE>` = the identifiers used for TC IDs in Step 9, which come from `docs/project-context.md` when available; if either is unknown, it was already resolved with the user in Step 9). Do not ask for confirmation on a first-time save at this path.
    - If the target CSV file already exists: read it first. Do not overwrite it or renumber/reuse any existing TC ID (per `manual_testcases_rule` Section 13 and Strict Rule 14).
      - New test cases → append with the next sequential `_TC_NNN` number.
      - A generated case that appears to update/replace an existing TC ID (same Test Scenario/Requirement ID) → show the old vs. new content and ask the user to confirm before modifying that row; never modify it silently.
    - Report the final file path (and, for an existing-file update, a short summary of what was added/changed) to the user after saving.
