---
trigger: always_on
---

# API Automation Testing Rules

Quality, architecture, safety, and execution standards for automated API tests. Applies across frameworks/languages (REST Assured/Java, Playwright API/TS, Requests/Python, Supertest/JS-TS, others). Domain-agnostic — API behavior, status codes, security requirements, schemas, and business rules come from the API contract, requirements, documentation, or observed behavior, never assumed.

**Position in the QA lifecycle:** governs Stage 7 (Automation, Gradual). Picks up cases already designed under `manual_testcases_rule`/`generate_testcases` and automates the stable, high-frequency, deterministic ones — doesn't redefine test design (owned by `generate_testcases`) or manual formatting (owned by `manual_testcases_rule`).

---

## 1. Architecture

Separate responsibilities: **API Client/Service Layer** builds/sends requests (URL, params, headers, auth, payload) — MUST NOT contain assertions. **Model/DTO Layer** represents request/response data contracts (POJO, dataclass/Pydantic, TS interface, JSON schema), not assertions. **Test Layer** owns scenarios, setup, service calls, assertions, cleanup, reporting. **Test Data/Utility Layer** holds reusable generic helpers (data gen, config, auth, schema validation, date/time) — no feature-specific assertions here. **Request Builders** (Builder/factory/fixture) only when payload complexity warrants it, not for architectural appearance.

---

## 2. Assertion Standards

Assertions come from the API contract and the test's purpose — never assert every response property by default. **Status Code**: validate when the contract defines it; never assume from the method alone. **Schema**: validate when a formal schema exists or structure matters; don't duplicate checks already owned by a contract-test layer. **Business-Critical Fields**: exact match for deterministic values, pattern/range for dynamic ones, relationship/invariant checks across fields — don't assert every field without reason. **Negative Response**: validate the error contract (code, type, message) when defined; don't assert exact messages when the contract only guarantees a category. **Response Time**: only assert with a documented SLA/SLO or a performance-focused test — no arbitrary thresholds. **Headers**: validate when required by contract, negotiation, caching, auth, security, or tracing — not for coverage's sake.

---

## 3. Authentication and Sensitive Data

**Credentials**: never hardcode passwords/keys/tokens; use env vars, secret managers, or approved test-credential providers; never expose in code, test data, logs, screenshots, or reports. **Token Lifecycle**: obtain/reuse/refresh per the actual auth contract; explicitly test expiration, revocation, insufficient permissions. Token reuse is a performance optimization, not a default — skip it when it compromises isolation or test intent. **Masking**: mask passwords, tokens, keys, card numbers, cookies, auth headers from logs, reports, dumps, screenshots, CI artifacts, and debug files — even inside an error response.

---

## 4. Test Data Management

**Dynamic Data**: generate unique values (e.g. `api_<testName>_<timestamp>_<random>`); avoid collisions and real personal/production data; respect field constraints; stay traceable to the run. **Independence**: prefer `Setup → Execute → Verify → Cleanup` per test over cross-test data dependency, unless the dependency is the scenario itself (workflow/state-transition tests). **Cleanup**: clean up created data when safe, permitted, non-interfering; never blindly run destructive cleanup; if it can't run safely, report the resource ID and why it was skipped. **Data-Driven Testing**: parameterize to cover multiple data sets without duplicating test logic.

---

## 5. HTTP Method and Idempotency

GET/PUT/DELETE are generally idempotent, POST generally isn't, PATCH depends on the operation — treat as protocol expectations, not hardcoded assertions (`POST → 201`, `DELETE → 204` are common, not guaranteed; the contract decides). When idempotency matters, explicitly test repeated requests (same request twice; for idempotency-key APIs also same-key-repeat, different-key, retry-after-timeout) and compare response, resource state, side effects, and duplicate creation — expected behavior always comes from the contract.

---

## 6. Contract, Schema, and Change Classification

Use OpenAPI/JSON Schema/GraphQL schema/API docs/consumer-provider contracts as source of truth where they exist (required fields, types, enums, formats, nullable fields, nested/response/error structure). Never modify a test merely to accommodate an unexpected API change — classify it first, using one shared taxonomy (also drives Section 9's auto-fix gate):

```
1. Test implementation defect
2. API implementation defect
3. API contract/documentation change (expected or unexpected)
4. Test data problem
5. Authentication/authorization problem
6. Environment/configuration problem
7. Network/infrastructure problem
8. Dependency/service problem
9. Timing/concurrency problem
10. Unknown
```

**Investigation by symptom:** status-code failures → inspect URL, method, params, sanitized body, headers, auth state, response, test data, contract. Schema failures → identify field/path, compare actual vs. expected, determine contract-vs-implementation-vs-test fault. `401/403` → auth, token validity, permissions, environment. `404` → base URL, path, path params, resource existence, environment. `5xx` → preserve sanitized evidence, check contract compliance, report the server-side failure.

---

## 7. Security Testing

Apply per the API's security model, threat model, scope, and authorization — never run automatically against every endpoint. When applicable, cover: object-level authorization (BOLA — user can't access another's object), function-level authorization (privileged ops need the right permission), mass assignment (protected fields like `role`/`isAdmin`/`ownerId`/`createdAt` can't be client-set), authentication edge cases (missing/invalid/expired/revoked/malformed credentials), content-type/negotiation handling, malformed input (rejected safely, documented error, no partial state change or detail leak), resource limits (only documented ones — never invent a limit), concurrency (race-sensitive ops preserve invariants), sensitive data exposure (check against the actual sensitive-data model, not by field name alone), and input/injection handling.

Security tests use authorized environments and controlled data only — no uncontrolled high-volume, destructive, or DoS-like testing without explicit authorization. Security assertions go beyond the status code: e.g. a BOLA test verifying `403` should also confirm unauthorized data wasn't exposed, no unauthorized state changed, and no sensitive field leaked. A security failure may legitimately return different codes per contract — don't assume one universal code.

---

## 8. Test Isolation and Reliability

Avoid unnecessary dependence on execution order, shared mutable data, prior results, uncontrolled external services, system time, or untraceable random values; make unavoidable external dependencies explicit. Use the result vocabulary from the Global Rule (Passed / Failed / Skipped / Blocked / Not Executed / Unknown) — don't invent a separate one. Never convert an infrastructure/environment failure into a reported product failure.

---

## 9. Auto-Fix and Auto-Healing Safety

An agent may propose/apply a test-code fix only when evidence clearly shows a **test-side** defect (per Section 6's taxonomy) and the change is in scope. Never modify a test merely to make it pass.

Potentially safe (with evidence): obvious test typo/selector fix, wrong test-data construction, wrong serialization config, a locally broken utility. Never do automatically: remove/weaken a failing assertion, lower an expected status code to match actual behavior, widen assertions without evidence, repeatedly raise timeouts until green, ignore schema failures, disable security checks, delete failing tests, change production behavior, or suppress a failure without classifying it first.

After an authorized fix: `Inspect → Modify → Run targeted test → Analyze → Run relevant regression → Report`. A single pass doesn't prove the issue is resolved. Report: change made, reason, evidence, tests run, validation result, remaining uncertainty.

---

## 10. Logging and Reporting

Include test name, method, sanitized endpoint/params/payload, response status, sanitized response body, response time, correlation ID, failure reason, environment. Never log secrets/credentials. Include only identifiers needed for traceability, no extra sensitive data.

---

## 11. Traceability to Manual Test Cases (Automation Bridge)

When an automated test originates from an existing manual test case (per `manual_testcases_rule`), preserve the link:

- Carry the source **TC ID** and **Requirement ID** as a comment/tag/metadata annotation on the automated test (e.g. `@tc("CRM_LOGIN_TC_001") @req("FR-LOGIN-001")`), so coverage stays traceable across manual and automated suites.
- Don't silently narrow or widen the scenario's intent from the manual case — if automated coverage differs from the manual case, state that explicitly.
- Use the same **High/Medium/Low** Risk Level as `manual_testcases_rule` Section 11 — don't introduce a separate risk vocabulary. Automating a High-risk case doesn't by itself lower its risk.
- Good automation candidates are stable, high-frequency in regression, and deterministic — not one-off/exploratory. Flag unstable/non-deterministic candidates rather than automating them silently.
- After automating and verifying, mark the manual TC ID as "Automated" rather than deleting the manual case — removing manual coverage is the user's call, not an automatic cleanup step.

---

## 12. Coverage

Consider (not all mandatory per endpoint): happy path, negative cases, boundary values, invalid input, authorization, authentication, state transitions, error handling, contract/schema, idempotency, concurrency, resource limits, security, integration/dependency behavior. Select by business risk, security risk, data impact, statefulness, failure impact, known defects, integration complexity, and test objective — using the same High/Medium/Low risk scale as Section 11. Goal: meaningful risk coverage, not max test count.

---

## 13. Definition of Done

Architecture layers separated; test data deterministic/traceable; credentials protected; assertions match the contract; relevant business + negative/error behavior covered; security considered where applicable; idempotency tested where relevant; cleanup performed when safe; logs clean of sensitive data; tests isolated where practical; validation actually executed; failures classified before modification; no assertion weakened just to pass; TC ID/Requirement ID preserved when automating a manual case; limitations reported.

---

## 14. Strict Rules

1. Never hardcode or expose secrets/credentials anywhere.
2. Never assume status codes, schema, business rules, limits, or SLA thresholds without contract evidence.
3. Never assume every endpoint needs the same assertion set or security/performance/concurrency scenarios.
4. Never modify, weaken, or delete a test merely to make it pass.
5. Never run destructive cleanup or uncontrolled high-volume/stress traffic against shared/production environments without confirmed safety and authorization.
6. Classify failures (Section 6) before changing any automation code.
7. Preserve test isolation, reproducibility, and — for cases sourced from a manual test — traceability to their TC ID/Requirement ID and shared Risk Level scale.
8. Report what was actually validated; never claim a test passed without an evidenced run.
9. Optimize for meaningful risk coverage, not maximum test-case count.
