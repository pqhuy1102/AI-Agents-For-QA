---
name: test_summary_report
description: Compile a test cycle's execution and defect data, evaluate it against the confirmed test plan's exit criteria, and produce a Test Summary Report with an explicit go/no-go recommendation for a human decision-maker — never an autonomous decision.
---

# Test Summary Report

## 1. Role

Act as a QA lead compiling the end-of-cycle summary. **This skill evaluates and reports; it does not decide.** A release go/no-go is always presented as a recommendation for a human, per the Global Rule and `testing-strategy.md` Section 4.

Priorities, in order:

1. Evaluate against the confirmed test plan's actual exit criteria — never invent new ones at report time.
2. Every number and claim traces to a source (a CSV, a Jira query, a CI run) — no estimated pass rates.
3. State data gaps prominently, don't let a clean-looking report imply more certainty than exists.
4. A report someone can act on in a few minutes, not a wall of data.

---

## 2. Scope

This skill may:

- Load the confirmed exit criteria from `docs/test-plans/<plan-name>-test-plan.md`.
- Compile execution results and coverage from `tests/manual/<module>/executions/<cycle-name>-execution.csv` (and automated run results, when available).
- Compile defect status from `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` (or from Jira, once a connector is wired in; or from what the person supplies directly).
- Evaluate each exit criterion as **Met / Not Met / Cannot Assess**, with the evidence.
- Highlight High-risk items specifically, using the test plan's risk assessment.
- Produce a recommendation, explicitly labeled as a recommendation.

This skill must NOT:

- Invent or assume an exit-criteria threshold that the test plan left as `Requires Confirmation` — report it as unresolved instead.
- Fabricate execution or defect data when tracking data is incomplete or missing — report the gap.
- Issue an autonomous go/no-go decision, or word the recommendation so it reads as a decision already made.
- Edit the test plan, test cases, or defect tickets — this skill only reads and reports.

---

## 3. Inputs

- `docs/test-plans/<plan-name>-test-plan.md` — must be **Confirmed** status. If it's still Draft or doesn't exist, say so and ask whether to proceed against a draft (clearly labeled as such throughout the report) or wait for confirmation.
- `docs/project-context.md` — PROJECT code, modules, Jira project key.
- `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` — Requirement ID, Risk Level, Priority per TC ID (design-time data; never carries execution results itself).
- `tests/manual/<module>/executions/<cycle-name>-execution.csv`, when it exists (per `track_manual_execution`) — Execution Status, Actual Result, per TC ID for this cycle.
- Automated test run results, when the automation pipeline (`api_automation_testing_rule` / `ui_automation_testing_rule` / `automate_from_manual_testcase`) has produced any for this cycle.
- `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv`, when it exists (per `bug_report_rule` / `log_defect`) — Bug ID, Severity, Priority, Status, linked TC ID. A Jira connector, once wired in, would supplement or replace this as the defect source without changing how this skill evaluates it.

**Known current gap:** the execution log or bug list may not exist yet for every module/cycle — not every project has run `track_manual_execution` or `log_defect` for this cycle. When a module has no execution log, do not infer results from the TC CSV's Review Status (Reviewed/Approved is not the same as Executed/Passed) — mark execution data for that module `Not Tracked`. When no bug list exists, mark defect data `Not Available`. State that the report's figures are based on whatever partial data was actually supplied (e.g. the person pasting results directly).

---

## 4. Process

### Step 1 — Load exit criteria

Read the confirmed test plan's Section 5 (Exit, Suspension, and Resumption Criteria). Carry over each criterion's threshold as written. A criterion still marked `Requires Confirmation` in the plan stays unresolved here too — report it as **Cannot Assess (threshold unconfirmed)**, don't pick the plan's optional Suggestion value as if it were confirmed.

### Step 2 — Compile execution data

For each in-scope module, tally: total cases, executed, Passed / Failed / Skipped / Blocked / Not Executed / Unknown (the Global Rule Section 9 vocabulary — never a different set of labels). Compute Requirement ID coverage (requirements with at least one executed case vs. total in-scope requirements). Where automated results exist, merge them in by TC ID (avoid double-counting a case that exists both manually and as its automated counterpart — count it once, noting both methods if both ran).

### Step 3 — Compile defect data

Read `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` for each in-scope module. Group by the bug list's own Severity (`bug_report_rule` Section 4) — don't remap it into the QA Risk Level scale unless the person asks; they're different concepts. Exclude bugs with Status `Closed`/`Won't Fix` from "open defects" counts, but still list them if the report should show resolution history. Once a Jira connector is wired in, it supplements or replaces this file as the source without changing this step's logic. If no bug list exists for a module, mark defect data `Not Available` for it — never assume zero defects.

### Step 4 — Evaluate each exit criterion

For each criterion from Step 1, mark **Met**, **Not Met**, or **Cannot Assess**, with the evidence (the actual numbers/queries behind it) and cite which data (Step 2/3) it came from. A criterion referencing data marked `Not Tracked`/`Not Available` is automatically `Cannot Assess` — never guess it Met.

### Step 5 — Risk highlights

List High-risk items (from the test plan's risk assessment) and their individual status — executed/not, passed/failed, any open defect against them — regardless of the overall exit-criteria verdict. A cycle can meet its aggregate exit criteria while still having an unresolved High-risk item; surface that rather than letting the aggregate hide it.

### Step 6 — Data gaps and limitations

List explicitly what could not be assessed and why: untracked execution, unavailable defect data, modules out of scope, an unconfirmed threshold. This section is never empty by default — if truly nothing is missing, say so explicitly rather than omitting the section.

### Step 7 — Recommendation

State a recommendation — **Ready to release / Ready with noted exceptions / Not ready** — built only from Steps 4–6, with the reasoning shown. Label it explicitly: _"This is a recommendation for `<decision-maker>` to confirm, not an automated release decision."_ Never omit this label.

---

## 5. Output Structure

```markdown
# Test Summary Report: <cycle/release name>

Project: <PROJECT code> Test Plan: <plan name> (<Confirmed | Draft — caveat>)
Report date: <date> Modules in scope: <list>

## 1. Overview

- <cycle scope, dates if known, what this report covers>

## 2. Exit Criteria Evaluation

| Criterion | Threshold | Result | Status (Met/Not Met/Cannot Assess) | Evidence |

## 3. Execution Summary

| Module | Total Cases | Executed | Passed | Failed | Skipped | Blocked | Not Executed | Unknown | Req. ID Coverage |

## 4. Defect Summary

| Severity/Priority | Open | Notes |
(or "Not Available — no defect data source connected/supplied")

## 5. Risk Highlights

| Item | Risk Level | Status | Notes |

## 6. Data Gaps and Limitations

- <explicit list, or "None identified">

## 7. Recommendation

<Ready to release | Ready with noted exceptions | Not ready> — reasoning.
This is a recommendation for <decision-maker> to confirm, not an automated release decision.
```

---

## 6. Relationship with Other Skills and Rules

```text
test_planning → docs/test-plans/<plan>-test-plan.md   (exit criteria, risk)
        +
tests/manual/<module>/executions/<cycle>-execution.csv (execution, once tracked)
        +
tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv (defects, local — or Jira once connected)
        ↓
Test Summary Report   (this skill — evaluates, never decides)
        ↓
Human release decision
```

Reuses the shared vocabulary throughout: the Global Rule's result statuses, `manual_testcases_rule`'s Risk Level/Requirement ID, and the test plan's own exit-criteria wording — this skill never introduces a parallel vocabulary.

---

## 7. Strict Rules

1. Never invent or resolve an exit-criteria threshold the test plan left unconfirmed.
2. Never infer execution results from Review Status, or from anything other than actual execution data.
3. Mark a criterion `Cannot Assess` rather than guessing when its underlying data is untracked or unavailable.
4. Never issue the recommendation as anything other than a recommendation — the label is mandatory every time.
5. Never remap Jira's own severity/priority into the QA Risk Level scale without being asked.
6. Always surface unresolved High-risk items individually, even when the aggregate exit criteria are Met.
7. Never leave the Data Gaps section implicit — state "None identified" if genuinely none.
8. Only read; never edit the test plan, test cases, or defect tickets from this skill.
