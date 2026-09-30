---
description: End-to-end workflow that reads a module's manual test case suite, selects automation candidates, generates automated tests via the automate_testcases skill under the applicable architecture rule, validates them, and marks the source cases "Automated"
---

1. Before asking the user anything, check whether `docs/project-context.md` exists (created by the `project_onboarding` workflow). If it does, read it and treat its values as established project context: PROJECT code, module list (slug and code), tech stack and automation frameworks, automation repo/path, environments, CI tool, and supported browsers/devices. Ask only about what is `Not Provided` there or specific to this run. If the file does not exist, continue without it and mention once that running `project_onboarding` would avoid repeated questions; do not block on it. If the person's instruction differs from the file, follow the person's instruction, point out the difference, and suggest re-running `project_onboarding` to update it. This workflow never edits `docs/project-context.md` itself.

   Then clarify scope with the user: which module (`tests/manual/<module>/*.csv`), which track(s) apply (API via `api_automation_testing_rule`, UI/E2E via `ui_automation_testing_rule`, or both), and which automation codebase to generate into (confirm the target repo/path — a fresh project or an existing one). If the context file records an automation framework, language, and repo path, propose them as the default target rather than asking from scratch, and treat "None" for a track as a reason to ask before generating code for that track. If any of this is ambiguous, ask before proceeding.

2. Confirm access to both inputs: the manual test case CSV for the module, and the target automation codebase (read access to inspect existing conventions, write access to add new files). Do not assume access that hasn't been confirmed.

3. Read the full manual test case CSV for the module. For every row (or the subset the user specified), run the `automate_testcases` skill's Step 1 (Candidate Suitability Check) — stable, high-frequency, deterministic, not exploratory — without generating any code yet.

4. Present the shortlist to the user before generating anything: which TC IDs passed the check and will be automated, which were rejected and why. Pause here for confirmation — do not generate code for a rejected case, and do not silently drop a case from the list without showing it.

5. For the confirmed shortlist, group cases by track (API / UI / Hybrid) per `automate_testcases` Step 2. If a case's track is unclear from its steps, ask rather than guess.

6. Before generating into an existing codebase, inspect its current conventions once per session (Page Object/Service structure, naming patterns, fixture/retry helpers already in place) per `automate_testcases` Section 11 — reuse and extend rather than introducing a second inconsistent pattern. Use the tech stack recorded in `docs/project-context.md` as a starting hint, but verify it against the actual code and report any mismatch rather than trusting the file over what is observed.

7. For each case in the shortlist, run `automate_testcases` Steps 3-6 to design and generate the automated test: map steps to the correct architecture layer, map the Expected Result to concrete assertions, map test data, and annotate the generated test with its TC ID, Requirement ID, and Risk Level. Carry forward any `Requires Clarification` flags from that skill rather than silently resolving them.

8. If step 7 raised any `Requires Clarification` items that materially affect what the automated test checks, pause and resolve them with the user before treating that case as generated — per the same coverage-blocking-gap principle used in `generate_testcases_from_requirement`.

9. Write the generated code files to the target codebase, following its existing structure/naming (per Step 6). Do not overwrite an existing automated test for a different TC ID; if a file already exists for this TC ID, treat it as an update and show what changed before overwriting.

10. Run the newly generated tests in a targeted (not full-suite) run and inspect the result. Run against an environment listed in `docs/project-context.md` when one is recorded; if several are listed or none is, ask which to use rather than picking one, and never run against a production environment without explicit authorization (Global Rule Section 9).

11. If a test fails: classify the failure first (per `api_automation_testing_rule` Section 6 / `ui_automation_testing_rule` Section 8) before changing anything. Apply a fix only within the Auto-Fix and Auto-Healing Safety bounds of the applicable rule (its Section 9) — never widen an assertion, add a blanket sleep, or switch to a more fragile locator just to get a pass. For a UI/timing-related fix, re-run several times before considering it validated, per `ui_automation_testing_rule` Section 9.

12. Once a test passes and is validated, do not treat CI suite placement as automatic. Recommend smoke vs. regression placement per Risk Level and execution cost (`ui_automation_testing_rule` Section 11 for UI; analogous cost/risk judgment for API), using the CI tool recorded in `docs/project-context.md` when available. Treat any change to actual pipeline configuration as an external-system, human-confirmed action — do not modify pipeline config without explicit confirmation.

13. Update the source manual test case CSV: mark each successfully automated TC ID as "Automated" (per the applicable rule's Traceability Bridge section) rather than deleting the manual case. Never renumber or overwrite an unrelated existing TC ID while doing this.

14. Deliver a final report to the user: cases automated (with TC ID/Requirement ID), cases rejected at Step 3/4 and why, validation results per case, any pipeline-placement recommendations awaiting confirmation from Step 12, and any remaining Open Questions carried from `automate_testcases`.
