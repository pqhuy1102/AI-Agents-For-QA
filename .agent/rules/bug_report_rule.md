---
trigger: always_on
---

# Bug Report Rule

Mandatory standard for writing a bug report from a Failed or Blocked test execution. Domain-agnostic. A bug report produced under this rule must be usable by a developer to reproduce the issue without needing to ask the reporter anything first.

**Position in the QA lifecycle:** Stage 3.6 (Defect Logging), consuming a Failed/Blocked record from an execution log (`tests/manual/<module>/executions/<cycle-name>-execution.csv`, per `track_manual_execution`). This rule defines the report's standard; a separate Workflow (`log_defect`) applies it to actually save a bug report.

---

## 1. Core Principles

- **Reproducible over described.** Steps to reproduce must be concrete enough that someone unfamiliar with the test case can follow them exactly — not a summary of what the test case does.
- **Expected vs. Actual, kept distinct.** Never blend them into one narrative sentence; a reader should be able to see the gap at a glance.
- **Evidence over assumption.** Never state a root cause unless it was actually diagnosed; report the observed symptom.
- **Severity ≠ Priority.** They answer different questions (Section 4) and must never be used interchangeably, the same principle as `manual_testcases_rule` Section 12.
- **Traceable.** Every bug links back to its source TC ID and Requirement ID — never a freestanding report with no origin.

---

## 2. Required Fields

| Field              | Description                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| Bug ID             | `[PROJECT]_[MODULE]_BUG_[NNN]` — see Section 6                                                              |
| Title              | One line, specific: `<component/action> <fails to/produces> <symptom>` — not "Login doesn't work"           |
| Source TC ID       | The test case that surfaced this bug                                                                        |
| Requirement ID     | Carried from the source TC ID                                                                               |
| Module             | Module slug/code                                                                                            |
| Severity           | Critical / High / Medium / Low — Section 4                                                                  |
| Priority           | P0–P3 — Section 4, same scale as `manual_testcases_rule` Section 12                                         |
| Environment        | Build/version, environment name, browser/device if relevant — from `docs/project-context.md` where possible |
| Steps to Reproduce | Numbered, concrete, executable without guessing (same bar as `manual_testcases_rule` Section 7)             |
| Expected Result    | From the source test case                                                                                   |
| Actual Result      | What was actually observed — the concrete deviation, not a summary                                          |
| Evidence           | Screenshot/log reference — sensitive data masked before attaching (Section 5)                               |
| Status             | New / Confirmed / In Progress / Fixed / Verified / Closed / Reopened / Won't Fix — Section 7                |
| Reporter           | Who/what filed it (the person, or the agent on their behalf)                                                |
| Date Reported      | —                                                                                                           |
| Root Cause         | Optional; filled in once actually diagnosed — never guessed by QA to fill the field                         |

---

## 3. Steps to Reproduce

- Start from a stated precondition (account/role, data state, starting screen) — don't assume the reader knows where to start.
- Each step is one action; number them, don't narrate multiple actions in one sentence.
- Use the concrete test data actually used during execution (from the execution log's Actual Result / the source TC's Test Data) — not a generic placeholder.
- If the bug is intermittent, say so explicitly and report how many attempts produced it out of how many tried, rather than presenting one occurrence as guaranteed reproducibility.
- Do not soften or generalize a step to make the report look cleaner — an exact, slightly awkward step beats a smooth but imprecise one.

---

## 4. Severity vs. Priority

**Severity** — the technical/functional impact of the bug itself, independent of business timing:

| Severity | Meaning                                                                      |
| -------- | ---------------------------------------------------------------------------- |
| Critical | Data loss/corruption, security exposure, core flow completely blocked, crash |
| High     | Major function broken with no reasonable workaround                          |
| Medium   | Function impaired but a workaround exists                                    |
| Low      | Cosmetic, minor, or edge-case-only impact                                    |

**Priority** — how urgently it should be worked on, using the same P0–P3 scale as `manual_testcases_rule` Section 12. Priority is a business/planning decision — the reporter may propose one, but do not assume Critical severity always means P0, or that priority is fixed once set; it can change independently of severity as context changes.

A bug's Priority is not automatically inherited from its source test case's Risk Level — record both if relevant, but let a human confirm the actual Priority rather than deriving it silently.

---

## 5. Evidence and Sensitive Data

Never include, in the bug report or its evidence: passwords, tokens, API keys, session cookies, full card numbers, or other sensitive personal data (Global Rule Section 4.2) — mask them even if they appear naturally in a screenshot or log excerpt. If evidence would require including any of this to be meaningful, describe what was seen instead of attaching the raw evidence, and note that it was withheld and why.

---

## 6. Bug ID Convention

`[PROJECT]_[MODULE]_BUG_[NNN]` — same PROJECT/MODULE codes as `manual_testcases_rule` Section 13's TC ID convention, three-digit sequential numbering, never reused, never renumbered. Preserve an existing Bug ID when updating a report (status change, added evidence) — this mirrors the TC ID preservation rule.

---

## 7. Status Lifecycle

`New` → `Confirmed` → `In Progress` → `Fixed` → `Verified` → `Closed`, with `Reopened` available from `Verified`/`Closed` if the issue recurs, and `Won't Fix` available from `New`/`Confirmed` with a stated reason. Only the person (or whoever owns the tracker once one is wired in) advances a bug past `New`/`Confirmed` — the reporting step does not self-verify or self-close a bug it just filed.

---

## 8. Duplicate Handling

Before filing, check for an existing open bug (not `Closed`/`Won't Fix`) against the same TC ID or the same symptom on a related TC ID. If one exists, do not file a second bug — link the new occurrence to the existing Bug ID (e.g. as an additional observation/evidence entry) instead.

---

## 9. Strict Rules

1. Never state a root cause that wasn't actually diagnosed.
2. Never conflate Severity and Priority, or assign Priority without it being a confirmable decision.
3. Never include unmasked credentials, tokens, or sensitive personal data in a report or its evidence.
4. Steps to Reproduce must be concrete and executable without guessing — no vague placeholders.
5. Every bug must link to a source TC ID and Requirement ID; no freestanding bug reports.
6. Never reuse or renumber an existing Bug ID.
7. Check for a duplicate before filing a new bug; link to the existing one instead of duplicating.
8. Only present an intermittent bug's reproducibility as what was actually observed (N out of M attempts), never as guaranteed.
9. The filing step never advances a bug's Status past `New`/`Confirmed` on its own.
