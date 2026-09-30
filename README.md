# QA Automation Agent — README

An Antigravity agent setup that runs a full manual-to-automated QA testing lifecycle: requirements → test design → test cases → execution → defects → automation → release reporting. Project-agnostic — built to be dropped into any project, not tied to one codebase or stack.

For the detailed lifecycle map, current status, and roadmap, see **`docs/testing-strategy.md`**. This README is the quick-reference entry point.

---

## 1. The Four-Layer Model

| Layer | Answers | Lives in | Auto-loaded? |
|---|---|---|---|
| **Rule** | What are the limits and standards? | `.agent/rules/*.md` (per project) or `~/.gemini/GEMINI.md` (global) | Yes — always active |
| **Skill** | What can the agent do, and how? | `.agent/skills/<name>/SKILL.md` | On demand, when relevant |
| **Workflow** | In what order, with which checkpoints? | `.agent/workflows/*.md` | On demand — type `/workflow-name` or describe the task |
| **Plan** | How does it all fit together across the whole process? | `docs/testing-strategy.md` | Never auto-loaded — point the agent at it explicitly |

**Rules** are always-on guardrails (max 12,000 characters each). **Skills** are reusable capabilities the agent draws on. **Workflows** are step-by-step procedures that call skills and respect rules, with named human checkpoints. **Plans** are the map showing which stage of the process each of the above belongs to.

---

## 2. What's Built

### Global Rule (applies to every project)
| File | Purpose |
|---|---|
| `GEMINI.md` (`~/.gemini/`) | Communication, evidence discipline, scope discipline, destructive-action and secrets safety, git safety, validation, uncertainty handling — the baseline every other file sits on top of. |

### Rules (`.agent/rules/`)
| File | Purpose |
|---|---|
| `manual_testcases_rule.md` | Standard for writing, formatting, and ID-ing manual test cases (TC ID, Risk Level, Priority, CSV rules). |
| `api_automation_testing_rule.md` | Architecture, assertions, safety, and auto-fix guardrails for API test automation. |
| `ui_automation_testing_rule.md` | Same, for UI/E2E automation — Page Object Model, locators, wait/retry, flaky-test handling. |
| `non_functional_testing_rule.md` | Performance, accessibility, and infrastructure-adjacent security testing — only runs on an explicit trigger, never by default. |
| `bug_report_rule.md` | Standard for a reproducible bug report — Severity vs. Priority kept separate, traceable to its source TC ID. |

### Skills (`.agent/skills/`)
| Skill | Purpose |
|---|---|
| `requirements_analyzer` | Turns page/app evidence into a structured Requirements Specification (FR/BR/Field Specs/Actors & Permissions/State Machine). |
| `generate_testcases` | Selects test design techniques (Equivalence Partitioning, Boundary Value Analysis, Decision Table, State Transition) and designs a minimal, risk-based set of test scenarios. |
| `automate_testcases` | Turns one manual test case into automation code — candidate check, track selection, architecture mapping, traceability. |
| `review_testcases` | Read-only second pass on authored test cases — format, coverage, duplicates, risk/priority consistency. |
| `test_planning` | Drafts scope, risk assessment, test approach, and entry/exit criteria for a project or release. |
| `test_summary_report` | Evaluates a cycle's results against the confirmed test plan's exit criteria; produces a recommendation, never a decision. |
| `exploratory_testing` | Charter-driven, time-boxed exploratory session — finds what scripted tests don't reach. |

### Workflows (`.agent/workflows/`)
| Workflow | What it orchestrates |
|---|---|
| `project_onboarding` | Captures PROJECT code, modules, environments, tech stack once into `docs/project-context.md`, so other workflows stop re-asking. |
| `generate_testcases_from_requirement` | `requirements_analyzer` → `generate_testcases` → `manual_testcases_rule`, end to end. |
| `automate_from_manual_testcase` | Candidate shortlist → `automate_testcases` → generate, run, and validate automation code → mark the source case "Automated". |
| `track_manual_execution` | Records execution results into a per-cycle log, without touching the source test case file. |
| `log_defect` | Drafts a bug per `bug_report_rule` from a Failed/Blocked result and saves it to a local bug list (Jira filing deferred by choice). |
| `generate_test_summary_report` | Gathers the test plan, execution log, and bug list, runs `test_summary_report`, and saves the final report. |
| `regression_impact_analysis` | Diffs a changed requirements spec and flags which test cases (manual and automated) are now Orphaned, Need Review, or represent a Coverage Gap. |

### Plan
| File | Purpose |
|---|---|
| `docs/testing-strategy.md` | The full lifecycle map — every stage, its status, its human checkpoints, and the roadmap. The one file that shows how everything above connects. |

---

## 3. How to Set Up

**Global Rule (once, applies to all projects):**
1. Antigravity → **•••** (top-right of Agent Manager) → **Additional options** → **Customizations** → **Rules** tab → **+ Global**.
2. Paste the content of `GEMINI.md`. (Or place the file directly at `~/.gemini/GEMINI.md`.)

**Per project (repeat for each project you use this on):**
1. Create `.agent/rules/`, `.agent/skills/<name>/`, `.agent/workflows/` in the project root.
2. Copy each Rule file into `.agent/rules/`, each Skill's `SKILL.md` into its own folder under `.agent/skills/`, each Workflow file into `.agent/workflows/`.
3. Copy `testing-strategy.md` into `docs/` (create the folder if missing).
4. Run `/project_onboarding` first — everything downstream depends on `docs/project-context.md` existing.

Rules and Workflows are capped at 12,000 characters each in Antigravity — all files here already respect that.

---

## 4. Cheat Sheet — "I want to..."

| You want to... | Run |
|---|---|
| Set up a new project for the first time | `/project_onboarding` |
| Define scope/risk/exit criteria for a release | `test_planning` skill |
| Turn a page/feature into requirements | `requirements_analyzer` skill, or `/generate_testcases_from_requirement` for the full pipeline |
| Get manual test cases from a requirements spec | `/generate_testcases_from_requirement` |
| Sanity-check test cases before execution | `review_testcases` skill |
| Automate an existing manual test case | `/automate_from_manual_testcase` |
| Record results after running tests | `/track_manual_execution` |
| Log a bug from a failed test | `/log_defect` |
| Run a performance/accessibility/security check | `non_functional_testing_rule` (only if a real trigger exists — see the rule's Section 1) |
| Do unscripted, exploratory testing | `exploratory_testing` skill |
| Produce an end-of-cycle report | `/generate_test_summary_report` |
| Check what a requirements change affects | `/regression_impact_analysis` |
| Understand the whole process / what's missing | Read `docs/testing-strategy.md` |

---

## 5. Pipeline at a Glance

```text
project_onboarding → docs/project-context.md
        │
test_planning → docs/test-plans/<plan>-test-plan.md  (scope, risk, exit criteria)
        │
requirements_analyzer → docs/requirements/<module>-requirements-spec.md
        │
generate_testcases  →  manual_testcases_rule  →  tests/manual/<module>/<PROJECT>_<MODULE>_TC.csv
        │                                                   │
   review_testcases  (quality gate)                automate_testcases → automated code
        │                                                   │
track_manual_execution → executions/<cycle>-execution.csv   │
        │                                                   │
   log_defect → tests/bugs/<module>/..._BUG.csv             │
        │                                                   │
        └──────────────► generate_test_summary_report ◄─────┘
                                    │
                        docs/reports/<cycle>-test-summary-report.md
                                    │
                          Human release decision

regression_impact_analysis: triggered separately, whenever a requirements
spec changes — flags impacted cases on both the manual and automated sides.
```

---

## 6. File Locations

```text
~/.gemini/GEMINI.md                          Global Rule

<project>/
├── .agent/
│   ├── rules/          (5 files)
│   ├── skills/          (7 folders)
│   └── workflows/       (7 files)
├── docs/
│   ├── project-context.md
│   ├── testing-strategy.md
│   ├── test-plans/
│   ├── requirements/
│   └── reports/
└── tests/
    ├── manual/<module>/
    │   ├── <PROJECT>_<MODULE>_TC.csv
    │   └── executions/<cycle>-execution.csv
    └── bugs/<module>/<PROJECT>_<MODULE>_BUG.csv
```

---

## 7. Worked Example — Applying This to a Real Project

Scenario: **"ShopEase"**, an e-commerce web app. You're the QA/SDET bringing this agent setup in for the first time. Below is a realistic session, module by module.

### Step 0 — One-time setup
Global Rule installed once (Section 3). Skill/Rule/Workflow files copied into `shopease/.agent/`.

### Step 1 — Onboard the project
```
/project_onboarding
```
The agent inspects the repo (finds a React frontend, a Java backend, `package.json`, an existing `tests/` folder), then asks one grouped question for what it can't detect. You answer:
```
PROJECT: SHOP
Modules: checkout, cart, product-search
Jira key: SHOP
Environments: staging - https://staging.shopease.test
Automation: none yet, considering Playwright + TypeScript
Browsers: Chrome, Safari (desktop only for now)
```
→ Saves `docs/project-context.md`. Every workflow after this reads it automatically instead of asking again.

### Step 2 — Plan the release
```
Draft a test plan for the "Checkout Redesign" release, scope is the checkout module only.
```
`test_planning` drafts scope (checkout in; cart/search out, with reasons), risk per area (payment step = High, order summary display = Low), entry/exit criteria, and asks you to confirm the thresholds it couldn't invent (e.g. "max open High-severity defects — your call"). You confirm → `docs/test-plans/checkout-redesign-test-plan.md`.

### Step 3 — Analyze requirements
```
Analyze the checkout page at https://staging.shopease.test/checkout for requirements.
```
`requirements_analyzer` inspects the live page, DOM, and linked API calls, and produces FR/BR, Field Specs (card number, expiry, CVV rules it can observe), Actors & Permissions (guest vs. logged-in checkout), and a State Machine (Cart → Checkout → Payment → Confirmed). It flags 2 Open Questions (e.g. "is there a max order quantity? not observed") instead of guessing. → `docs/requirements/checkout-requirements-spec.md`.

### Step 4 — Generate test cases
```
/generate_testcases_from_requirement
```
Reads the spec, designs scenarios (Boundary Value Analysis on quantity fields, Decision Table on guest/logged-in × payment method, State Transition on the checkout flow), formats them per `manual_testcases_rule`, assigns IDs like `SHOP_CHECKOUT_TC_001`. → `tests/manual/checkout/SHOP_CHECKOUT_TC.csv`.

### Step 5 — Review before execution
```
Review the checkout test cases.
```
`review_testcases` flags one duplicate pair and one case with a vague expected result ("payment works" — not observable), and confirms Requirement ID coverage is otherwise complete. You fix the vague one; the duplicate gets merged.

### Step 6 — Execute manually
```
/track_manual_execution
```
You choose mode (b) — results already run by hand — and report them in a batch. Recorded into `tests/manual/checkout/executions/checkout-redesign-cycle1-execution.csv`. One case (`SHOP_CHECKOUT_TC_007`, invalid CVV handling) comes back Failed.

### Step 7 — Log the defect
```
/log_defect
```
Drafts a bug report from `SHOP_CHECKOUT_TC_007`'s Failed result (reproducible steps + your actual result), proposes Severity: High, asks you to confirm Priority. → `tests/bugs/checkout/SHOP_CHECKOUT_BUG.csv` — local, not filed to Jira yet, by design.

### Step 8 — Automate the stable cases
```
/automate_from_manual_testcase
```
Checks each case against the candidate criteria; the guest-checkout happy path and the payment decision-table cases pass (stable, high-frequency, deterministic), while a one-off "abandoned cart after 30 min" case is rejected as a poor candidate. Generates Playwright/TypeScript Page Objects for the ones that pass, runs them, and marks their source TC IDs "Automated".

### Step 9 — Report at cycle end
```
/generate_test_summary_report
```
Evaluates cycle 1 against the confirmed exit criteria: coverage Met, but "zero open High-severity defects" is Not Met because of `SHOP_CHECKOUT_BUG_001`. Recommendation: **"Ready with noted exceptions — one open High-severity defect on invalid CVV handling; recommend fixing before release. This is a recommendation for the release owner to confirm."** → `docs/reports/checkout-redesign-cycle1-test-summary-report.md`.

### Step 10 — Requirements change mid-cycle
Product adds a discount-code field to checkout. You re-run `requirements_analyzer`, then:
```
/regression_impact_analysis
```
Diffs the spec: the discount-code requirement is new (Coverage Gap — no case yet), and the payment-amount field spec changed slightly (Needs Review — 2 existing cases and 1 automated test reference it). You run `generate_testcases_from_requirement` for the gap and `review_testcases` for the flagged cases.

The same sequence applies to any other module or project — only the specifics change.

---

## 8. Design Principles Worth Knowing

* **Shared vocabulary everywhere.** Risk Level (High/Medium/Low), Requirement ID, TC ID, and the result statuses (Passed/Failed/Skipped/Blocked/Not Executed/Unknown) are defined once and reused across every file — no file invents its own competing scale.
* **Evidence over assumption.** Nothing is invented — a missing threshold, an unclear requirement, or an unreproduced bug is labeled `Requires Clarification` / `Not Provided` / `Not Tracked`, never guessed.
* **Recommendations, not decisions.** Release readiness, bug severity-to-priority mapping, and CI suite placement are always presented as recommendations for a human to confirm.
* **Local-first where integration isn't ready yet.** Defect logging currently saves to a local file instead of Jira, by choice — designed so a future Jira-filing workflow can read the same file without redoing anything.
* **Nothing is silently overwritten.** TC IDs, Bug IDs, and prior execution records are never renumbered or replaced without being shown to the person first.