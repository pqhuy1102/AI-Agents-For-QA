---
trigger: always_on
---

# Manual Test Cases Rule

## 1. Purpose

Produce test cases that are specific, testable, reproducible, traceable, risk-aware, domain-agnostic, free from unsupported assumptions, and executable by another QA without interpretation. Applies to all domains (e-commerce, fintech, healthcare, SaaS, logistics, enterprise, security, consumer). Domain behavior must come from evidence, not assumption. When a requirements spec exists (e.g. from a requirements_analyzer skill), its FR/BR/Field Spec/Actors & Permissions/State Machine sections map directly to coverage below.

## 2. Core Principles

- **Requirement-Driven**: every test case validates a known requirement, rule, AC, or observed behavior. Don't create a case just because a category exists.
- **Risk-Based Coverage**: prioritize by business/user impact, severity, likelihood, data sensitivity, dependencies, complexity, state transitions, integration points, compliance, defect history. Optimize for meaningful coverage, not count.
- **Evidence Over Assumptions**: never invent business rules, validation rules, roles, permissions, limits, error messages, API behavior, security requirements, formats, supported browsers/devices, test data constraints, or state transitions. Missing info → flag as assumption/clarification, never silently convert to a test case.

## 3. Coverage Dimensions (mandatory only when applicable)

**3.1 Happy Path** — primary supported workflow with valid data.

**3.2 Negative Testing** — invalid/unexpected input or actions: invalid input, missing required data, invalid state, unsupported operation, failed dependency, duplicate submission. Only if relevant.

**3.3 Boundary Testing** — min/max value/length, just-below/above, empty, zero, negative — only from documented/observed constraints, never invented.

**3.4 Edge Cases** — empty/very large dataset, repeated/rapid actions, special characters, unexpected state transitions, interrupted workflow, expired session, delayed dependency. Must be relevant, not filler.

**3.5 Security & Permission Testing** — when feature involves auth, roles, permissions, sensitive data, privileged actions, ownership, sessions, access control. Skip if not applicable.

_3.5.1 Systematic Actor/Permission Coverage_ (when an Actors & Permissions matrix exists):

1. List every actor × action combination relevant to the feature.
2. Classify each: **Allowed** (cover as positive case), **Denied** (cover as negative case — verify exact denial: hidden/disabled control, error message, or 403), **N/A** (no case needed).
3. Merge combos only if actor+action+expected behavior are identical; keep separate if denial mechanism differs (hidden vs disabled vs API error).
4. Unknown expected behavior → `Requires Clarification`, never guessed.

**3.6 Validation Testing** — required fields, type, format, length, range, allowed values, character restrictions, cross-field/conditional validation, all evidence-based.

**3.7 State Transition Testing** (when a State Machine exists or entity has multiple named states):

- Valid transitions: verify move + documented side effects (notifications, field/permission changes).
- Invalid transition attempts: verify system blocks/rejects moves not reachable from current state.
- Boundary/terminal states: verify no further action where marked irreversible/terminal.
- Actor-restricted transitions: verify both permitted actor's success and other actors' denial (cross-ref 3.5.1).
- Undocumented transitions → `Requires Clarification`, never assumed absent.

## 4. Test Case Design

Each case must answer: what's tested, required pre-state, exact action, exact data, expected result. Avoid vague steps ("enter a valid email") — use concrete data ("enter `qa.user01@example.com`"). For invalid data, state why it's invalid (e.g. "`abc`, below the documented 8-char minimum").

**4.1 Splitting vs. Combining Workflow Steps**
Prefer **smaller, focused cases** when: steps are owned by different actors; a step has multiple meaningful outcomes (approve/reject) that would force duplicating earlier steps per variant; or early steps are stable/low-risk and shouldn't be re-executed in every case (express as precondition instead).
Prefer **one E2E case** when: the tested value IS the full chain working together (e.g. SLA across a full process), or the workflow is short (~3–5 steps) with no branching.
Either way: an E2E case still lists every Requirement ID it exercises; split cases covering one workflow share a common scenario name/tag for execution grouping.

## 5. Test Data Rules

- **Specific & reproducible**: concrete values (e.g. `Password: Test@12345`, `Date: 2026-10-15`), not "invalid data".
- **Must match the scenario**: data should clearly demonstrate the condition tested; pair with a specific expected result.
- **Environment-dependent values**: use explicit variables (`<AUTHORIZED_TEST_ACCOUNT>`, `<CONFIGURED_MAX_VALUE>`) rather than invented values; define the variable in data/precondition.
- **Sensitive data**: never real passwords, API keys, tokens, private keys, production credentials, or real PII — use synthetic/safe test accounts only.

## 6. Preconditions

Describe required state before execution only when relevant (e.g. "User is authenticated with the required role", "Target item exists and is in Draft status"). Avoid unnecessary preconditions.

## 7. Test Steps

Sequential, atomic, unambiguous, executable by another tester. Don't combine independent actions into one step if it creates ambiguity. Number steps explicitly.

## 8. Expected Results

Must be observable: UI state, exact displayed message (only if documented/observed), navigation, data change, API-visible behavior, state transition, access restriction, error handling. Never invent an exact message not backed by evidence.

## 9. Test Case Independence

Each case independently executable where practical. Avoid "TC-002 only after TC-001" dependencies — build state via setup instead. If dependency is unavoidable, document it explicitly in the precondition.

## 10. Duplicate/Overlap Prevention

Don't create superficially-different cases for the same behavior/condition. Keep cases separate only when business rule, risk, role, state, validation behavior, dependency, or expected outcome genuinely differs.

## 11. Risk Level

| Level  | Meaning                                                         |
| ------ | --------------------------------------------------------------- |
| High   | Major business/user/data/security/operational/compliance impact |
| Medium | Meaningful but non-critical functional/user impact              |
| Low    | Limited impact, minor functionality                             |

Never assign risk based on test complexity alone.

## 12. Priority

| Priority | Meaning                                      |
| -------- | -------------------------------------------- |
| P0       | Blocks core usage / severe impact            |
| P1       | High-value, important for release confidence |
| P2       | Normal, not release-blocking                 |
| P3       | Low-impact / secondary                       |

Priority = execution importance, not defect severity. Never interchange with Risk Level.

## 13. Test Case ID

Format: `[PROJECT]_[MODULE]_TC_[NNN]` (e.g. `CRM_LOGIN_TC_001`). Uppercase unless project convention differs; stable project/module identifiers; 3-digit sequential numbering; never reuse or unnecessarily renumber IDs; preserve IDs when editing existing cases. Unknown project/module → ask for clarification or use an explicit placeholder.

## 14. Test Case Structure

| Field           | Description                                                                    |
| --------------- | ------------------------------------------------------------------------------ |
| TC ID           | Unique identifier                                                              |
| Module          | Feature/module under test                                                      |
| Requirement ID  | Source FR/BR/VAL this case validates; `Not Available` if no formal spec exists |
| Risk Level      | Per Section 11                                                                 |
| Test Scenario   | High-level behavior validated                                                  |
| Pre-Condition   | Required state before execution                                                |
| Test Steps      | Ordered steps                                                                  |
| Test Data       | Concrete data used                                                             |
| Expected Result | Observable outcome                                                             |
| Priority        | Per Section 12                                                                 |
| Environment     | Browser/device/OS/platform if execution differs; `Any` otherwise               |
| Review Status   | `Draft` / `Reviewed` / `Approved`, defaults to `Draft`                         |

Environment/Review Status may be omitted when the consuming tool's fixed schema excludes them — state the omission explicitly.

## 15. CSV Output

Full header:

```csv
TC ID,Module,Requirement ID,Risk Level,Test Scenario,Pre-Condition,Test Steps,Test Data,Expected Result,Priority,Environment,Review Status
```

If the import schema is fixed and lacks Requirement ID/Environment/Review Status, use the reduced header and note the omission:

```csv
TC ID,Module,Risk Level,Test Scenario,Pre-Condition,Test Steps,Test Data,Expected Result,Priority
```

Standard CSV escaping: wrap fields containing commas, quotes, or line breaks in double quotes; escape internal quotes by doubling. Never produce malformed CSV for visual compactness.

## 16. Traceability

Maintain Requirement ID ↔ Test Case linkage (Section 14). One case may reference multiple Requirement IDs (comma-separated) when it validates several (e.g. an E2E case per 4.1). If the CSV schema has no Requirement ID column, preserve traceability via Test Scenario or another configured mechanism — never lose it silently. Never invent requirement IDs.

## 17. Handling Missing Information

Known behavior → create the case from available evidence. Unknown behavior → do not invent; mark `Testability Status: Blocked / Requires Clarification`. Examples: unknown max value, unspecified error message, unknown required role, undefined file formats, undocumented actor×action behavior, unconfirmed state transition.

## 18. Domain Adaptation

Domain-agnostic by default; adapt coverage to context using evidence. Potential (not mandatory) concerns: Healthcare → privacy, clinical workflow. E-commerce → inventory, pricing, order state. Financial → transaction integrity, authorization. SaaS → tenant isolation, subscription state. Logistics → shipment/delivery state. Never assume domain risk without context.

## 19. Final Quality Checks

**Testability**: executable without guessing? Result observable?
**Data**: concrete? Matches scenario? No sensitive values?
**Coverage**: happy path, negative, boundary, edge cases, security/permission (3.5.1), state transitions (3.7), validation — covered where applicable?
**Quality**: no duplicates; risk/priority consistent; Requirement ID traceable; split-vs-E2E (4.1) justified; no unsupported assumptions.
**Format**: valid TC IDs; valid CSV escaping; required columns present or omission noted.

## 20. Strict Rules

1. Never use vague test data when concrete data is reasonable.
2. Never fabricate business rules, validation rules, limits, permissions, state transitions, or expected behavior.
3. Coverage categories mandatory only when applicable.
4. Prioritize risk and meaningful coverage over quantity.
5. Every case has a clear, observable expected result.
6. Steps must be executable without guessing.
7. No duplicate/meaningless variations.
8. No real credentials, secrets, or sensitive PII in test cases.
9. Never conflate Risk Level and Priority.
10. Never assume domain-specific risk without project context.
11. Never invent exact validation messages not documented/observed.
12. Use `Not Observed` / `Not Applicable` / `Requires Clarification` instead of guessing.
13. No test cases for unknown requirements — record the clarification needed instead.
14. Preserve existing TC IDs when modifying cases.
15. CSV must be standards-compliant, not visually-formatted pseudo-CSV.
16. Manual test case generation stays independent from automation implementation unless explicitly requested.
17. Never assume actor×action behavior or state transition existence without evidence — use `Requires Clarification`.
18. Preserve Requirement ID traceability whenever a requirements spec is available.
