---
name: plan-feature
description: Create a small, implementation-ready plan for an Approved Trip Expense Tracker feature, with every acceptance criterion mapped to a named unit test or code review. Refuses specs with unresolved open questions.
argument-hint: F<N> (e.g. F6)
model: haiku
---

# Plan a feature

Create or revise the implementation plan for one approved feature. Do not implement the feature.

Target feature: **$ARGUMENTS**

## 1. Resolve the feature

- Accept an ID such as `F6`, `f6`, or `6`.
- If no ID is supplied, list the feature IDs and names available in `docs/specs/features/` and use `AskUserQuestion` to ask which one to plan.
- Find exactly one matching `docs/specs/features/F<N>-<slug>.md`. Confirm its heading and requirement/acceptance-criterion IDs match the requested feature. If it is missing, ambiguous, or inconsistent, stop and explain the issue; do not guess a matching feature.
- Use the spec filename’s slug for the plan path: `docs/plans/F<N>-<slug>.md`.

## 2. Gate on spec status and open questions

Read the complete target spec before writing or revising any plan.

- The spec must have a status line whose status value is exactly `Approved` (case-insensitive). If it is Draft, missing a status, or has another status, stop. Tell the user that `/spec-feature F<N>` must be completed and the spec explicitly approved before planning.
- The spec must contain an `## Open questions` section. Read through the next section heading. Treat unchecked checklist items (`- [ ]`, `* [ ]`, or `+ [ ]`) and any other substantive text as unresolved. Blank content, `None`, `None.`, and fully checked checklist items (`[x]` or `[X]`) are resolved. If the section is missing or malformed, stop because the gate cannot be verified.
- If either gate fails, do not create or revise a plan and do not edit the spec. Report the status and unresolved questions, then direct the user to `/spec-feature F<N>` to resolve product decisions. Do not resolve product questions in this skill.

## 3. Read implementation context

Before planning:

- Read the complete root `CLAUDE.md`.
- Read every file under `docs/rules/` recursively; do not assume there is only one rules file.
- Read `docs/specs/product.md` when the target spec relies on its business rules or worked examples; use it as context, not to override decisions recorded in the approved feature spec.
- Read any other feature specs explicitly referenced by the target spec’s dependencies or acceptance criteria.
- Read relevant existing API/data-model documentation, application source files, and tests to follow current names, structure, and patterns. Do not invent existing files or tests.
- Use `CLAUDE.md` for the stack, layout, and test commands. If a required context file is missing or contradicts the approved spec, stop and explain the conflict rather than silently choosing one.

## 4. Build the acceptance-criterion verification map

Extract every `F<N>.AC<k>` from the approved spec. The plan must map every one of them to one or more specific verification items:

- **Unit test:** name the test file and the exact proposed test name. Include the full AC ID in the test name or in a comment, as required by the project rules.
- **Code review:** use only when the project rules say behavior cannot be proven adequately by unit tests (for example, ownership predicates in database queries, database constraints, or cascades). Name the concrete file/query/constraint to review and the check to perform.

Do not use a generic “tests pass” entry as an AC mapping. Do not invent integration or end-to-end tests when the rules require unit tests only. Use fakes for database/network behavior.

Before writing the plan, verify that:

1. Every acceptance criterion in the spec appears in the verification map.
2. Every map row names a unit test or a concrete code-review check.
3. Every test maps back to one or more acceptance criteria and is included in a task.

If an acceptance criterion is ambiguous, appears impossible to verify as written, or cannot be mapped without deciding product behavior, stop and direct the user to `/spec-feature F<N>`. Never edit the spec here.

## 5. Create or revise the plan

Write the plan to `docs/plans/F<N>-<slug>.md`, using [plan-template.md](plan-template.md).

- Keep work small and ordered. Give each task a stable ID `F<N>.T<k>` and one cohesive outcome; avoid catch-all tasks such as “implement the feature.”
- Every task must list the exact files to create or modify, its actual task dependencies (or `None`), its named tests/review items, and a measurable **Done when** check.
- Include all mapped acceptance criteria in the tasks and in the plan’s verification table.
- Split setup, domain logic, persistence/API, UI, and verification into separate tasks only when they are real, independently checkable work. Keep tightly coupled changes together.
- Use the existing code layout and conventions. If the application does not exist yet, use only the planned layout documented in `CLAUDE.md` and label files as new.
- If the product spec, repository rules, or another required source conflicts with the feature spec and the feature spec does not explicitly resolve that conflict, stop and direct the user to `/spec-feature F<N>`.
- Keep implementation choices within the approved spec and repository rules. Do not expand scope or add product behavior.
- If the plan already exists, read it first, preserve its Decisions log and user-authored decisions, and revise it without discarding unrelated content. If a revision changes an Approved plan, return it to Draft and record why in its Decisions log.
- Never edit the feature spec, product spec, or project rules from this command.

## 6. Finish

- Save the plan with status `Draft` and today’s date.
- Summarize the task IDs and acceptance-criterion coverage.
- Ask whether the user approves the plan. Change its status to `Approved` and record today’s date only after explicit approval. If approval is not given, leave it Draft.
- Do not implement tasks in this skill. If the user asks for implementation, use the appropriate implementation workflow after the plan is approved.
