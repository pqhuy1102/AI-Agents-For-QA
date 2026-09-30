---
description: One-time (re-runnable) workflow that captures project-level context — PROJECT code, modules, Jira key, environments, tech stack, supported browsers/devices — into docs/project-context.md so other QA workflows can read it instead of re-asking.
---

1. Identify the target project. If the workspace contains more than one project (e.g. a monorepo) or the target is unclear, ask which one before doing anything else. Then check whether `docs/project-context.md` already exists for it. If it does, read it and continue in update mode (see Step 9); otherwise continue in create mode.

2. Inspect the workspace read-only for evidence before asking anything: repository layout, README, build/dependency files (e.g. `package.json`, `pom.xml`, `requirements.txt`), automation framework configs, existing `tests/` and `docs/` folders (including any `tests/manual/` or `docs/requirements/` content), and the names of CI configuration files. Never open `.env` files, key files, or anything that looks like a secret store; if one is encountered, note that it exists without reading its contents.

3. Build a draft from what was found. Label every field as one of: **Confirmed** (stated by the person), **Detected** (found in files, still needs confirmation), or **Not Provided**. Do not guess values that cannot be observed: PROJECT code, Jira project key, environment URLs, supported browsers/devices, and requirement ID convention must come from the person or an explicit document, never from inference.

4. Ask the person for the missing or unconfirmed fields in one grouped message, not one question at a time, skipping anything already Confirmed. Fields, in priority order:
   - PROJECT code (short, stable identifier used in TC IDs)
   - Module list (name for each module)
   - Jira project key
   - Test environments (name, base URL, purpose)
   - Automation stack: API and/or UI framework and language, or "none yet"
   - Supported browsers/devices
   - Requirement ID convention (e.g. `FR-<MODULE>-NNN`), if the project has one
   - Known constraints or project-specific conventions that differ from the general rules

   The person may skip any field; a skipped field is recorded as `Not Provided`, not filled with a default.

5. Handle sensitive input safely (Global Rule Section 4.2/4.3). If the person supplies a password, token, API key, connection string, or a URL with credentials embedded, do not write it to the file or repeat it in chat. Record only where it lives (for example, the environment variable or secret-manager entry name) and tell the person what was left out.

6. Normalize identifiers so they match what the other workflows use:
   - PROJECT code: uppercase, stable, no spaces (per `manual_testcases_rule` Section 13).
   - For each module, record two forms: a lowercase kebab-case slug for folder and file names (`<module>`, e.g. `docs/requirements/<module>-requirements-spec.md`, `tests/manual/<module>/`) and an uppercase code for TC IDs (`<MODULE>`, e.g. `CRM_LOGIN_TC_001`).
   - If two modules would produce the same slug or code, ask the person to disambiguate.
   - If the project already uses a different TC ID format, record it under Project-Specific Overrides only when the person states it. The project override then takes precedence for that project, per the "more specific rule overrides" clause of the Global Rule.

7. Present the complete draft to the person, with every field's label (Confirmed / Detected / Not Provided) visible, and ask for confirmation. Do not save anything before the person confirms. This is a human checkpoint.

8. In create mode, after confirmation, save the file to `docs/project-context.md` (create the `docs/` folder only if it is missing) using this structure, written in English:

   ```markdown
   # Project Context: <Project Name>

   Last confirmed: <YYYY-MM-DD>

   ## Identity

   - PROJECT code: <CODE>
   - Project name: <name>
   - Application type / domain: <value or Not Provided>
   - Repository / workspace root: <path or Not Provided>

   ## Modules

   | Module | Slug (`<module>`) | Code (`<MODULE>`) | Notes |
   | ------ | ----------------- | ----------------- | ----- |

   ## Requirements & Traceability

   - Requirement sources: <docs, tickets, or Not Provided>
   - Requirement ID convention: <pattern or Not Provided>

   ## Environments

   | Name | Base URL | Purpose | Credentials location (name only, never the value) |
   | ---- | -------- | ------- | ------------------------------------------------- |

   ## Tech Stack & Automation

   - Application stack: <value or Not Provided>
   - API automation: <framework/language or None>
   - UI automation: <framework/language or None>
   - Automation repo/path: <path or Not Provided>
   - CI tool: <name or Not Provided>

   ## Supported Browsers / Devices

   - <list or Not Provided>

   ## Defect Tracking

   - Tool: Jira
   - Jira project key: <KEY or Not Provided>

   ## Project-Specific Overrides

   - <none, or convention differences the person stated>

   ## Known Constraints & Notes

   - <notes or none>
   ```

   Create or modify only this one file. Do not create module folders, edit code, change configuration, or touch pipeline files as part of onboarding.

9. In update mode (the file already exists), change only the fields the person named or newly confirmed, and keep every other field exactly as it is. Show a field-level diff (old value → new value) and ask for confirmation before writing. Never blank out or overwrite an existing value silently, and update `Last confirmed` only for fields that were actually re-confirmed in this run.

10. Report the result to the person:
    - The saved path, and whether the file was created or updated.
    - Which fields are Confirmed, which were Detected and then confirmed, and which remain `Not Provided`.
    - That other QA workflows are meant to read `docs/project-context.md` first and ask only about fields still `Not Provided`; where a workflow does not read it yet, the person can point the agent at the file explicitly.
    - That the file describes the project as of `Last confirmed`, and this workflow should be re-run when modules, environments, or the tech stack change.
