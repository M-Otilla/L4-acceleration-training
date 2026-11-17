# F<N> — <Feature name> implementation plan

_Status: Draft · Last updated: <YYYY-MM-DD> · Source: `docs/specs/features/F<N>-<slug>.md`_

## Goal
<Briefly state the approved feature outcome. Do not add behavior beyond the spec.>

## Context and constraints
- Stack and layout: <from `CLAUDE.md`>
- Rules to preserve: <relevant rules from `docs/rules/`>
- Dependencies: <feature IDs or tasks this plan relies on>
- Out of scope: <relevant exclusions from the approved spec>

## Tasks

### F<N>.T1 — <Small, cohesive task>

**Files**
- `<path>` — create|modify: <specific responsibility>

**Work**
- <Concrete change that stays within the approved spec.>

**Depends on:** None | `F<N>.T<k>`

**Verification**
- `<test path>` — `<Exact test name containing F<N>.AC<k>>`
- Or: **Code review** — `<specific file/query/constraint and check>`

**Done when**
- <Observable result and the exact test, command, or review check that proves completion.>

## Acceptance-criterion verification map

| Acceptance criterion | Planned verification | Task |
|---|---|---|
| `F<N>.AC1` | `<test path>` — `<exact named unit test>` OR **Code review** — `<concrete check>` | `F<N>.T1` |

Include one or more rows for every acceptance criterion in the spec. Never leave an AC unmapped.

## Plan checks
- [ ] Every AC in the spec is mapped to a named unit test or a concrete code-review check.
- [ ] Every task has an ID, exact files, dependencies, verification, and a measurable done-when check.
- [ ] Tests follow the repository rules and use the commands in `CLAUDE.md`.
- [ ] No product question or unresolved spec behavior was decided in this plan.

## Open implementation questions
<List only questions about implementation details that do not change product behavior, or write `None.`. Product questions must return to `/spec-feature F<N>`, not be answered here.>

## Decisions log
- <YYYY-MM-DD>: <plan decision and rationale, or explicit user approval>
