# QA Testing Strategy & Process Plan

## 1. Purpose

This document is the top-level map of how AI agents and the QA team work together across the **entire manual-to-automated testing lifecycle** — not just test case generation. It sits above the individual Rules, Skills, and Workflows: those answer _what to do_ and _how_; this document answers _when, in which order, and who owns the checkpoint_.

This is a reference document, not an Antigravity Rule/Skill/Workflow file — it is not auto-loaded by the agent. Point the agent at it explicitly (e.g. "follow `docs/testing-strategy.md`") when starting a new cycle, or reference specific stages when only part of the process is relevant.

Current state:

- Execution model: **manual-first, automating gradually** — test cases are designed and formatted manually first; stable, high-value ones are automated per-project once mature (Stage 7). This pipeline is project-agnostic — it doesn't assume any specific codebase, language, or CI stack.
- Defect tracking: **Jira**.
- Project context: **captured once per project** by the `project_onboarding` workflow into `docs/project-context.md`. `generate_testcases_from_requirement` and `automate_from_manual_testcase` read it first and ask only about fields still `Not Provided` (Section 3.0).
- Test planning: **Skill `test_planning` built** — drafts scope, risk, test approach, and entry/exit criteria into `docs/test-plans/<plan-name>-test-plan.md` for a human to confirm (Section 3.1).
- Test case review: **Skill `review_testcases` built** — read-only quality gate over authored test cases (Section 3.4b).
- Manual execution tracking and defect logging: **built, local-first** — Workflow `track_manual_execution` writes a per-cycle execution log to `tests/manual/<module>/executions/<cycle-name>-execution.csv` (never touching the source test case CSV); Rule `bug_report_rule` + Workflow `log_defect` save bug reports to a local `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` instead of Jira for now, by design, so nothing needs redoing once Jira filing is added later. `test_summary_report` reads both of these.
- Test summary / release sign-off: **Skill `test_summary_report` + Workflow `generate_test_summary_report` built** — evaluates results against the confirmed test plan's exit criteria and produces a recommendation, never an autonomous decision (Section 3.9). Now reads real local data when `track_manual_execution`/`log_defect` have been run for the cycle; still marks a module `Not Tracked`/`Not Available` when they haven't.
- Non-functional and exploratory testing: **Rule `non_functional_testing_rule` and Skill `exploratory_testing` built** — both optional layers, invoked only on an explicit trigger (a documented requirement, an explicit ask, or a plan-flagged risk), not run by default.
- Automation governance: **fully built and project-agnostic** — Rules `api_automation_testing_rule` and `ui_automation_testing_rule`, Skill `automate_testcases`, Workflow `automate_from_manual_testcase`. The pipeline from manual test case → automation candidate check → generated, traceable automated test is end-to-end and ready to apply to any project. See Section 3.7.

---

## 2. Full Testing Lifecycle Map

| #   | Stage                                            | Status                                            | Primary Artifact(s)                                                                                                                        |
| --- | ------------------------------------------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 0   | Project Onboarding (one-time context setup)      | ✅ Built                                          | Workflow: `project_onboarding` → `docs/project-context.md`                                                                                 |
| 1   | Test Planning (scope, risk, entry/exit criteria) | ✅ Built                                          | Skill: `test_planning` → `docs/test-plans/<plan-name>-test-plan.md`                                                                        |
| 2   | Requirements Analysis                            | ✅ Built                                          | Skill: `requirements_analyzer`                                                                                                             |
| 3   | Test Design                                      | ✅ Built                                          | Skill: `generate_testcases`                                                                                                                |
| 4   | Test Case Authoring & Formatting                 | ✅ Built                                          | Rule: `manual_testcases_rule`; Workflow: `generate_testcases_from_requirement`                                                             |
| 4b  | Test Case Review (quality gate before execution) | ✅ Built                                          | Skill: `review_testcases`                                                                                                                  |
| 5   | Manual Test Execution                            | ✅ Built                                          | Workflow: `track_manual_execution` → `tests/manual/<module>/executions/<cycle>-execution.csv`                                              |
| 6   | Defect Logging (local-first)                     | ✅ Built                                          | Rule: `bug_report_rule`; Workflow: `log_defect` → `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv`                                         |
| 7   | Automation (gradual)                             | ✅ Built, project-agnostic                        | Rules: `api_automation_testing_rule`, `ui_automation_testing_rule`; Skill: `automate_testcases`; Workflow: `automate_from_manual_testcase` |
| —   | Non-Functional Testing (optional layer)          | ✅ Built, trigger-only                            | Rule: `non_functional_testing_rule`                                                                                                        |
| —   | Exploratory Testing (optional complement)        | ✅ Built                                          | Skill: `exploratory_testing`                                                                                                               |
| 8   | Regression Suite Maintenance                     | ✅ Built                                          | Workflow: `regression_impact_analysis`                                                                                                     |
| 9   | Test Summary Report / Release Sign-off           | ✅ Built — data completeness depends on Stage 5/6 | Skill: `test_summary_report`; Workflow: `generate_test_summary_report`                                                                     |

Legend: ✅ built and in use · 🔶 exists but outside this AI pipeline · ⚠️ happens but not formalized · ❌ not started

Global safety/behavior constraints (Section 4-16 of the Global Rule) apply to every stage below at all times and are not repeated per stage.

---

## 3. Stage-by-Stage Detail

### 3.0 Project Onboarding

**Goal:** Capture project-level context once, so every later workflow reads it instead of re-asking the same questions.

**Current state:** Built. The `project_onboarding` Workflow creates `docs/project-context.md` for a project — PROJECT code (used in TC IDs), module list (slug and code), Jira project key, test environments/URLs (no credentials), tech stack and automation framework (if any), supported browsers/devices, and known constraints. `generate_testcases_from_requirement` and `automate_from_manual_testcase` read this file first and only ask about what is missing. The workflow can be re-run in update mode when modules, environments, or the stack change. Other workflows never edit the file themselves.

**Constraint that applies:** Global Rule Section 4.2/4.3 — the context file must never contain credentials, tokens, or secrets; reference where they are stored (env var name, secret manager) instead of the value.

**Human checkpoint:** The person confirms the captured context before it is saved, and confirms any later change to it.

---

### 3.1 Test Planning

**Goal:** Define scope, risk appetite, and entry/exit criteria for a test cycle before requirements analysis starts.

**Current state:** Built. Skill `test_planning` drafts a master (project-level) or release/cycle test plan: scope (in/out), risk assessment on the High/Medium/Low scale, test approach, entry/exit/suspension/resumption criteria, environments, deliverables, assumptions, and open questions. Output goes to `docs/test-plans/<plan-name>-test-plan.md`, and its status stays Draft until the person confirms it. Numeric thresholds (pass rate, maximum open defects, coverage) are never invented — they are marked `Requires Confirmation`.

**Owner:** Human (QA lead / the person driving the cycle). The skill drafts and proposes with evidence; scope, risk appetite, and exit-criteria thresholds are decided by the person.

**Entry point into the AI pipeline:** The scope decided here becomes the `<module>` and evidence sources fed into Step 1-2 of the `generate_testcases_from_requirement` workflow. That workflow does not read the plan automatically yet; point the agent at the plan file when it should inform scope or risk.

**Downstream use:** The exit criteria written here are what the Test Summary Report (3.9) evaluates results against, which is why the skill requires every exit criterion to be objectively checkable.

---

### 3.2 Requirements Analysis

**Goal:** Turn available evidence (live app, docs, API behavior) into a structured, evidence-based Requirements Specification.

**Owned by:** Skill `requirements_analyzer`.

**Output:** `docs/requirements/<module>-requirements-spec.md` (FR / BR / Field Specs / Actors & Permissions / State Machine / Open Questions, scaled to feature complexity).

**Human checkpoint:** Any Open Question that would materially change coverage must be resolved or explicitly accepted as an assumption before moving to Test Design (workflow Step 5).

---

### 3.3 Test Design

**Goal:** Select test design techniques (Equivalence Partitioning, Boundary Value Analysis, Decision Table, State Transition) and produce a minimal, risk-aware set of candidate scenarios.

**Owned by:** Skill `generate_testcases`, consuming the spec from 3.2 directly (Actors & Permissions → Decision Table; State Machine → State Transition).

**Output:** Test Design Summary + candidate test cases + Coverage Matrix + design-level Open Questions.

**Human checkpoint:** Review the Test Design Summary before final formatting (workflow Step 8) — catches design issues before they're baked into formatted test cases.

---

### 3.4 Test Case Authoring & Formatting

**Goal:** Format candidate scenarios into standardized, execution-ready manual test cases.

**Owned by:** Rule `manual_testcases_rule` (structure, IDs, risk/priority scale, CSV rules); orchestrated end-to-end by Workflow `generate_testcases_from_requirement`.

**Output:** `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv`.

**Human checkpoint:** Existing TC IDs are never silently overwritten (workflow Step 13) — any apparent update to an existing case is shown to the user for confirmation.

---

### 3.4b Test Case Review

**Goal:** Act as a quality gate between authoring and execution: check generated test cases for duplicates, coverage gaps against the source requirements, vague steps or expected results, invented rules or messages, and deviations from `manual_testcases_rule`.

**Current state:** Built. Skill `review_testcases` reads `tests/manual/<module>/*.csv` together with the requirements spec (and the test plan's risk assessment, when confirmed) and produces a review report: mechanical checks, traceability/coverage, case quality, duplicates, and Risk/Priority consistency. Findings are leveled Blocking / Should Fix / Suggestion. Read-only by default — it reports findings; it applies fixes only when the person explicitly asks, and never sets Review Status to `Approved` itself.

**Human checkpoint:** Findings are presented to the person before any test case is modified.

---

### 3.5 Manual Test Execution

**Goal:** Execute the formatted manual test cases and record actual results.

**Current state:** Built. Workflow `track_manual_execution` supports the agent executing steps itself or the person reporting results already run, and writes a **per-cycle execution log** — `tests/manual/<module>/executions/<cycle-name>-execution.csv` — that never modifies the source `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv`. This design choice (separate log per cycle, not columns bolted onto the design-time CSV) preserves history across re-runs and multiple cycles against the same test cases.

**Result vocabulary:** Passed / Failed / Skipped / Blocked / Not Executed / Unknown (Global Rule Section 9) — used consistently by this workflow and by `test_summary_report`.

---

### 3.6 Defect Logging (Local-First)

**Goal:** Record a bug for each Failed/Blocked result with enough detail to reproduce (steps, data, expected vs. actual, environment), without requiring Jira integration to exist yet.

**Current state:** Built. Rule `bug_report_rule` defines the standard (reproducible steps, Severity kept distinct from Priority, sensitive data masked, traceable to TC ID/Requirement ID, a New→Closed status lifecycle). Workflow `log_defect` applies it and saves to a **local** `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` — deliberately not Jira yet, per the person's choice to defer that integration. The workflow checks for duplicates first and cross-references the bug back into the execution log.

**Forward compatibility:** a future `log_defect_to_jira` workflow can read this same CSV as its source and file each row as a ticket — nothing recorded locally needs to be redone when Jira filing is added.

---

### 3.7 Automation (Gradual)

**Goal:** Progressively automate stable, high-value manual test cases.

**Current state — split by track:**

- **API automation:** ✅ Rule `api_automation_testing_rule` built — architecture layers, assertion standards, auth/data safety, auto-fix/auto-healing safety, and a Traceability Bridge (its Section 11) that defines how a TC ID/Requirement ID and the shared High/Medium/Low risk scale should carry over from a manual case. Framework-agnostic (REST Assured/Java, Playwright API/TS, Requests/Python, Supertest, etc.) — applies to whichever project it's pointed at.
- **UI/E2E automation:** ✅ Rule `ui_automation_testing_rule` built — Page Object Model architecture, locator strategy, wait/retry strategy (race condition vs. slow-app distinction), flaky-test handling, auto-fix/auto-healing safety, artifact handling, CI suite placement (e.g. a fast PR-triggered smoke suite vs. a scheduled full regression suite), and the same Traceability Bridge pattern as the API rule. Framework-agnostic (Playwright, Cypress, Selenium, WebdriverIO, etc.).
- **Design/orchestration layer:** ✅ Skill `automate_testcases` (candidate check, track selection, architecture mapping, assertion/data mapping, traceability annotation for one case at a time) and Workflow `automate_from_manual_testcase` (batch candidate shortlist → generation → execution → failure classification → CI-placement recommendation → marking the source TC ID "Automated") are both built and project-agnostic, completing the bridge from either rule's Traceability Bridge section to actual generated code in whatever codebase the workflow is pointed at.

**Candidate selection principle (defined identically in both rules' Traceability Bridge section, enforced by `automate_testcases` Step 1):** stable (low UI/contract churn), high-frequency in regression, deterministic, not one-off/exploratory.

**Remaining gap — not a build gap, a validation gap:** none of these rules/skill/workflow have yet been run end-to-end against a real project's codebase. First real use should be treated as a light validation pass — confirm the architecture/conventions fit that project's language and stack, and adjust if a genuinely project-specific convention is needed (kept in that project's own workspace-level rule override, not by editing these general rules).

---

### 3.10 Non-Functional Testing (Optional Layer)

**Goal:** Cover performance, accessibility, and non-functional/infrastructure-adjacent security when a project actually needs it — never by default.

**Current state:** Built. Rule `non_functional_testing_rule` applies only on an explicit trigger: a documented requirement (SLA/SLO, WCAG target, security policy), an explicit ask, or a High risk flagged in the confirmed test plan — and records which trigger applied. Performance thresholds, accessibility conformance level, and security severity are never invented; unconfirmed ones are reported as `Baseline`/unconfirmed rather than pass/fail. Functional API security stays owned by `api_automation_testing_rule` Section 7 — this rule doesn't duplicate it.

---

### 3.11 Exploratory Testing (Optional Complement)

**Goal:** Find what scripted test design doesn't reach — unscripted paths, edge interactions, usability friction — through time-boxed, charter-driven sessions.

**Current state:** Built. Skill `exploratory_testing` turns a mission into a charter, keeps a live session log, uses heuristics (boundaries, state/sequence, interruption, data variation, consistency) as prompts, and separates reproducible findings from `Suspected — not reproduced` ones. It does not write formatted test cases or file defects directly — a keeper finding is handed to `generate_testcases`, and a reproducible bug goes through the (not yet built) defect-logging process.

---

### 3.8 Regression Suite Maintenance

**Goal:** Keep the regression suite (manual + automated) current as requirements change — retire obsolete cases, flag cases affected by a changed Business Rule or State Machine.

**Current state:** Built. Workflow `regression_impact_analysis` diffs a requirements spec against its prior version (via git history, or a person-supplied prior version), extracts added/removed/modified Requirement IDs, then searches both `tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv` and — when an automation repo path is known — the automated tests' TC ID/Requirement ID annotations for matches. Each match is classified **Orphaned** (requirement removed), **Needs Review** (requirement modified), or **Coverage Gap** (requirement added, nothing covers it yet).

**Human checkpoint:** This workflow only analyzes and reports (`docs/requirements/<module>-regression-impact-<date>.md`) — it never retires, edits, or regenerates a test case itself. The person chooses which suggested follow-up (a `review_testcases` pass, a `generate_testcases_from_requirement` run) to actually trigger.

---

### 3.9 Test Summary Report / Release Sign-off

**Goal:** At the end of a test cycle, produce a report summarizing what was planned, executed, passed/failed/blocked, outstanding defects (from the local bug list, or Jira once connected), and open risks — as the basis for a release go/no-go decision.

**Current state:** Built. Skill `test_summary_report` evaluates a cycle's data against the confirmed test plan's exit criteria (Met / Not Met / Cannot Assess, each with evidence), surfaces unresolved High-risk items individually even when the aggregate criteria are Met, lists data gaps explicitly, and ends with a recommendation always labeled as a recommendation for a human decision-maker — never an autonomous decision. Workflow `generate_test_summary_report` orchestrates gathering the plan, `tests/manual/<module>/executions/<cycle>-execution.csv`, and `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv`, then saves the confirmed report to `docs/reports/<cycle-name>-test-summary-report.md`.

**Data completeness:** now reads real local data whenever `track_manual_execution` and `log_defect` have actually been run for a cycle. A module without an execution log is still marked `Not Tracked`, and a module without a bug list is marked `Not Available` — the report never guesses either.

---

## 4. Human Checkpoints Summary

Across the whole lifecycle, the AI pauses for human input at these points:

1. Project context (3.0) — confirmed by the person before it is saved or changed.
2. Test plan (3.1) — scope, risk basis, and exit-criteria thresholds are decided by the person; the plan stays Draft until confirmed.
3. Coverage-blocking Open Questions from Requirements Analysis (3.2).
4. Test Design Summary review before formatting (3.3).
5. Any apparent overwrite of an existing TC ID (3.4).
6. Test case review findings (3.4b) — presented before any test case is modified.
7. (Once built) Any Jira ticket the agent proposes filing, before it is actually created — filing tickets is an external-system write action per Global Rule Section 11.
8. Test Summary Report recommendation (3.9) — always presented as a recommendation for a human decision-maker, never as an autonomous go/no-go decision.

---

## 5. Roadmap — What to Build Next

In priority order, based on the current gaps above. Already built and therefore not listed: project onboarding (3.0), test planning (3.1), test case review (3.4b), manual execution tracking (3.5), local-first defect logging (3.6), automation (3.7), regression impact analysis (3.8), non-functional testing (3.10), exploratory testing (3.11), and test summary report (3.9).

1. **Jira filing** — a `log_defect_to_jira` Workflow, when the person is ready for it, that reads `tests/bugs/<module>/<PROJECT>_<MODULE>_BUG.csv` (already the source of truth per `bug_report_rule`) and files each row as a ticket, with human confirmation before each filing.
2. **First real-project validation pass** — once the pipeline is pointed at an actual project, do a light check of the whole chain end-to-end, including that `api_automation_testing_rule`/`ui_automation_testing_rule` fit that project's stack and conventions; capture any genuinely project-specific addition as that project's own workspace-level rule rather than editing these general ones.

With this, every stage in Section 2 is built except the one item above (Jira filing is deferred by choice, not by gap) and the validation pass, which by nature can only happen once real project use begins.

---

## 6. File Map (Reference)

```text
.agent/
├── rules/
│   ├── manual_testcases_rule.md
│   ├── api_automation_testing_rule.md
│   ├── ui_automation_testing_rule.md
│   ├── non_functional_testing_rule.md
│   └── bug_report_rule.md
├── skills/
│   ├── requirements_analyzer/SKILL.md
│   ├── generate_testcases/SKILL.md
│   ├── automate_testcases/SKILL.md
│   ├── review_testcases/SKILL.md
│   ├── test_planning/SKILL.md
│   ├── test_summary_report/SKILL.md
│   └── exploratory_testing/SKILL.md
└── workflows/
    ├── generate_testcases_from_requirement.md
    ├── automate_from_manual_testcase.md
    ├── project_onboarding.md
    ├── track_manual_execution.md
    ├── log_defect.md
    ├── generate_test_summary_report.md
    ├── regression_impact_analysis.md
    └── log_defect_to_jira.md                   (planned — Roadmap #1)

~/.gemini/GEMINI.md              (Global Rule — all projects)

docs/
├── project-context.md
├── test-plans/
│   └── <plan-name>-test-plan.md
├── reports/
│   └── <cycle-name>-test-summary-report.md
└── requirements/
    └── <module>-requirements-spec.md

tests/
├── manual/
│   └── <module>/
│       ├── <PROJECT>_<MODULE>_TC.csv
│       └── executions/
│           └── <cycle-name>-execution.csv
└── bugs/
    └── <module>/
        └── <PROJECT>_<MODULE>_BUG.csv
```

Planned file names are proposals and may change when each item is actually built.
