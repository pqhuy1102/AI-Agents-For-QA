---
trigger: always_on
---

# UI/E2E Automation Testing Rules

Quality, architecture, safety, and execution standards for automated UI/E2E tests. Applies across frameworks (Playwright, Cypress, Selenium, WebdriverIO, and others). Domain-agnostic — UI behavior, flows, and business rules come from the requirements spec, observed application behavior, or explicit user instruction, never assumed.

**Position in the QA lifecycle:** governs Stage 7 (Automation, Gradual), the UI/E2E track alongside `api_automation_testing_rule` (API track). Picks up test cases already designed under `manual_testcases_rule`/`generate_testcases` and automates the stable, high-frequency, deterministic ones — doesn't redefine test design (owned by `generate_testcases`) or manual formatting (owned by `manual_testcases_rule`).

---

## 1. Architecture — Page Object Model

- **Page Objects** — one per logical page/screen; expose action methods (`login()`, `addToCart()`) and readable state queries, not raw locators, to the Test Layer.
- **Component Objects** — for repeated UI units (e.g. a product card, a modal, a nav bar) reused across pages — avoid duplicating locator logic per page.
- **Action methods own their resulting-state verification** — a method like `submitForm()` should itself wait for and confirm the state it causes (URL change, element appearance/disappearance, network idle) before returning, rather than leaving the caller to guess whether the action actually completed.
- **Test Layer** — scenarios, setup/preconditions, assertions, cleanup — calls Page/Component Object methods, does not touch raw locators directly.
- **Base Page/Fixture** — shared setup (browser context, auth state, navigation helpers) factored out, not duplicated per test file.

---

## 2. Locator Strategy

Prefer stable, semantic locators over brittle ones, in this order of preference: role/accessible-name locators (`getByRole`, ARIA), explicit test IDs (`data-testid`), stable text content, then CSS/XPath only when nothing more stable is available. Avoid absolute/deep XPath and auto-generated class names that change with styling. When the application has no test IDs and locators must rely on fragile structure, flag it as a testability gap rather than silently building on it.

---

## 3. Wait and Retry Strategy

- Rely on the framework's built-in auto-waiting (element visible/enabled/stable) instead of hardcoded `sleep()`/fixed delays — a fixed sleep is never an acceptable fix for a race condition.
- For state that becomes consistent asynchronously (e.g. a session/cookie settling, a modal finishing its transition, an eventually-consistent API-backed value), use an explicit retry-until-true pattern (e.g. Playwright's `toPass()` / expect-retrying assertions) scoped to the specific condition — not a blanket increase of a global timeout.
- Distinguish an app that is _slow_ (increase a documented, justified timeout) from an app that has a _race condition_ (fix the synchronization point, e.g. wait for the specific network response or DOM mutation that signals true readiness) — never treat the second as the first.
- Never add a wait/retry merely because a test is intermittently failing without first classifying why (see Section 9).

---

## 4. Assertions

- Assert observable UI state relevant to the scenario: visible text, element presence/absence, URL, ARIA state, or an application-observable side effect (record created, cart total updated) — not implementation details (internal class names, DOM structure not visible to the user).
- Avoid full-page pixel/screenshot assertions for functional tests — they are brittle to unrelated styling changes; use them only for tests whose explicit purpose is visual regression, and scope them to a stable region when possible.
- For dynamic values (timestamps, generated IDs), assert format/pattern or presence rather than an exact hardcoded value, unless the value is genuinely deterministic in the test setup.
- Don't assert every visible element on a page "for coverage" — assert what the scenario is actually about.

---

## 5. Test Isolation and Session/State Management

- Each test should be independently executable — avoid relying on state left behind by a previous test unless the dependency is the scenario itself (a multi-step workflow test).
- Use isolated browser contexts per test (not a shared browser/session) so parallel execution doesn't leak state between tests.
- **Authentication reuse** (e.g. saved storage state/session) is a performance optimization, not a default — don't reuse a session when it would mask a bug the test is meant to catch (e.g. a login/permission test) or compromise isolation.
- Be explicit about session lifecycle assumptions (e.g. server-side session expiry, cookie-based state) discovered during automation — these are exactly the kind of race condition (session vs. UI state) that should be hardened once and documented, not re-solved ad hoc in every affected test.

---

## 6. Test Data Management

- Generate unique, traceable test data per run (e.g. `ui_<scenario>_<timestamp>_<random>`); avoid real personal/production data; respect field constraints.
- Prefer `Setup → Execute → Verify → Cleanup` per test; avoid cross-test data dependency unless it's the scenario itself.
- Clean up UI-created data when safe, permitted, and non-interfering (e.g. via an API/DB teardown call rather than replaying UI steps when practical, to keep teardown fast and reliable). If cleanup can't run safely, report what was created and why cleanup was skipped.

---

## 7. Authentication and Sensitive Data

Never hardcode credentials in test code, fixtures, or config — use environment variables, secret managers, or approved test-credential providers. Never expose credentials, session tokens, or cookies in logs, console output, screenshots, videos, traces, or CI artifacts. Mask sensitive fields in any captured artifact before it's stored or uploaded.

---

## 8. Flaky Test Handling

A test that fails intermittently is not automatically "flaky and safe to retry" — classify first, using the same taxonomy as `api_automation_testing_rule` Section 6 (test defect / app defect / environment / data / timing-concurrency / unknown), adapted for UI causes: a genuine race condition (Section 3), an unstable locator (Section 2), test-order/data leakage (Section 5), or a real, intermittent app defect.

- A blanket test-level retry (re-run N times until green) may mask a real race condition — use it only as a documented, last-resort mitigation for a classified, understood cause, never as the first fix attempted.
- Track intermittently-failing tests; a test that needs a retry to pass is a signal to fix the underlying synchronization, not evidence the test is now reliable.

---

## 9. Auto-Fix and Auto-Healing Safety

An agent may propose/apply a test-code fix only when evidence clearly shows a test-side defect and the change is in scope — never modify a test merely to make it pass.

**Never do automatically:** switch to a more permissive/fragile locator just to make the element "found", add or increase a `sleep()`/fixed wait to mask a race condition, remove or weaken an assertion, delete a failing test, widen a screenshot-diff threshold without evidence, or suppress a failure without classifying it (Section 8) first.

**Potentially safe (with evidence):** an obviously broken/renamed locator matching a genuine UI change, a corrected test-data setup, a fix to a shared fixture/utility bug.

After an authorized fix: `Inspect → Modify → Run targeted test (multiple times if timing-related) → Analyze → Run relevant regression → Report`. A single green run doesn't prove a race condition is resolved — re-run a timing-sensitive fix several times before considering it validated. Report: change made, reason, evidence, tests run, validation result, remaining uncertainty.

---

## 10. Artifacts: Screenshots, Video, Trace

Capture screenshots/video/trace on failure by default (not on every run) to keep CI storage and runtime reasonable, unless a specific debugging session calls for always-on capture. Never let a captured artifact contain unmasked credentials, tokens, or sensitive personal data — mask before storing/uploading (Section 7). Retain failure artifacts long enough to support the Section 8/9 investigation and classification, per the project's CI storage policy.

---

## 11. CI Suite Placement

Split automated UI tests by role, not by arbitrary count: a **smoke suite** (fast, small, covers critical paths, runs on PR — e.g. GitHub Actions) and a **full regression suite** (broader coverage, tolerant of longer runtime, runs on a schedule — e.g. nightly via Jenkins). A test's suite placement should follow its Risk Level (Section 12) and execution cost, not just "where it happens to be right now" — a High-risk, fast, stable test belongs in smoke; a low-risk or slow test belongs in regression only.

---

## 12. Traceability to Manual Test Cases (Automation Bridge)

Mirrors `api_automation_testing_rule` Section 11 for the UI track:

- Carry the source **TC ID** and **Requirement ID** as a comment/tag/metadata annotation on the automated test, so coverage stays traceable across manual and automated suites.
- Use the same **High/Medium/Low** Risk Level as `manual_testcases_rule` Section 11 — don't introduce a separate scale.
- Good candidates are stable (low UI churn), high-frequency in regression, and deterministic — flag unstable/non-deterministic candidates rather than automating them silently.
- After automating and verifying, mark the manual TC ID "Automated" rather than deleting the manual case — removal is the user's call.

---

## 13. Coverage

Consider (not all mandatory per flow): happy path, negative input, boundary values, validation messages, state transitions, error/empty states, authorization-visible differences (if a UI element's visibility itself encodes permission), and cross-browser/viewport variance only when the project's supported-browser matrix requires it. Select coverage by the same High/Medium/Low risk scale as Section 12. Goal: meaningful risk coverage of user-facing flows, not maximum test count or every visual permutation.

---

## 14. Definition of Done

Page Object/Component architecture followed; locators stable and semantic; waits are condition-based, not fixed sleeps; assertions target observable, relevant UI state; test data deterministic/traceable with safe cleanup; credentials protected; flaky failures classified before any fix; no assertion/locator weakened just to pass; artifacts captured on failure without exposing sensitive data; TC ID/Requirement ID preserved when automating a manual case; CI suite placement matches risk; remaining limitations reported.

---

## 15. Strict Rules

1. Never hardcode or expose credentials, tokens, or session data anywhere (code, logs, screenshots, video, traces, CI artifacts).
2. Never use a fixed `sleep()`/delay to paper over a race condition — fix the synchronization point instead.
3. Never switch to a more fragile/permissive locator just to make an element "found".
4. Never modify, weaken, or delete a test merely to make it pass.
5. Never treat a passing retry as proof a race condition is resolved.
6. Classify a flaky/failing test (Section 8) before changing any automation code.
7. Preserve test isolation and — for cases sourced from a manual test — traceability to their TC ID/Requirement ID and shared Risk Level scale.
8. Report what was actually validated; never claim a test passed without an evidenced run.
9. Optimize for meaningful risk coverage of real user flows, not maximum test-case or assertion count.
