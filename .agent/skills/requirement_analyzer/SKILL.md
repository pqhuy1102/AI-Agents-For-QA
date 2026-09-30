---
name: requirements_analyzer
description: Analyze a web page, UI, module, or available application evidence and produce an evidence-based requirements specification, including functional requirements, user stories, acceptance criteria, field specifications, business rules, validations, workflows, actors/permissions, state machines, and identified gaps. Designed to scale from simple features to complex, multi-actor, multi-state domains.
---

---

# Requirements Analyzer

## 1. Purpose

The `requirements_analyzer` skill transforms available application evidence—such as a live web page, UI, DOM/HTML structure, screenshots, API behavior, or existing documentation—into a structured and testable requirements specification.

The primary goal is to help QA Engineers, Testers, Developers, and other engineering agents understand **what the system currently does**, **what can be reasonably derived from the available evidence**, and **what remains unknown or requires clarification**.

The skill must prioritize **accuracy, traceability, and explicit uncertainty** over completeness based on assumptions, and must scale its depth to the complexity of the feature or domain being analyzed — from a single-field form to a multi-actor, multi-state enterprise workflow.

---

## 2. Core Principles

### 2.1 Evidence First

Only state a behavior as an existing system requirement when it is supported by available evidence.

Valid evidence may include:

- Visible UI behavior
- DOM/HTML attributes
- Browser interaction results
- Screenshots
- Existing requirements or product documentation
- API requests/responses
- Validation messages
- Application state changes
- Existing test cases
- Explicit instructions from the user

Do not invent business rules that cannot be supported by evidence.

---

### 2.2 Separate Facts from Inferences

Clearly distinguish between:

- **Observed Behavior** — directly verified from the application or provided evidence.
- **Derived Requirement** — logically and narrowly inferred from one or more observed behaviors, with the inference chain stated explicitly (e.g., "Field X is disabled AND tooltip says Y → derived rule: Z"). If the inference requires more than one reasonable logical step, or if a plausible alternative explanation exists, classify it as an **Assumption** instead.
- **Assumption** — a hypothesis that has not been verified, or a derivation with more than one plausible interpretation.
- **Open Question** — information required from the Product Owner, Business Analyst, Developer, or other source.

Never present an assumption as a confirmed requirement.

---

### 2.3 Testability

Requirements should be written so that a QA Engineer can determine whether the requirement is satisfied.

Prefer requirements that define:

- Preconditions (including actor/role, when relevant)
- User action
- Expected system behavior
- Resulting state
- Validation rules
- Error behavior
- Relevant dependencies

Avoid vague statements such as:

> The system should work correctly.

Prefer:

> When the user submits the form with a required field empty, the system displays the corresponding validation message and prevents form submission.

---

### 2.4 Do Not Over-Infer

Do not infer complex business logic solely from UI structure.

For example, the following UI evidence:

```text
Amount: [input]
Submit: [button]
```

does not justify assuming:

- Maximum transaction amount
- Daily transaction limits
- User eligibility rules
- Approval workflows
- Backend validation rules
- Fraud detection rules

Unless these behaviors are explicitly observable or documented, record them as **Open Questions**.

---

### 2.5 Scale Depth to Complexity

Not every feature needs every section of this skill. Before generating the document, estimate complexity signals:

- **Single actor, single state, few fields** → simple feature. Skip Actors & Permissions, State Machine, and most Dependencies sub-sections; keep the document short.
- **Multiple actors/roles with different visible behavior, or an entity that moves through multiple named states, or a process spanning multiple pages/approvals** → complex feature. Include Actors & Permissions and/or State Machine sections, and expand Workflow to show decision points.

State which complexity signals were observed at the start of the Overview section, so the reader understands why certain sections are included or omitted.

---

## 3. Input Analysis

When analyzing a web page or module, inspect the available evidence systematically.

### 3.1 Page Structure

Identify relevant application areas such as:

- Header
- Navigation
- Sidebar
- Main content
- Forms
- Tables
- Cards
- Modals
- Footer
- Notifications
- Other interactive components

Focus on components relevant to the target feature rather than documenting every visual element.

---

### 3.2 Form and Input Analysis

For each input component, inspect available attributes and behavior, including:

- Label
- Input type
- Placeholder
- Default value
- Required state
- Disabled state
- Read-only state
- Minimum value
- Maximum value
- Minimum length
- Maximum length
- Pattern
- Accepted formats
- Input dependencies
- Validation behavior

Where available, inspect both DOM attributes and actual runtime behavior.

---

### 3.3 Interactive Elements

Identify meaningful user actions, including:

- Buttons
- Links
- Checkboxes
- Radio buttons
- Dropdowns
- Tabs
- Toggles
- Menus
- Pagination
- Search
- Filters
- Sorting controls
- Edit/Delete actions
- Submit/Save/Cancel actions

For each meaningful interaction, determine:

1. What action the user performs.
2. What condition enables or disables the action.
3. What system behavior follows.
4. Whether the page, state, URL, API response, or displayed data changes.
5. Whether success or error feedback is presented.
6. Whether the behavior differs by actor/role (see 3.6).

---

### 3.4 Validation and Error Behavior

Identify observable:

- Field-level validation
- Form-level validation
- Error messages
- Warning messages
- Toast notifications
- Inline messages
- Disabled states
- Error pages
- Empty states
- Loading states
- Retry behavior

Record the exact validation message when it is observable.

Do not invent an error message when none is available.

---

### 3.5 Workflow Analysis

Identify user workflows and dependencies between components.

Examples:

- A Submit button becomes enabled only after required fields are completed.
- Selecting a country changes the available states/provinces.
- Selecting a payment method changes the required fields.
- Saving a form redirects the user to another page.
- Deleting an item requires confirmation.

Represent workflows as ordered steps when the sequence is important. When a workflow branches based on a condition, role, or system response (approve/reject, success/failure, eligible/ineligible), represent it as a **decision point** rather than forcing it into a single linear sequence (see Section 5.6).

---

### 3.6 Actors & Permissions Analysis (when applicable)

Apply this sub-section whenever evidence shows more than one type of user/role interacting with the feature, or when UI elements are conditionally visible/enabled based on role, ownership, or permission level.

For each actor/role identified, determine:

- What that actor can view.
- What that actor can do (create/edit/delete/approve/reject/etc.).
- What is hidden, disabled, or read-only for that actor compared to others.
- Whether evidence shows explicit permission checks (e.g., "Insufficient permissions" messages, hidden buttons, 403 responses) or whether role differences are only assumed.

If only one actor is observable but the domain strongly implies others exist (e.g., an approval button with no visible approver flow), record the missing actor as an **Open Question**, not an assumption.

---

### 3.7 State Machine / Entity State Analysis (when applicable)

Apply this sub-section whenever the feature centers on an entity that moves through multiple named states (e.g., order, ticket, application, request, document, transaction).

For each state observed, determine:

- The state's name/label as shown in the UI or data.
- What actions are available while the entity is in that state.
- What triggers a transition out of that state (user action, system event, time-based, external event).
- Which states a given state can transition to, based on observed evidence.
- Whether any transition appears irreversible.

Do not assume the full state graph if only some states/transitions have been observed — mark unobserved transitions as Open Questions rather than inferring the complete lifecycle.

---

## 4. Requirements Classification

Classify discovered information into the appropriate category.

### 4.1 Functional Requirements

Describe what the system does.

Examples:

- The system allows users to search products by keyword.
- The system displays validation feedback when required fields are empty.
- The system prevents submission when mandatory information is missing.

---

### 4.2 Business Rules

Describe business constraints or decisions that affect system behavior.

Examples:

- A transaction cannot exceed a configured limit.
- A user must complete verification before accessing a feature.

Only document a business rule when supported by evidence.

---

### 4.3 UI / Interaction Rules

Describe user-interface behavior.

Examples:

- The Submit button remains disabled until required fields are completed.
- A confirmation modal appears before deletion.

---

### 4.4 Data / Field Rules

Describe constraints associated with data fields.

Examples:

- Email must follow a valid email format.
- Password requires at least 8 characters.
- Amount accepts numeric values only.

---

### 4.5 Non-Functional Constraints (domain-agnostic)

Record any non-functional behavior that is actually observable — do not speculate about non-functional requirements that cannot be evidenced. Typical categories to check for evidence, applicable across domains (fintech, healthcare, logistics, SaaS, e-commerce, etc.):

- **Access control** — does the UI/API enforce who can see or act on data?
- **Data integrity / audit trail** — is there any visible history, versioning, or "last modified by" evidence?
- **Performance-related feedback** — loading indicators, timeouts, rate-limit messages.
- **Concurrency handling** — evidence of conflict messages (e.g., "This record was updated by another user").
- **Regulatory/compliance markers** — visible disclaimers, consent checkboxes, mandatory acknowledgments (record what is observed; do not infer the underlying regulation).

If no evidence exists for a category, omit it rather than filling in a generic statement.

---

### 4.6 Unknowns / Open Questions

Capture behavior that cannot be determined from the available evidence.

Examples:

- Is there a maximum transaction amount?
- Does the backend enforce the same validation rules as the UI?
- What happens when the downstream payment provider is unavailable?
- What roles, if any, exist beyond the one observed?
- What is the full set of valid state transitions for this entity?

Do not attempt to answer these questions through speculation.

---

## 5. Requirements Document Structure

Generate the following structure unless the user explicitly requests a different format. Include only the sections relevant to the analyzed feature's complexity (see 2.5); omit Actors & Permissions or State Machine when not applicable.

# Requirements Specification

## 1. Overview

Include:

- Feature/module name
- Purpose
- Scope
- Actors/users (list all identified, even if only one)
- Available evidence
- Complexity signals observed (multi-actor? multi-state? multi-step? — briefly note which optional sections are included and why)
- Analysis limitations

---

## 2. User Stories

For each meaningful feature:

### [Feature Name]

**User Story**

> As a [user/actor], I want to [action], so that [goal].

**Acceptance Criteria**

Use Given/When/Then where appropriate.

```text
Given [precondition]
When [user action]
Then [expected system behavior]
```

Acceptance criteria must be observable and testable.

---

## 3. Functional Requirements

Use a table when multiple requirements exist.

| ID     | Requirement | Evidence                           | Confidence          | Related IDs     |
| ------ | ----------- | ---------------------------------- | ------------------- | --------------- |
| FR-001 | ...         | UI / DOM / Runtime / Documentation | Confirmed / Derived | BR-001, VAL-001 |

Every requirement should have a unique identifier. Use the Related IDs column to link a requirement to the business rules, validations, or field specs it depends on — this makes the document easier to convert into test cases later.

---

## 4. Field Specifications

| Field | UI Type | Required | Default | Constraints | Validation | Related IDs | Notes |
| ----- | ------- | -------- | ------- | ----------- | ---------- | ----------- | ----- |
| ...   | ...     | ...      | ...     | ...         | ...        | VAL-001     | ...   |

Use `Not Observed` when the information cannot be determined.

Do not use guessed values.

---

## 5. Business Rules

| ID     | Rule | Evidence | Confidence | Related IDs |
| ------ | ---- | -------- | ---------- | ----------- |
| BR-001 | ...  | ...      | ...        | FR-001      |

---

## 6. Actors & Permissions (include only if applicable — see 3.6)

| Actor/Role | Can View | Can Do | Cannot Do / Hidden | Evidence |
| ---------- | -------- | ------ | ------------------ | -------- |
| ...        | ...      | ...    | ...                | ...      |

---

## 7. State Machine (include only if applicable — see 3.7)

| State | Entry Condition | Available Actions | Possible Next States | Evidence |
| ----- | --------------- | ----------------- | -------------------- | -------- |
| ...   | ...             | ...               | ...                  | ...      |

Note explicitly which transitions were NOT observed, rather than omitting them silently.

---

## 8. Workflows

Document important workflows using numbered steps.

Example (linear):

```text
1. User opens the feature.
2. User enters required information.
3. User submits the form.
4. System validates the input.
5. System processes the request.
6. System displays the result.
```

Example (with decision point):

```text
1. User submits the request.
2. System validates the request.
   - If valid → proceed to step 3.
   - If invalid → display validation error, remain on the same step.
3. Request enters "Pending Approval" state.
4. Approver reviews the request.
   - If approved → request moves to "Approved" state; proceed to step 5.
   - If rejected → request moves to "Rejected" state; workflow ends.
5. System executes the approved request.
```

Include alternate and error paths when they are observable. Prefer this decision-point format whenever a step's outcome depends on a role, condition, or system response, rather than forcing it into a single linear list.

---

## 9. Validation & Error Handling

| ID      | Condition               | Expected Behavior       | Message                | Related IDs |
| ------- | ----------------------- | ----------------------- | ---------------------- | ----------- |
| VAL-001 | Required field is empty | Submission is prevented | Exact observed message | FR-001      |

If a validation message cannot be observed:

```text
Message: Not Observed
```

Do not invent one.

---

## 10. States & Edge Cases

Identify observable states such as:

- Initial state
- Loading
- Empty
- Success
- Validation error
- System error
- Disabled
- Unauthorized
- Session expired
- Retry

Only include states supported by available evidence. (For entities with a formal multi-step lifecycle, prefer Section 7 State Machine instead.)

---

## 11. Dependencies

Document relevant dependencies such as:

- Authentication
- Authorization
- APIs
- External services
- Database-backed data
- Feature flags
- Browser/device requirements

Clearly distinguish observed dependencies from assumptions.

---

## 12. Open Questions

List requirements that require clarification.

| ID     | Question | Reason                                                |
| ------ | -------- | ----------------------------------------------------- |
| OQ-001 | ...      | Behavior cannot be determined from available evidence |

---

## 13. QA Considerations

Summarize areas that require particular testing attention, such as:

- Boundary validation
- Negative scenarios
- State transitions
- Actor/permission differences
- Data consistency
- Dependency failures
- Retry behavior
- Duplicate submissions
- Concurrency-sensitive behavior

Do not create test cases unless explicitly requested.

---

## 6. Analysis Workflow

Follow this sequence when executing the skill.

### Step 1 — Identify the Target

Determine:

- Page/module/feature being analyzed
- Primary user/actor(s)
- Scope of analysis
- Initial complexity signals (single vs. multi-actor, single vs. multi-state)

If the target is ambiguous, ask for clarification before proceeding.

---

### Step 2 — Collect Evidence

Inspect the available sources in the following order:

1. Explicit user-provided requirements
2. Existing product/business documentation
3. Runtime application behavior
4. DOM/HTML structure
5. Screenshots
6. Existing tests or automation code
7. Reasonable inference

Higher-priority evidence takes precedence over lower-priority evidence.

---

### Step 3 — Analyze Behavior

Identify:

- Components
- Inputs
- Actions
- Preconditions
- State transitions
- Validations
- Success behavior
- Error behavior
- Dependencies
- Actors/permissions (if applicable)

---

### Step 4 — Classify Findings

Classify each finding as:

- Confirmed
- Derived
- Assumption
- Open Question

Never silently convert an assumption into a requirement.

---

### Step 5 — Generate Requirements

Create only the sections relevant to the feature's complexity:

- User Stories
- Acceptance Criteria
- Functional Requirements
- Field Specifications
- Business Rules
- Actors & Permissions (if applicable)
- State Machine (if applicable)
- Workflows
- Validation Rules
- Edge Cases
- Dependencies
- Open Questions

---

### Step 6 — Perform Consistency Review

Before producing the final document, verify:

- No requirement contradicts observed behavior.
- No unsupported business rule has been presented as fact.
- Required fields are consistent across sections.
- Validation rules are consistent with acceptance criteria.
- Workflow steps are logically ordered, and decision points are represented as branches, not flattened into a false linear sequence.
- Actor/permission differences (if any) are consistent between Section 6 and the rest of the document.
- State transitions (if any) are consistent between Section 7 and the Workflow section.
- Unknown behavior is explicitly marked.
- No validation message has been fabricated.
- Related ID references point to IDs that actually exist in the document.
- Every important requirement has traceable evidence where possible.

---

## 7. Tool Usage

When browser automation or browser inspection tools are available, prefer runtime inspection over static assumptions.

For web applications, use browser capabilities to:

- Navigate to the target page.
- Inspect relevant UI elements.
- Interact with the feature when necessary.
- Capture screenshots when visual evidence is useful.
- Inspect DOM attributes.
- Observe state changes.
- Observe validation and error behavior.
- Inspect network/API behavior when the tool supports it.
- Where possible, interact as different roles/accounts to observe actor-based differences.

Do not perform destructive or irreversible actions unless explicitly authorized by the user.

---

## 8. Output Language

Default output language: **English**.

If the user explicitly requests another language, produce the requirements document in that language.

Technical identifiers, API names, UI labels, error messages, and code-related terminology should remain unchanged where accuracy requires it.

---

## 9. Strict Rules

1. **Evidence over assumptions.**
2. **Never fabricate requirements, business rules, validation messages, or system behavior.**
3. **Clearly distinguish confirmed behavior from derived information and unknowns.**
4. **Do not infer complex backend or business logic from UI appearance alone.**
5. **Use exact observed values whenever available.**
6. **Use `Not Observed` rather than guessing missing values.**
7. **Acceptance criteria must be testable and observable.**
8. **Requirements must be traceable to available evidence whenever possible.**
9. **Do not create test cases unless explicitly requested.**
10. **Do not modify or invent product behavior merely to make the requirements document appear complete.**
11. **Prioritize functional behavior over purely visual descriptions unless visual requirements are relevant to the target feature.**
12. **When evidence conflicts, report the conflict explicitly instead of choosing one interpretation silently.**
13. **Ask for clarification when the missing information materially affects the requirements.**
14. **Protect user data and credentials encountered during analysis; do not expose secrets in the generated document.**
15. **Do not perform destructive actions without explicit authorization.**
16. **Scale the document's depth to the feature's actual complexity — do not pad a simple feature with unused sections, and do not compress a multi-actor/multi-state feature into a flat, linear description.**

---

## 10. Definition of Done

The requirements analysis is complete when:

- The target feature is clearly defined, including its complexity signals.
- Relevant observable behavior has been analyzed.
- Functional requirements are documented.
- Acceptance criteria are testable.
- Important fields and validation rules are documented.
- Relevant workflows and states are documented, including decision points where applicable.
- Actor/permission differences are documented where applicable.
- Entity state machines are documented where applicable.
- Known business rules are separated from assumptions.
- Unknowns and open questions are explicitly identified.
- Evidence is traceable where possible, including cross-references via Related IDs.
- The final document contains no unsupported fabricated behavior.
