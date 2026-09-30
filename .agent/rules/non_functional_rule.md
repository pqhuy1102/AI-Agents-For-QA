---
trigger: always_on
---

# Non-Functional Testing Rules

Standards for performance, accessibility, and (non-functional) security testing. Domain-agnostic and tool-agnostic. These are never run by default — only when a documented requirement, an explicit ask, or a risk flagged in the test plan calls for them (mirrors `api_automation_testing_rule` Section 2's "validate only when a documented SLA exists" principle, generalized to this whole rule).

**Position in the QA lifecycle:** an optional layer across Test Design (3.3) and Execution (3.5/3.7). Functional API security (BOLA, mass assignment, auth edge cases) stays owned by `api_automation_testing_rule` Section 7 — this rule covers performance, accessibility, and broader/infrastructure-adjacent security that neither existing rule covers.

---

## 1. When to Apply

Apply a category only when one of these holds, and record which one in the test plan or report:

- A documented requirement (SLA/SLO, WCAG conformance target, a security policy/compliance need) exists.
- The person explicitly asks for this category.
- The test plan (`test_planning`) flags the area as High risk.

Otherwise, treat the category as out of scope and say so — never invent a threshold, compliance level, or check to fill a category that wasn't asked for.

---

## 2. Performance Testing

**Types:** load (expected traffic), stress (beyond expected, find the breaking point), spike (sudden surge), soak/endurance (sustained duration, catches leaks), scalability (does capacity scale with added resources). Pick the type(s) the objective calls for — don't run all of them by default.

**Metrics and thresholds:** response time (report percentiles — p50/p95/p99, not just average), throughput, error rate, and resource utilization (CPU/memory/connections) where observable. Validate against a documented SLA/SLO only; never invent a threshold like "under 2s". If none exists, report the measured numbers and label them `Baseline — no threshold defined`, not pass/fail.

**Design:** establish a baseline on a known-good build first; use a realistic load profile (traffic shape, data variety) rather than a single flat number; ramp up rather than instantly applying peak load unless the objective is specifically a spike test.

**Safety:** never run uncontrolled high-volume traffic against a shared or production environment without explicit authorization (Global Rule Section 4.1, echoing `api_automation_testing_rule` Section 7's stress-testing caution). Confirm the target environment and its capacity before starting; a shared staging environment can be degraded for other teams.

**Reporting:** the measured metric, the threshold (or `Baseline`), pass/fail against that threshold, environment, load profile, and any anomaly (error spikes, resource exhaustion) observed during the run — not just the end-state number.

---

## 3. Accessibility Testing

**Standard:** confirm the target conformance level (commonly WCAG 2.1/2.2 AA) with the person before testing — never assume a level. If none is stated, default to reporting findings against WCAG 2.1 AA as a reference baseline and say explicitly that this wasn't confirmed as the project's actual target.

**Coverage areas:** keyboard navigation and focus order/visibility, screen reader compatibility and ARIA correctness, color contrast, text alternatives (alt text, labels), form field labeling and error association, and heading/landmark structure.

**Method:** automated scanning (e.g. axe, Lighthouse) catches roughly a third to half of real issues — never report an automated scan's "0 issues" as "accessible". Pair it with a manual pass (keyboard-only navigation, and a screen reader check) on the critical flows the test plan or requirement scope identifies, not every screen by default.

**Reporting:** each finding with the WCAG success criterion it violates, severity (Blocking / Should Fix / Suggestion — same three-level scale as `review_testcases` Section 5), the affected element/flow, and a remediation suggestion. State the conformance level actually assessed and which criteria (not just which pages) were covered.

---

## 4. Security Testing (Non-Functional / Infrastructure-Adjacent)

Functional/API-level security (object/function authorization, mass assignment, injection, auth edge cases) is `api_automation_testing_rule` Section 7 — don't duplicate it here. This section covers what that one doesn't:

- **Dependency/vulnerability scanning** — known-CVE scan of dependencies and container/base images, using an established scanner; report findings by their own CVSS/vendor severity, don't invent a severity scale.
- **Transport and headers** — TLS configuration, HSTS, and security headers (CSP, X-Frame-Options, etc.) where applicable to the application type.
- **Session/cookie security** — Secure/HttpOnly/SameSite attributes, session timeout and invalidation behavior.
- **Client-side exposure (UI-facing)** — reflected/stored XSS surface and CSRF protection on state-changing UI actions, exercised through the UI/E2E track when in scope; coordinate with `ui_automation_testing_rule` rather than duplicating its test isolation/auth rules.

**Authorization and safety:** use authorized environments and controlled data only. Never run an intrusive, destructive, or exploit-style test (actual data exfiltration, DoS-style traffic, live exploitation) without explicit, specific authorization — the same bar as `api_automation_testing_rule` Section 7. A scan that only detects and reports (not exploits) a weakness is the default; escalate to anything more invasive only when explicitly asked and authorized.

**Reporting:** finding, evidence, the standard/scanner it came from, its own severity rating, affected component, and remediation reference — never assert a security conclusion (e.g. "no vulnerabilities") beyond what the specific tests actually covered.

---

## 5. Risk-Based Selection and Traceability

Map each applied category to the shared **High/Medium/Low** risk scale (`manual_testcases_rule` Section 11) when reporting business impact, while keeping the category's own native severity scale (CVSS, WCAG level, SLA breach magnitude) as the primary technical classification — don't collapse one into the other; report both when they differ.

When a non-functional test traces back to a specific Requirement ID or TC ID (e.g. an NFR stated in the requirements spec, or a flagged item in the test plan), preserve that link in the finding, the same way `api_automation_testing_rule`/`ui_automation_testing_rule` Section 11/12 preserve TC ID/Requirement ID for automated functional tests.

---

## 6. Classification Before Acting on a Finding

Before recommending a fix or re-testing, classify a non-functional finding using the same shared taxonomy as `api_automation_testing_rule` Section 6 / `ui_automation_testing_rule` Section 8 (test-side issue, implementation defect, environment/configuration issue, contract/requirement mismatch, unknown) adapted here — e.g. a performance regression could be a real implementation issue, a test-environment capacity problem, or a flawed load profile. Never propose loosening a threshold or suppressing a scan finding merely to get a passing result — that mirrors the Auto-Fix/Auto-Healing Safety prohibition in both automation rules.

---

## 7. Definition of Done

Category applied only where Section 1's trigger was met and recorded; thresholds/conformance level came from a document or explicit confirmation, never invented; findings include evidence, native severity, and (where applicable) a mapped Risk Level; destructive/intrusive tests ran only with explicit authorization; a finding was classified before any fix was proposed; the report states what was and wasn't covered rather than implying full coverage; Requirement ID/TC ID traceability preserved when it exists.

---

## 8. Strict Rules

1. Never apply a non-functional category without a documented requirement, explicit ask, or plan-flagged risk — and record which one applied.
2. Never invent a performance threshold, a conformance level, or a security severity scale — use the documented one, the tool/standard's native scale, or label the result `Baseline` / unconfirmed.
3. Never run uncontrolled high-volume, destructive, or exploit-style tests against shared/production environments without explicit authorization.
4. Never report an automated scan's clean result as full compliance/security — state what method and scope were actually covered.
5. Never duplicate `api_automation_testing_rule` Section 7's functional API security checks here.
6. Classify a finding's cause before proposing a fix; never loosen a threshold or suppress a finding just to pass.
7. Preserve Requirement ID/TC ID traceability when a finding maps to one.
8. Report both the category's native severity and, where relevant, the shared High/Medium/Low risk mapping — don't merge them into one number.
