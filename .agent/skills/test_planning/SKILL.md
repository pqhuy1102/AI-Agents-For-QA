---
name: test_planning
description: Draft an evidence-based test plan for a project, release, or test cycle — scope, risk assessment, test approach, entry/exit criteria, environments, deliverables, assumptions and open questions — for a human to review and confirm.
---

# Test Planning

## 1. Role

Act as a QA lead drafting a test plan. **The skill drafts; the person decides.** Scope boundaries, risk appetite, and exit-criteria thresholds are business and team decisions, so this skill proposes them with evidence and marks them for confirmation. It never presents a draft as an approved plan.

Priorities, in order:

1. Scope and risk backed by evidence, not assumption.
2. Criteria that can be checked objectively later (the Test Summary Report will evaluate results against them).
3. A clear line between what the person confirmed and what the skill only proposed.
4. A plan sized to the project or release, not padded.

---

## 2. Scope

This skill may:

- Define what is in and out of scope, with reasons.
- Assess risk per in-scope item on the shared High/Medium/Low scale.
- Choose the test approach per module or risk tier.
- Propose entry, exit, suspension, and resumption criteria.
- List environments, test data needs, roles, deliverables, assumptions, and open questions.

This skill must NOT:

- Invent numeric thresholds (pass rate, coverage percentage, maximum open defects), dates, staffing, environments, or business risk.
- Decide scope on its own — scope comes from the person's stated release/cycle content.
- Generate test cases (owned by `generate_testcases`) or edit requirements specs, test cases, or `docs/project-context.md`.
- Present a plan as approved before the person confirms it.

---

## 3. Inputs

- `docs/project-context.md` — read first: PROJECT code, modules (slug and code), environments, tech stack, supported browsers/devices, Jira project key.
- Release or cycle information from the person: name, dates if any, features/tickets in scope, size of the change, team and constraints.
- `docs/requirements/<module>-requirements-spec.md`, when it exists: complexity signals, Actors & Permissions, State Machine, Open Questions.
- `tests/manual/<module>/*.csv`, when it exists: current coverage.
- Defect history, incident notes, or risk information, if the person provides them.

A plan may be drafted before requirements analysis has happened. In that case risk is coarser (based on release scope only) and should be refined once specs exist. Missing inputs are recorded as `Not Provided` or asked about, never filled in.

---

## 4. Process

### Step 1 — Plan type and name

Decide whether this is a **master plan** (project-level, long-lived) or a **release/cycle plan**. Ask if unclear. Pick a short name for the file.

### Step 2 — Scope

List in-scope items mapped to module slugs from `docs/project-context.md`, and out-of-scope items with a reason each. Scope is taken from what the person stated. An item whose status is unclear becomes an Open Question, not a silent inclusion or exclusion.

### Step 3 — Risk assessment

Assign each in-scope item a **High / Medium / Low** level, using the same scale as `manual_testcases_rule` Section 11. Record the basis (business impact, data sensitivity, complexity, size of change, integrations, defect history) and label the basis **Confirmed**, **Derived**, or **Assumption**. Without business-provided risk information, say the level is a functional-risk estimate, as `generate_testcases` Step 3 does. Add the resulting emphasis (deeper coverage, early execution, automation candidate).

### Step 4 — Test approach

For each module or risk tier, state which activities apply: manual functional testing through the existing pipeline (`requirements_analyzer` → `generate_testcases` → `manual_testcases_rule`), regression, and API/UI automation for stable, high-frequency cases via `automate_from_manual_testcase`. Add security/permission testing where an Actors & Permissions section exists. Non-functional testing (performance, accessibility, security) is included only when there is a documented requirement or the person asks; otherwise it is an Open Question. Do not add a test type just because it exists.

### Step 5 — Entry criteria

Conditions that must hold before execution starts. Each must be objectively checkable. Typical candidates drawn from pipeline artifacts: a requirements spec exists with no coverage-blocking Open Questions; test cases are authored (and reviewed, once a review step exists); the test environment is available and stable; test data and accounts are ready (credential locations by name only); the build or version under test is identified.

### Step 6 — Exit, suspension, and resumption criteria

- **Exit criteria** describe when testing can be considered complete, for example: all planned High-risk cases executed; no open defect at a severity the team decides blocks release; every in-scope Requirement ID has an executed test case or a recorded reason; remaining Open Questions and unexecuted cases explicitly accepted by the decision-maker.
- **Numeric thresholds** are never invented. Write `Threshold: Requires Confirmation`, and offer a value only as a clearly labelled **Suggestion**.
- **Suspension criteria** (for example a blocking defect or an unavailable environment) and **resumption criteria** are stated the same way.
- Criteria that refer to results use the vocabulary from the Global Rule Section 9: Passed / Failed / Skipped / Blocked / Not Executed / Unknown.

### Step 7 — Environments, data, roles, schedule

Take environments, tech stack, and browsers/devices from `docs/project-context.md`. Record roles and schedule only as the person provides them; otherwise `Not Provided`. Defect tracking uses the Jira project key from the context file. Never put credentials in the plan.

### Step 8 — Deliverables and traceability

List the expected artifacts by stage: requirements specs, test case CSVs, execution results, defect list, test summary report. State that the Test Summary Report is meant to evaluate results against Step 6, which is why those criteria must be written so they can be checked.

### Step 9 — Consistency review

Before presenting, verify: every in-scope item has a risk level and an approach; every exit criterion is objectively checkable; no numeric threshold, date, or staffing detail was invented; each item is labelled Confirmed or Proposed; module slugs match `docs/project-context.md`; unknowns are recorded as `Not Provided` or Open Questions.

### Step 10 — Present, confirm, save

Show the full draft and ask for confirmation. Save to `docs/test-plans/<plan-name>-test-plan.md` only after confirmation (create `docs/test-plans/` if missing). The plan's status stays **Draft** until the person confirms it. If the file already exists, show a section-level diff and confirm before overwriting. Create or modify only this file.

---

## 5. Output Structure

```markdown
# Test Plan: <plan name>

Status: Draft | Confirmed (<date>, confirmed by the person)
Plan type: Master | Release/Cycle
Project: <PROJECT code> Release/cycle: <name or Not Provided>

## 1. Scope

| Item (module slug) | In/Out | Reason | Status (Confirmed/Proposed) |

## 2. Risk Assessment

| Item | Risk Level | Basis | Basis label | Test emphasis |

## 3. Test Approach

| Module / risk tier | Activities | Notes |

## 4. Entry Criteria

- <checkable criterion> — Confirmed | Proposed

## 5. Exit, Suspension, and Resumption Criteria

- <checkable criterion> — Confirmed | Proposed
- Threshold: Requires Confirmation (Suggestion: <value>, optional)

## 6. Environments and Test Data

- <from project-context, credential locations by name only>

## 7. Roles and Schedule

- <as provided, otherwise Not Provided>

## 8. Deliverables

- <artifact → stage>

## 9. Assumptions and Open Questions

| ID | Item | Type (Assumption / Open Question) | Impact |
```

For a small release, sections that add nothing (for example Roles and Schedule) may be one line reading `Not Provided`.

---

## 6. Relationship with Other Skills and Rules

```text
project_onboarding  →  docs/project-context.md
        ↓
Test Planning   (this skill — scope, risk, approach, entry/exit criteria)
        ↓
requirements_analyzer → generate_testcases → manual_testcases_rule
        ↓
(execution, defects)  →  Test Summary Report evaluates results against this plan's exit criteria
```

The existing workflows do not read the plan automatically; point the agent at `docs/test-plans/<plan-name>-test-plan.md` explicitly when it should inform scope or risk. This skill reuses the shared vocabulary: High/Medium/Low risk, Requirement ID, the result statuses of the Global Rule, and the Confirmed/Derived/Assumption labels.

---

## 7. Strict Rules

1. Never invent numeric thresholds, dates, staffing, environments, or business risk; use `Requires Confirmation`, `Not Provided`, or an Open Question.
2. Never decide scope on your own; it comes from the person.
3. Never present a draft as an approved plan; status stays Draft until the person confirms.
4. Every exit criterion must be objectively checkable.
5. Label each item Confirmed or Proposed, and each risk basis Confirmed, Derived, or Assumption.
6. Use the shared High/Medium/Low risk scale and the Global Rule result vocabulary.
7. Never put credentials or secrets in the plan; reference their location by name only.
8. Do not add test types or sections that the evidence does not call for.
9. Modify only the plan file; never edit requirements specs, test cases, or `docs/project-context.md`.
10. Confirm with the person before overwriting an existing plan.
