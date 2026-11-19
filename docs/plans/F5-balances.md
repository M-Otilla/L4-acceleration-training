# F5 — Balances implementation plan

_Status: Draft · Last updated: 2026-11-17 · Source: `docs/specs/features/F5-balances.md`_

## Goal
Calculate and show every person's current balance and component breakdown for an owned trip, using integer cents and preserving the zero-sum invariant.

## Context and constraints
- Stack and layout: Vue 3, Vite, and JavaScript in `app/web/`; Node.js, Express, JavaScript, and `mysql2` in `app/api/`. Use Vitest unit tests in each app. The apps and package manifests do not exist yet; all app files listed below are new and follow the planned `app/web/` and `app/api/` layout.
- Rules to preserve: use integer cents, parameterized MySQL queries, UTC timestamps, and fakes for database/network behavior in unit tests. Each test name includes its full acceptance-criterion ID. Review SQL ownership predicates; do not persist a separate balance record.
- Dependencies: F1 supplies authentication and ownership; F2 supplies trips; F3 supplies people and their stable addition order; F4 supplies cent-accurate expense shares; F6 supplies recorded payments. Use their established contracts and source data.
- Out of scope: suggested payments and marking them paid (F6), editing or manually adjusting balances, currency selection or conversion.

## Tasks

### F5.T1 — Calculating derived balances

**Files**
- `app/api/src/services/balances.js` — create: calculate per-person paid, share, payment, and net components from current source records using integer cents.
- `app/api/test/services/balances.test.js` — create: unit-test calculations with in-memory fixtures; no database or network.

**Work**
- Calculate net balances using amounts paid minus expense shares plus recorded payments made minus recorded payments received.
- Consume F4's cent-accurate share allocations in F3 person-addition order rather than duplicating the expense-splitting rule.
- Exclude unpaid suggestions; include a payment only when it is recorded.
- Return derived values without storing balance records.

**Depends on:** F3 and F4 source data and share contracts; F6 recorded-payment contract.

**Verification**
- `app/api/test/services/balances.test.js` — `F5.AC1 calculates each net balance from paid, shares, and recorded payments`
- `app/api/test/services/balances.test.js` — `F5.AC2 uses F4 cent-accurate shares in F3 addition order and sums to the expense total`
- `app/api/test/services/balances.test.js` — `F5.AC5 excludes unpaid suggestions and includes payments after they are marked paid`
- `app/api/test/services/balances.test.js` — `F5.AC7 preserves the zero-sum invariant for populated and empty trips`

**Done when**
- All four named Vitest unit tests pass with integer-cent fixtures, including product.md Worked examples 1 and 2, a rounding-remainder case, and an empty trip.

### F5.T2 — Serving owned-trip balances

**Files**
- `docs/specs/api.md` — create: document the read-only `GET /api/trips/:tripId/balances` contract, response components, and F1 authorization outcomes.
- `app/api/src/routes/tripBalances.js` — create: authenticate the request and expose the balance read operation.
- `app/api/src/repositories/tripBalances.js` — create: retrieve current people, expense shares, and recorded payments for the requested owned trip using parameterized queries.
- `app/api/test/routes/tripBalances.test.js` — create: unit-test route behavior with fake authentication, repository, and balance service dependencies.

**Work**
- Return freshly derived balances for an authenticated owner; do not return data or perform writes when authentication or ownership fails.
- Query source records on each balance read so the result reflects current trip people, expenses, shares, and payments.
- Keep database reads parameterized and constrain trip data to both the requested trip and authenticated owner.

**Depends on:** `F5.T1`; F1 authentication/ownership and F2–F4/F6 persistence contracts.

**Verification**
- `app/api/test/routes/tripBalances.test.js` — `F5.AC6 recalculates balances from updated source records`
- `app/api/test/routes/tripBalances.test.js` — `F5.AC8 rejects unauthenticated and non-owner requests without returning trip or balance data or writing`
- **Code review** — `app/api/src/repositories/tripBalances.js`: verify every trip-scoped source query binds the requested trip ID and authenticated owner ID in its ownership predicate.

**Done when**
- The named route tests pass using fakes and confirm denied requests return no trip or balance data; the API contract documents the read-only endpoint and response; and code review confirms the owner predicate on every trip-scoped query.

### F5.T3 — Displaying and refreshing balances

**Files**
- `app/web/src/api/tripBalances.js` — create: request balances from the API.
- `app/web/src/views/TripBalances.vue` — create: display the ordered balance rows, component breakdown, and empty state.
- `app/web/src/views/TripBalances.test.js` — create: unit-test presentation and refresh behavior with a fake API client.

**Work**
- Display people in trip-addition order with paid, shares, payments made, payments received, and net values formatted to two decimal places.
- Show “is owed,” “owes,” or “Settled up” for positive, negative, or zero balances; include the zero amount as 0.00 and provide an empty state when there are no people.
- Show loading and empty states; reload current balances after the app's trip-record update flow signals a person, expense, or recorded-payment change. Report request failures without presenting stale values as current.
- Keep the view keyboard-operable and ensure component labels, status, loading, empty, and update announcements are accessible and status is not communicated by color alone.

**Depends on:** `F5.T2`; F3/F4/F6 trip-record update flows.

**Verification**
- `app/web/src/views/TripBalances.test.js` — `F5.AC3 renders a zero-activity person in order as Settled up 0.00`
- `app/web/src/views/TripBalances.test.js` — `F5.AC4 renders positive, negative, and zero statuses with two decimal places`
- `app/web/src/views/TripBalances.test.js` — `F5.AC6 displays loading and replaces stale balances after source records change`
- `app/web/src/views/TripBalances.test.js` — `F5.AC6 reports refresh failure without presenting stale balances as current`
- `app/web/src/views/TripBalances.test.js` — `F5.AC7 renders an empty state without balance rows when a trip has no people`

**Done when**
- The named Vitest component tests pass for ordered zero-balance rows, all three net-status cases, two-decimal formatting, accessible loading/empty states, and refreshed/error results after changed source data.

## Acceptance-criterion verification map

| Acceptance criterion | Planned verification | Task |
|---|---|---|
| `F5.AC1` | `app/api/test/services/balances.test.js` — `F5.AC1 calculates each net balance from paid, shares, and recorded payments` | `F5.T1` |
| `F5.AC2` | `app/api/test/services/balances.test.js` — `F5.AC2 uses F4 cent-accurate shares in F3 addition order and sums to the expense total` | `F5.T1` |
| `F5.AC3` | `app/web/src/views/TripBalances.test.js` — `F5.AC3 renders a zero-activity person in order as Settled up 0.00` | `F5.T3` |
| `F5.AC4` | `app/web/src/views/TripBalances.test.js` — `F5.AC4 renders positive, negative, and zero statuses with two decimal places` | `F5.T3` |
| `F5.AC5` | `app/api/test/services/balances.test.js` — `F5.AC5 excludes unpaid suggestions and includes payments after they are marked paid` | `F5.T1` |
| `F5.AC6` | `app/api/test/routes/tripBalances.test.js` — `F5.AC6 recalculates balances from updated source records`; `app/web/src/views/TripBalances.test.js` — `F5.AC6 displays loading and replaces stale balances after source records change`; `app/web/src/views/TripBalances.test.js` — `F5.AC6 reports refresh failure without presenting stale balances as current` | `F5.T2`, `F5.T3` |
| `F5.AC7` | `app/api/test/services/balances.test.js` — `F5.AC7 preserves the zero-sum invariant for populated and empty trips`; `app/web/src/views/TripBalances.test.js` — `F5.AC7 renders an empty state without balance rows when a trip has no people` | `F5.T1`, `F5.T3` |
| `F5.AC8` | `app/api/test/routes/tripBalances.test.js` — `F5.AC8 rejects unauthenticated and non-owner requests without returning trip or balance data or writing`; **Code review** — ownership predicate in `app/api/src/repositories/tripBalances.js` | `F5.T2` |

## Plan checks
- [x] Every AC in the spec is mapped to a named unit test or a concrete code-review check.
- [x] Every task has an ID, exact files, dependencies, verification, and a measurable done-when check.
- [x] Tests follow the repository rules and use the commands in `CLAUDE.md`.
- [x] No product question or unresolved spec behavior was decided in this plan.

## Open implementation questions
None.

## Decisions log
- 2026-11-17: Use a read-only `GET /api/trips/:tripId/balances` endpoint and derive balances from current source records rather than adding persisted balance rows; this follows F5's derived-balance model and is documented in `docs/specs/api.md`.
