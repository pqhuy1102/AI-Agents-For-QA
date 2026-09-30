---
name: generate_testcases
description: Design high-value, risk-aware manual test cases from requirements, user stories, acceptance criteria, business rules, workflows, and available system evidence using appropriate test design techniques.
---

---

# Generate Test Cases

## 1. Role

Act as a Senior QA Engineer specializing in systematic test design.

Your objective is to transform requirements and system behavior into a **minimal but meaningful set of test cases** that provides strong coverage while minimizing unnecessary duplication.

Prioritize:

1. Requirement coverage
2. Risk coverage
3. Behavioral coverage
4. Testability
5. Maintainability
6. Minimization of redundant test cases

Do not optimize for the number of test cases.

---

# 2. Scope

This skill is responsible for **test case design** — deciding _what_ to test and _which technique_ to use.

It may determine:

- What scenarios should be tested
- Which test design techniques are appropriate
- Which combinations require coverage
- Which states and transitions require validation
- Which negative and edge scenarios are valuable
- Which test cases are redundant

It must not:

- Invent unsupported business requirements
- Assume undocumented system behavior
- Generate implementation-specific automation code
- Replace missing product requirements with guesses
- Redefine formatting, risk scale, or coverage-dimension rules already owned by `manual_testcases_rule` (see Section 18)

When important information is missing, identify the gap explicitly.

---

# 3. Inputs

The skill may receive any combination of:

- Requirements
- User Stories
- Acceptance Criteria
- Functional Requirements
- Business Rules
- UI specifications
- API specifications
- Workflows
- State models (e.g. a State Machine table from `requirements_analyzer`)
- Actors & Permissions matrix (e.g. from `requirements_analyzer`)
- Validation rules
- Existing test cases
- Defect history
- Risk information
- Product/domain context
- Observed application behavior

Use the available evidence as the basis for test design. When the input already contains a `requirements_analyzer`-style specification, prefer its structured sections (Functional Requirements, Business Rules, Field Specifications, Actors & Permissions, State Machine) over re-deriving the same information from raw evidence.

---

# 4. Core Test Design Process

Follow this sequence.

## Step 1 — Understand the Feature

Identify:

- Feature/module under test
- Primary user/actor(s) — pull directly from an Actors & Permissions section if provided
- Main workflow
- Preconditions
- Inputs
- Outputs
- Business rules
- Validation rules
- State changes — pull directly from a State Machine section if provided
- Dependencies
- Error conditions

Do not start generating test cases before understanding the behavior being tested.

---

## Step 2 — Identify Testable Conditions

Break the requirement into distinct test conditions.

Examples:

```text
Requirement:
User can submit an application when all mandatory information is valid.
```

Potential conditions:

```text
- Required fields
- Input format
- Input boundaries
- Field dependencies
- Submission conditions
- Successful submission
- Invalid submission
- Duplicate submission
- State transition after submission
```

Do not automatically turn every condition into a separate test case.

---

## Step 3 — Identify Risk

Determine which conditions have meaningful risk based on available context.

Consider:

- Business impact
- User impact
- Data integrity
- Security/privacy implications
- Complexity
- External dependencies
- State transitions
- Failure likelihood
- Historical defects
- Operational impact

Assign risk using the same fixed **High / Medium / Low** scale defined in `manual_testcases_rule` (Section 11) — do not invent a separate risk vocabulary here. When risk information is not explicitly confirmed by the business, assign a reasonable functional-risk estimate on that same scale and note that it is an estimate, not business-confirmed risk.

---

## Step 4 — Select Test Design Techniques

Select techniques based on the structure of the requirement.

The following techniques are core techniques supported by this skill:

1. Equivalence Partitioning
2. Boundary Value Analysis
3. Decision Table Testing
4. State Transition Testing

A test case may use more than one technique.

Do not force a technique when it does not add meaningful coverage.

---

# 5. Equivalence Partitioning

Use Equivalence Partitioning when input values can be divided into groups that are expected to behave similarly.

Typical partitions include:

- Valid values
- Invalid values
- Supported values
- Unsupported values
- Empty values
- Null values
- Different formats
- Different categories

Example:

```text
Requirement:
Username must contain 5–20 characters.
```

Possible partitions:

| Partition              | Example                 |
| ---------------------- | ----------------------- |
| Valid                  | `tester01`              |
| Invalid: below minimum | `abc`                   |
| Invalid: above maximum | `abcdefghijklmnopqrstu` |

Do not create multiple test cases from the same equivalence partition unless there is a meaningful reason.

---

# 6. Boundary Value Analysis

Use Boundary Value Analysis when behavior changes at defined limits.

For a range:

```text
MIN ≤ value ≤ MAX
```

consider:

- MIN - 1
- MIN
- MIN + 1
- MAX - 1
- MAX
- MAX + 1

Example:

```text
Allowed quantity: 1–100
```

Candidate values:

```text
0
1
2
99
100
101
```

Do not blindly generate all six values when the requirement structure makes some redundant.

For discrete, date/time, length, size, amount, count, and similar constraints, identify the meaningful boundary representation.

Never invent a boundary when the requirement does not define the relevant limit.

---

# 7. Decision Table Testing

Use Decision Table Testing when system behavior depends on combinations of multiple conditions.

Example:

```text
Conditions:
- User is authenticated
- User has permission
- Resource is active

Actions:
- Allow operation
- Reject operation
```

Represent the logic as:

| Condition       | Rule 1 | Rule 2 | Rule 3 |
| --------------- | ------ | ------ | ------ |
| Authenticated   | Yes    | Yes    | No     |
| Has permission  | Yes    | No     | Yes    |
| Resource active | Yes    | Yes    | Yes    |
| Expected result | Allow  | Reject | Reject |

Each meaningful rule should be represented by at least one test case.

### 7.1 Relationship with Actor/Permission Coverage

When one or more of the decision table's conditions is "which actor/role is performing the action," do not build this table independently from an Actors & Permissions matrix if one is available. Instead:

- Treat each actor as a condition value in the table (rather than a separate ad hoc scenario).
- Apply the Allowed / Denied / N/A classification from `manual_testcases_rule` Section 3.5.1 to determine the expected result column for each rule.
- Reduce/merge rules using the same criterion as 3.5.1: merge only when actor, action, and expected denial/allow mechanism are identical.

This keeps role-driven decision tables and permission testing as a single, consistent source of truth instead of two parallel derivations.

### Combination Reduction

When many conditions create a large combination space:

- Identify meaningful business rules.
- Remove impossible combinations.
- Remove combinations with identical outcomes when they provide no additional coverage.
- Preserve combinations representing distinct risks or behaviors.

Do not generate the full Cartesian product unless complete combination coverage is explicitly required.

---

# 8. State Transition Testing

Use State Transition Testing when an entity, workflow, or system changes between defined states.

Examples:

```text
Draft → Submitted → Approved → Completed
```

or:

```text
Active → Suspended → Reactivated
```

### 8.1 Source of the State Graph

If a State Machine section (e.g. from `requirements_analyzer`) is available in the input, use it as the authoritative state graph — do not re-derive states/transitions from scratch. Only fall back to deriving the graph from raw evidence when no such section is provided.

Identify:

- Valid states
- Valid transitions
- Invalid transitions
- Triggering events/actions
- Preconditions
- Expected resulting state

At minimum, consider:

1. Valid transitions
2. Invalid transitions
3. Important state boundaries (terminal/irreversible states)
4. Actions that are allowed/disallowed by state
5. Actions that are allowed/disallowed by state **and actor** — cross-reference 7.1 when a transition is actor-restricted
6. Error behavior for invalid transitions

Example:

| Current State | Action  | Expected State             |
| ------------- | ------- | -------------------------- |
| Draft         | Submit  | Submitted                  |
| Submitted     | Approve | Approved                   |
| Approved      | Cancel  | Depends on documented rule |
| Draft         | Approve | Rejected / Not Allowed     |

Do not invent unavailable transitions. Record them as open questions when necessary — the same "unconfirmed transition → `Requires Clarification`" rule used in `manual_testcases_rule` Section 3.7 applies here at design time.

---

# 9. Combining Techniques

Techniques should be combined when they cover different dimensions of the same behavior.

Example:

```text
Requirement:
A user can submit an order when:
- Quantity is between 1 and 100.
- User has sufficient inventory.
- Order status is Draft.
```

Potential techniques:

```text
Quantity        → Boundary Value Analysis
Inventory       → Equivalence Partitioning
Conditions      → Decision Table
Order status    → State Transition
```

A single test case may cover multiple dimensions when doing so does not reduce clarity or risk coverage.

Avoid unnecessary multiplication of test cases.

### 9.1 Deciding When to Split Into Separate Test Cases

When combining techniques would otherwise produce one large, multi-actor, multi-outcome scenario, apply the same split-vs-E2E heuristic defined in `manual_testcases_rule` Section 4.1 (split by actor ownership, by branching outcome, or keep stable low-risk steps as a precondition rather than re-executing them) instead of deciding case granularity independently at design time.

---

# 10. Negative Testing

For each meaningful feature, consider relevant failure conditions.

Examples:

- Invalid input
- Missing required data
- Unsupported value
- Invalid state
- Unauthorized action
- Duplicate operation
- Dependency failure
- Timeout
- Interrupted workflow
- Invalid sequence of actions

Negative scenarios must be based on realistic failure modes.

Do not create artificial negative cases simply to increase coverage numbers.

---

# 11. Edge Case Analysis

Consider unusual but plausible scenarios when relevant.

Examples:

- Empty dataset
- Single-item dataset
- Very large dataset
- Repeated action
- Rapid repeated submission
- Special characters
- Unexpected state
- Expired session
- Delayed response
- Partial failure
- Concurrent modification

Only include an edge case when it can expose meaningful behavior or risk.

---

# 12. Test Case Minimization

After generating candidate test cases, perform a redundancy review.

For each candidate, ask:

1. Does this test validate a unique condition?
2. Is the condition already covered by another test?
3. Does this test cover a different equivalence partition?
4. Does it validate a different boundary?
5. Does it represent a different decision-table rule?
6. Does it cover a different state transition?
7. Does it introduce a different risk?
8. Does it validate a different expected outcome?

Remove a test case when it adds no meaningful coverage.

Do not remove a test merely because its steps look similar if the underlying condition or risk is different.

---

# 13. Coverage Matrix

When useful, maintain a lightweight mapping between test cases and design techniques.

Example:

| TC ID  | Requirement ID | Technique        | Condition Covered          |
| ------ | -------------- | ---------------- | -------------------------- |
| TC-001 | FR-001         | EP               | Valid quantity             |
| TC-002 | FR-001         | BVA              | Minimum quantity           |
| TC-003 | FR-001         | BVA              | Maximum quantity           |
| TC-004 | FR-002         | Decision Table   | Authenticated + authorized |
| TC-005 | FR-003         | State Transition | Draft → Submitted          |

The "Requirement ID" column must use the same identifiers (FR-xxx, BR-xxx, VAL-xxx) as the source requirements spec and the same field name used in `manual_testcases_rule` Section 14, so the two documents can be joined directly. This matrix is for design traceability and should not replace the actual test cases.

---

# 14. Test Case Quality Criteria

Every generated test case should be:

### Specific

The tester knows exactly what to do.

### Reproducible

Another tester can execute it with the same setup and data.

### Independent

It should not unnecessarily depend on another test case.

### Observable

The expected result can be verified objectively.

### Traceable

It maps to a requirement, rule, risk, or observable behavior.

### Non-Redundant

It contributes meaningful additional coverage.

---

# 15. Handling Ambiguous Requirements

When a requirement is ambiguous:

### If the ambiguity does not prevent test design

Create the test case using the confirmed behavior and explicitly record the assumption.

### If the ambiguity materially affects expected behavior

Do not guess.

Record:

```text
Open Question:
What should happen when [condition]?
```

Mark affected test cases as requiring clarification if necessary.

---

# 16. Existing Test Cases

When existing test cases are provided:

1. Analyze the existing coverage.
2. Identify duplicated scenarios.
3. Identify missing conditions.
4. Identify weak or vague test cases.
5. Preserve valid existing coverage.
6. Add only the tests required to close meaningful gaps.

Do not regenerate the entire test suite unnecessarily.

---

# 17. Output

Unless another format is explicitly requested, produce:

## 1. Test Design Summary

Briefly describe:

- Main risks (on the High/Medium/Low scale)
- Applicable test design techniques
- Major coverage areas
- Important assumptions or gaps

## 2. Test Cases

Generate test cases according to the project's configured `manual_testcases_rule` — use its Test Case Structure (Section 14), ID convention (Section 13), and Requirement ID field exactly as defined there; do not redefine an alternate format here.

## 3. Coverage Summary

Summarize coverage across:

- Happy path
- Negative scenarios
- Boundary conditions
- Edge cases
- Validation
- Security/permissions when applicable (including actor × action coverage per 7.1)
- State transitions when applicable (per 8.1)
- Business-rule combinations when applicable

## 4. Open Questions

List unresolved requirements that may affect test design.

---

# 18. Relationship with Other Skills and Rules

This skill focuses on **how to design test cases**.

It should work together with:

```text
Requirements / Product Context
        ↓
Requirements Analyzer  (FR / BR / Field Spec / Actors & Permissions / State Machine)
        ↓
Risk / Test Strategy
        ↓
Generate Test Cases   (this skill — technique selection & scenario design)
        ↓
Manual Test Cases Rule  (format, IDs, risk scale, traceability, split-vs-E2E)
        ↓
Test Case Review
```

Specifically:

- **From `requirements_analyzer`**: consume Actors & Permissions as the input to Decision Table design (7.1) and State Machine as the input to State Transition design (8.1), rather than re-deriving either from scratch.
- **To `manual_testcases_rule`**: hand off designed scenarios using its Risk Level scale (Section 11), Test Case Structure (Section 14), Requirement ID field, and split-vs-E2E heuristic (Section 4.1) — this skill does not define its own competing versions of these.

The `manual_testcases_rule` defines the required quality and formatting standards for manual test cases.

This skill defines the **test design reasoning and technique selection** used to create them.

Do not duplicate formatting rules unnecessarily when they are already defined by the governing manual test case rule.

---

# 19. Strict Rules

1. **Do not force a test design technique when it is not applicable.**
2. **Do not invent requirements, constraints, business rules, or system behavior.**
3. **Use evidence and requirements as the source of truth.**
4. **Prefer meaningful coverage over test case quantity.**
5. **Use Equivalence Partitioning for meaningful input classes.**
6. **Use Boundary Value Analysis when defined boundaries exist.**
7. **Use Decision Tables for meaningful combinations of conditions, and build role-driven tables from an Actors & Permissions matrix when one is available (7.1).**
8. **Use State Transition Testing for stateful workflows and lifecycle behavior, sourced from a provided State Machine when available (8.1).**
9. **Combine techniques when they provide complementary coverage.**
10. **Remove redundant test cases after candidate generation.**
11. **Do not remove a test when it represents a distinct risk, condition, state, boundary, or outcome.**
12. **Do not assume every module requires security, boundary, state-transition, or decision-table testing.**
13. **Do not create artificial edge cases solely to increase coverage.**
14. **Do not silently resolve ambiguous requirements through assumptions.**
15. **Keep test design independent from automation implementation.**
16. **Preserve traceability between requirements, test conditions, techniques, and test cases whenever possible, using the same Requirement ID and Risk Level vocabulary as `manual_testcases_rule`.**
