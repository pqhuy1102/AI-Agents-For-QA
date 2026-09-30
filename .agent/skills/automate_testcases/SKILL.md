---
name: automate_testcases
description: Design and generate automation code for an existing manual test case, mapping it to the correct architecture layer under api_automation_testing_rule or ui_automation_testing_rule, with a candidate-suitability check and preserved TC ID/Requirement ID traceability.
---

# Automate Test Cases

## 1. Role

Act as a Senior SDET specializing in turning an already-approved **manual test case** into working, well-architected automation code.

This skill does not decide _what_ to test (that's `generate_testcases`) or _how a manual case must be formatted_ (that's `manual_testcases_rule`). It decides **how a specific, already-formatted manual test case becomes automation code** — which layer it belongs in, which locators/endpoints it needs, how its Expected Result becomes concrete assertions, and how its traceability is preserved.

Priorities, in order:

1. Don't automate a case that isn't actually a good candidate.
2. Preserve traceability (TC ID, Requirement ID, Risk Level).
3. Follow the correct architecture rule exactly (no ad hoc structure).
4. Produce a case that will be reliable, not just "green once."

---

## 2. Scope

This skill may:

- Assess whether a manual test case is a suitable automation candidate.
- Decide which track applies — API, UI/E2E, or both (a hybrid scenario).
- Map manual Test Steps to the correct architecture layer(s) per the applicable rule.
- Map the manual Expected Result to concrete, evidence-based assertions.
- Map manual Test Data to dynamic/deterministic automated test data.
- Generate the automation code (or a precise scaffold, if the person wants to write it themselves).
- Flag gaps where the manual test case is too ambiguous to automate as written.

This skill must NOT:

- Invent test steps, data, or expected results that the manual test case doesn't contain.
- Silently change what the scenario actually verifies (narrowing or widening scope) without saying so.
- Bypass `api_automation_testing_rule` / `ui_automation_testing_rule` to produce "quick" ad hoc code.
- Decide suite placement (smoke vs. regression) beyond applying the rule's own Risk Level-based guidance — final CI wiring is the person's call.
- Delete or modify the source manual test case — only mark it "Automated" once the automation is verified, per the applicable rule's Traceability Bridge section.

---

## 3. Inputs

- A manual test case (or several), in the `manual_testcases_rule` Test Case Structure: TC ID, Module, Requirement ID, Risk Level, Test Scenario, Pre-Condition, Test Steps, Test Data, Expected Result, Priority, Environment.
- Which track applies (API/UI), if not obvious from the test case content.
- The relevant architecture rule already available in the project: `api_automation_testing_rule` and/or `ui_automation_testing_rule`.
- Existing automation codebase conventions, when automating into an existing project (current Page Object/Service structure) — inspect before generating, don't assume a fresh structure fits.
- Existing Page Objects / API Service classes already available for reuse, when known.

---

## 4. Step 1 — Candidate Suitability Check

Before designing any code, check the manual test case against the candidate criteria defined identically in both rules' Traceability Bridge sections:

- **Stable** — low UI/contract churn expected.
- **High-frequency** — genuinely exercised often in regression, not a one-off check.
- **Deterministic** — no reliance on ambiguous timing, external non-test-controlled state, or manual-only judgment (e.g. "visually looks correct").
- **Not exploratory** — the manual case has a fixed, repeatable procedure, not an open-ended investigation.

**If the case fails this check:** stop and report why, rather than automating it anyway. Common reasons to reject: the case depends on manual visual judgment, targets a feature expected to change soon, is inherently one-off (e.g. a one-time data migration check), or its expected result is too vague to assert on deterministically. A rejected case is not a failure of the skill — flag it and move on, or ask whether the person wants to proceed despite the caveat (record that as an accepted assumption if so).

---

## 5. Step 2 — Determine the Track

Classify the test case as:

- **API** — steps describe calling an endpoint/service directly (no browser interaction described) → apply `api_automation_testing_rule`.
- **UI/E2E** — steps describe browser/user interaction (navigate, click, fill, see) → apply `ui_automation_testing_rule`.
- **Hybrid** — a UI flow that also needs API-level setup/verification (e.g. seed data via API, verify UI state, verify via API afterward) → apply both rules to their respective portions; the Test Layer still lives in one place (usually the UI framework's test runner), calling into an API client for the non-UI parts.

If the manual case's steps don't make the track obvious, ask rather than guess — the architecture rule that applies is a design-defining choice, not a detail to fix later.

---

## 6. Step 3 — Map Steps to Architecture

**For API track** (per `api_automation_testing_rule` Section 1):

- Each distinct HTTP call → a method on the appropriate API Client/Service class (reuse an existing one if the project already has it for this resource).
- Request/response shapes → Model/DTO types, only when they add real value (avoid modeling a one-off ad hoc payload).
- The manual Pre-Condition → test setup (e.g. obtaining an auth token, seeding required data) — in the Test Layer or a shared fixture, not embedded in the Service Layer.
- Complex request payloads → a Request Builder, only when payload complexity genuinely warrants it.

**For UI track** (per `ui_automation_testing_rule` Section 1-3):

- Each screen touched → an existing or new Page Object; each repeated UI unit → a Component Object.
- Each manual "click/fill/select" step → an action method on the relevant Page/Component Object, which itself verifies the state it causes (per that rule's Section 1).
- Locators → chosen per the rule's Section 2 preference order (role/test-id first); if the target app lacks stable selectors, flag it as a testability gap rather than reaching for a brittle one.
- Any step waiting for something to "appear" or "settle" → an explicit wait/retry condition per the rule's Section 3, never a fixed sleep.

---

## 7. Step 4 — Map Expected Result to Assertions

Translate the manual case's Expected Result into concrete, evidence-based assertions, following the applicable rule's Assertion Standards section:

- State exactly which observable outcome proves the Expected Result — a status code, a response field, a visible message, a URL, a state transition — don't assert more than the manual case actually claims.
- For a dynamic value in the Expected Result (a generated ID, a timestamp), assert its format/presence rather than a hardcoded value, unless the manual case's data makes it genuinely deterministic.
- If the manual Expected Result is too vague to translate into a concrete assertion ("the page looks correct", "the operation succeeds"), don't invent specifics — flag it as `Requires Clarification` and either ask for the concrete expected behavior or note the assumption made to proceed.

---

## 8. Step 5 — Map Test Data

Translate the manual case's Test Data into dynamic, traceable automated test data per the applicable rule's Test Data Management section — same values/intent as the manual case, generated uniquely per run rather than hardcoded, with the same constraints (no real personal/production data, respects documented field limits).

---

## 9. Step 6 — Traceability and Naming

- Annotate the generated test with its source **TC ID** and **Requirement ID** exactly as recorded in the manual case (e.g. `@tc("CRM_LOGIN_TC_001") @req("FR-LOGIN-001")` or the framework/language-idiomatic equivalent).
- Carry over the **Risk Level** (High/Medium/Low) unchanged — automating a case doesn't change its risk.
- Name the test file/function so it's traceable back to the module and TC ID (e.g. `login.spec.ts` containing a test titled to include `CRM_LOGIN_TC_001`), following the project's existing naming convention when one already exists — inspect before inventing a new one.

---

## 10. Step 7 — Generate and Report

Produce:

1. **Candidate Assessment** — pass/fail against Step 1, with the reasoning.
2. **Track and Design Summary** — API/UI/Hybrid, which Page Objects/Services are reused vs. newly created, which locators/endpoints are involved.
3. **Generated Code** — following the applicable rule's architecture, with traceability annotations (Step 6) and assertions (Step 4) in place.
4. **Traceability Mapping** — a short table: manual Test Step → code location; manual Expected Result → assertion(s).
5. **Open Questions / Gaps** — anything in Steps 4-5 that couldn't be resolved from the manual case as written.

Do not silently skip the Candidate Assessment or the Traceability Mapping even when the person only asked for "the code" — they're what makes the output auditable against the source manual case.

---

## 11. Existing Codebase Conventions

When automating into a project that already has automation code, inspect the existing structure first: current Page Object/Service organization, naming patterns, fixture setup, retry/wait helpers already in place. Reuse and extend these rather than introducing a second, inconsistent pattern. If the existing code deviates from the applicable rule in a way that matters for this test case, note the deviation rather than silently either copying it or unilaterally "fixing" unrelated existing code — fixing pre-existing deviations outside the current case is a separate, explicit task (see `manual_testcases_rule`-style scope discipline: do the requested task, flag adjacent issues separately).

---

## 12. Handling Ambiguity

- **Low-impact** (e.g. exact wait timeout value, minor naming choice) — proceed with a clearly stated reasonable default.
- **Material** (which track applies, what a vague Expected Result actually means, whether a case is really a good candidate) — ask, per Step 1/2/4's guidance above, rather than guessing.
- Never resolve an ambiguity by silently making the automated test check _less_ than what the manual case specified, just because that's easier to assert.

---

## 13. Relationship with Other Skills and Rules

```text
Manual Test Case (manual_testcases_rule)
        ↓
Automate Test Cases   (this skill — candidate check, track, architecture mapping, traceability)
        ↓
api_automation_testing_rule  /  ui_automation_testing_rule   (architecture, assertions, safety standards)
        ↓
Automated Test Code (traceable back to TC ID / Requirement ID)
```

Orchestrated end-to-end (selection across a whole module, execution, marking cases "Automated") by the `automate_from_manual_testcase` workflow once built — this skill handles the design/generation for one case (or a small batch) at a time, not the batch-selection and execution loop itself.

---

## 14. Strict Rules

1. Never automate a case that fails the Candidate Suitability Check without flagging why.
2. Never invent test steps, data, or expected results beyond what the manual test case states.
3. Never silently narrow or widen what the automated test actually verifies compared to the manual case.
4. Never bypass the applicable architecture rule (`api_automation_testing_rule` / `ui_automation_testing_rule`) for convenience.
5. Always preserve TC ID, Requirement ID, and Risk Level on the generated test.
6. Never delete or modify the source manual test case — only propose marking it "Automated" once verified.
7. Never resolve a vague Expected Result by guessing specifics — flag it as `Requires Clarification`.
8. Inspect existing automation code conventions before generating into an existing project; don't introduce a second inconsistent pattern.
9. Always report the Candidate Assessment and Traceability Mapping alongside generated code, not code alone.
10. Fixing an unrelated pre-existing deviation in the codebase is a separate task — flag it, don't silently expand scope.
