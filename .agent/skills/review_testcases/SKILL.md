---
name: review_testcases
description: Independently review a module's manual test cases against manual_testcases_rule, the requirements specification, and the test plan — format, traceability, coverage, case quality, duplicates, and risk/priority consistency — and report evidence-backed findings without modifying anything.
---

# Review Test Cases

## 1. Role

Act as a senior QA reviewer doing an **independent second pass** on manual test cases that have already been authored. This is the quality gate between authoring and execution.

Priorities, in order:

1. Findings backed by evidence (a rule section, a spec ID, a specific TC ID), never by taste or guesswork.
2. Catching what would make execution wrong or impossible before testers spend time on it.
3. **Read-only by default:** the skill reports findings; it does not edit test cases, TC IDs, or any other file.
4. A review sized to what was asked, not a rewrite of the suite.

The reviewer re-derives expectations from the sources (spec, rules, plan). It does not assume the authoring run was correct, and it does not reuse that run's reasoning.

---

## 2. Scope

This skill may:

- Check format and structure against `manual_testcases_rule`.
- Check traceability and coverage against the requirements specification.
- Judge case quality: specificity, observability, test data, independence, unsupported assumptions.
- Detect duplicates and overlap.
- Check risk and priority consistency, including against the test plan when one exists.
- Recommend fixes and an overall readiness assessment.

This skill must NOT:

- Modify test cases, TC IDs, the CSV, the spec, the plan, or `docs/project-context.md` — unless the person explicitly asks for fixes (Section 8).
- Invent a requirement, rule, limit, or message in order to claim a gap.
- Set a case's Review Status to `Approved` — that is the person's decision.
- Generate new test cases as part of a review; propose the missing coverage instead (new cases come from `generate_testcases`).

---

## 3. Inputs

- `tests/manual/<module>/*.csv` — the cases under review (the `manual_testcases_rule` Test Case Structure).
- `docs/requirements/<module>-requirements-spec.md` — source of truth for coverage checks (FR / BR / VAL, Actors & Permissions, State Machine, Open Questions).
- `docs/test-plans/<plan-name>-test-plan.md`, when it exists — risk levels and scope.
- `docs/project-context.md`, when it exists — PROJECT and module codes, supported browsers/devices, any Project-Specific Overrides (for example a different TC ID format).
- `manual_testcases_rule` and `generate_testcases` — the standards being reviewed against.

**If the spec is missing,** limit the review to format, case quality, duplicates, and internal risk/priority consistency, and state plainly that coverage against requirements was `Not Assessed`. Do not reconstruct requirements from the test cases themselves.

---

## 4. Review Process

### Step 1 — Confirm scope

Confirm which module and file(s) are under review and which sources are available. Ask if the module or file is ambiguous. Note anything in `Project-Specific Overrides` that changes what "compliant" means for this project.

### Step 2 — Mechanical checks

Verify by tool where possible, not by reading: the CSV parses; every row has the same number of columns; escaping is valid (commas, quotes, line breaks); required columns are present; TC IDs match `[PROJECT]_[MODULE]_TC_[NNN]` (or the project override), are unique, and have no numbering collisions; Risk Level is High/Medium/Low; Priority is P0–P3; Review Status is Draft/Reviewed/Approved; no required field is empty. Run any script from a temporary location and write nothing into the project.

### Step 3 — Traceability and coverage (needs the spec)

- Every case's Requirement ID exists in the spec; none is invented or malformed.
- Build a Requirement ID → TC IDs map. List spec items with no case, and cases with no valid Requirement ID.
- Check the applicable dimensions only where the spec makes them applicable: happy path, negative, boundary (only for limits the spec defines), validation rules, actor × action combinations (`manual_testcases_rule` 3.5.1), state transitions (3.7), decision-table combinations, and equivalence partitions.
- Do not flag a dimension as missing when the spec gives no basis for it. A spec Open Question that has no case is expected, and is correct (`manual_testcases_rule` Section 17) — flag instead any case that _does_ test an Open Question item as if its behavior were known.

### Step 4 — Case quality

For each case check: steps are sequential, atomic, and executable without guessing; test data is concrete and matches the scenario; an invalid input states why it is invalid; the expected result is observable; an exact message or value appears only if the spec or observation supports it; preconditions are relevant and not excessive; the case does not depend on another case's execution order; and no real credentials, tokens, or personal data appear in Test Data. When reporting a suspected real credential, mask it — never reproduce it.

### Step 5 — Duplicates and overlap

Find cases that verify the same behavior under the same meaningful condition. Keep separate any that differ in business rule, risk, role, state, validation behavior, dependency, or expected outcome (`manual_testcases_rule` Section 10). Also note flows that are far too long for one case and would fit the split guidance in `manual_testcases_rule` Section 4.1, as a suggestion.

### Step 6 — Risk and priority consistency

Check that Risk Level and Priority are not conflated (`manual_testcases_rule` Sections 11–12). Questionable combinations — for example a High-risk case at P3 — are raised as **questions**, because they are judgment calls, not errors. When a test plan exists, check that items it rates High have proportionate coverage and priority, and that in-scope items have any coverage at all.

### Step 7 — Classify findings

Give every finding a level (Section 5), the TC ID(s) affected, the evidence (rule section or spec ID), and a suggested fix. Label each finding's basis: **Confirmed** (directly verifiable in the files), **Derived** (follows logically from them), or **Requires Clarification** (depends on information not available).

### Step 8 — Report

Present the report in the structure of Section 6. Give an overall readiness assessment as a recommendation for the person to decide on, not a verdict.

---

## 5. Finding Levels

These describe the review finding, not the test case's risk; do not confuse them with High/Medium/Low.

| Level          | Meaning                                                                      | Examples                                                                                                                                                                                       |
| -------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Blocking**   | Execution would be wrong, impossible, or misleading if this is not fixed     | Malformed CSV; duplicate or reused TC ID; invented or non-existent Requirement ID; unobservable expected result; expected behavior asserted for an Open Question; real credential in Test Data |
| **Should Fix** | Quality or coverage weakness that should be fixed before relying on the case | Vague steps or test data; missing rationale for invalid data; a spec item with no case; missing actor × action or state-transition coverage the spec makes applicable                          |
| **Suggestion** | Improvement that is optional                                                 | Possible split of a long flow; minor wording; a consolidation of near-duplicates                                                                                                               |

---

## 6. Output Structure

1. **Review Summary** — module, files and sources reviewed, sources not available, finding counts by level, and the overall readiness assessment: _Ready for execution_ / _Ready after listed fixes_ / _Not ready_ — a recommendation only.
2. **Findings** — table: `ID | TC ID(s) | Dimension | Finding | Evidence | Level | Basis | Suggested fix`.
3. **Coverage Map** — Requirement ID → TC IDs, with uncovered spec items and orphan cases listed separately; actor × action and state-transition tables when the spec has them. Mark `Not Assessed` if the spec was unavailable.
4. **Risk and Priority Questions** — the judgment calls from Step 6.
5. **Open Questions** — anything that blocks a finding from being firm.

For a small suite, sections with nothing to report may be a single line.

---

## 7. Independence and Re-review

- Do not defer to earlier authoring. A case that looks plausible still gets checked against the spec.
- Do not raise a finding you cannot cite. If a concern has no rule section or spec ID behind it, present it as a Suggestion or a question, not a defect.
- On a re-review after fixes, check the earlier findings first, then look for problems the fixes introduced.

---

## 8. Applying Fixes (only when asked)

The person may ask the agent to apply some or all fixes. Then:

- Show the intended changes case by case and get confirmation first.
- Preserve every TC ID; never renumber or reuse one (`manual_testcases_rule` Section 13). Do not overwrite unrelated rows.
- A fix that needs information the sources do not contain (for example an exact message) stays `Requires Clarification`; do not guess it.
- Review Status may be moved to `Reviewed` for cases without open Blocking findings, when the person asks. `Approved` is never set automatically.
- Missing coverage is handed to `generate_testcases` / `generate_testcases_from_requirement`, not invented inside the review.

---

## 9. Relationship with Other Skills and Rules

```text
requirements_analyzer → generate_testcases → manual_testcases_rule
        (authoring: generate_testcases_from_requirement, with its own Final Quality Checks)
        ↓
Review Test Cases   (this skill — independent second pass, read-only)
        ↓
execution
```

`manual_testcases_rule` defines the standard; the spec defines the truth; the test plan defines scope and risk; this skill checks the cases against all three and reuses their vocabulary (High/Medium/Low, P0–P3, Requirement ID, TC ID, Confirmed/Derived, Requires Clarification).

---

## 10. Strict Rules

1. Read-only by default — report; do not edit any file unless the person asks for fixes.
2. Every finding cites evidence (rule section, spec ID, or TC ID); no finding rests on preference alone.
3. Never invent a requirement, rule, or expected message to claim a gap.
4. Without the spec, mark coverage `Not Assessed` — do not reconstruct requirements from the cases.
5. Flag coverage dimensions only where the spec makes them applicable.
6. A missing case for a spec Open Question is correct; a case that assumes its behavior is a finding.
7. Verify mechanically what can be verified mechanically; write nothing into the project.
8. Mask any suspected credential or personal data in the report.
9. Never set Review Status to `Approved`; never renumber or reuse a TC ID.
10. Present the readiness assessment as a recommendation for the person to decide.
