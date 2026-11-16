# F2 — Trips

_Status: Draft · Last updated: 2026-11-16 · Source: product.md — Goals, Roles, Assumptions, Key terms, MVP scope, Business rules R8_

## Summary
An authenticated user can create a trip with a name, view their trips, and delete a trip they own. Each trip is associated with its creating user, and the trip endpoints enforce the ownership and authentication rules defined by F1. Deleting a trip requires confirmation and removes all data contained in that trip.

## User stories
- As a user, I want to create a named trip so that I can organize shared expenses.
- As a user, I want to see only my trips so that I can find and manage my own travel groups.
- As a trip owner, I want to delete a trip after confirming so that a mistake or finished trip can be removed along with its data.

## Scope
**In:** Create a trip with a name, list the authenticated user’s trips, and delete an owned trip after explicit confirmation. A new trip is owned by the authenticated user who created it.

**Out:** Editing trip names, creating people or expenses as part of trip creation, sharing a trip with another account, currencies, and deleting individual trip contents through this feature. People, expenses, and payments are managed by F3, F4, and F6 respectively. Trip deletion removes these contained records as required by product.md Business Rule R8.

## User flow & screens
1. An authenticated user opens the trip list. Only trips they own are shown, newest first.
2. The user chooses to create a trip and enters a name. The name is trimmed and must contain 1–100 characters.
3. On success, the trip is created for the authenticated user and the app opens its trip detail page, where the user can add people (F3).
4. From the trip list, the user chooses to delete a trip. A confirmation dialog identifies the trip and offers confirm and cancel actions.
5. Cancel closes the dialog and leaves the trip and its contents unchanged. Confirm deletes the trip and all data belonging to it, then returns the user to the trip list.

No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F2.R1** Creating a trip requires authentication. The owner is taken from the authenticated identity, not from client-supplied ownership data.
- **F2.R2** A trip name is required. Trim surrounding whitespace, then accept names from 1 through 100 characters. Do not impose uniqueness across a user’s trips; duplicate names are allowed.
- **F2.R3** On successful creation, persist a trip identifier, name, owner, and creation time, then open the new trip’s detail page.
- **F2.R4** The trip list contains only trips owned by the authenticated user and is ordered by creation time, newest first. An empty list is a valid state.
- **F2.R5** Deleting a trip requires an explicit confirmation that identifies the trip. Canceling confirmation makes no changes.
- **F2.R6** After confirmation, delete the trip and all people, expenses, and payments contained in it. Perform the multi-record deletion atomically, consistent with project-rules.md.
- **F2.R7** All create, list, and delete operations follow F1 authentication and ownership requirements. A user must not be able to read or delete another user’s trip by changing an identifier or making a direct API request.

## Data & contract
- A trip requires a stable identifier, name, owner reference, and creation timestamp. Store timestamps in UTC as required by project-rules.md. Exact field names and storage types are determined by the selected stack.
- The list operation returns only the authenticated user’s trips, ordered newest first. Create, list, and delete endpoint names, request/response shapes, and error formats are to be defined in the API contract after the stack is selected.
- Deletion must remove all records contained by the trip in one transaction. Cascades and database constraints are subject to code review as required by project-rules.md.
- No currency or other trip-level settings are introduced; the product specifies one currency without a per-trip setting.

## States & errors
- **Loading list:** indicate that trips are being loaded.
- **Empty list:** explain that the user has no trips yet and provide an action to create one.
- **Create validation failure:** show “Enter a trip name” for a blank name and “Trip names must be 100 characters or fewer” for an overlong name; do not create a trip.
- **Create failure:** keep the entered name available for correction or retry and report the failure without presenting a success state.
- **Delete confirmation:** identify the selected trip and provide clearly labeled confirm and cancel actions.
- **Delete failure:** keep the trip visible and report that it was not deleted.
- **Unauthenticated or not-owned trip request:** follow the unauthorized/not-found behavior specified by F1; do not return or change another user’s trip data.

## Accessibility
- Provide an associated label for the trip-name field and make validation errors available to assistive technology.
- The confirmation dialog must have an accessible name, move focus into the dialog when opened, keep keyboard focus within it while open, and return focus to its trigger when canceled or closed.
- Confirm and cancel actions must be distinguishable without relying on color alone and operable by keyboard.
- Announce successful creation/deletion and list loading or empty states to assistive technology.

## Edge cases
- Names containing only whitespace are invalid after trimming.
- Names that differ only by surrounding whitespace are stored without that whitespace; names that otherwise duplicate another trip’s name are allowed.
- A user with no trips sees the empty state rather than an error.
- Canceling deletion, dismissing the dialog, or a failed delete leaves the trip and all its contents intact.
- A trip identifier belonging to another user must not be treated as permission to access or delete that trip.
- If deletion fails partway through, the transaction must leave the trip and all its contents intact.

## Acceptance criteria
- **F2.AC1** Given an authenticated user enters a trimmed trip name of 1–100 characters, when they create the trip, then one trip is persisted with that user as owner and the user is taken to the new trip’s detail page.
- **F2.AC2** Given a trip name is blank after trimming or exceeds 100 characters after trimming, when creation is submitted, then validation fails and no trip is created.
- **F2.AC3** Given a user already has a trip with a particular name, when they create another trip with the same name, then creation succeeds and the trips have distinct identifiers.
- **F2.AC4** Given a user owns trips and another user owns different trips, when the first user opens the trip list, then only the first user’s trips are returned, newest first; if they own none, the list is empty.
- **F2.AC5** Given a user starts deleting an owned trip, when they cancel or dismiss the confirmation, then the trip and all its contents remain unchanged.
- **F2.AC6** Given a user confirms deletion of an owned trip, when deletion succeeds, then the trip and all its contained people, expenses, and payments are deleted atomically and the user returns to the trip list.
- **F2.AC7** Given a user is unauthenticated or does not own the requested trip, when they call a trip endpoint directly, then the request is rejected according to F1 and no unauthorized trip data is returned or changed.

## Dependencies
- F1 provides authentication and trip-ownership enforcement.
- F2 is a prerequisite for F3–F6, which add people, expenses, balances, and payments to a trip.
- F3 and later features provide the trip detail content; no external service is required.
- The application stack must be selected before fixing API shapes, persistence details, and cascade implementation.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F2.AC1–F2.AC3:** Unit-test name validation, owner assignment from authenticated identity, duplicate-name behavior, successful creation, and absence of records on invalid input using fakes.
- **F2.AC4:** Unit-test owner-scoped listing, newest-first ordering, and the empty-list state.
- **F2.AC5–F2.AC6:** Unit-test confirmation/cancel outcomes and transaction behavior using fakes. Code review the database cascade/constraints to verify all dependent records are deleted atomically.
- **F2.AC7:** Unit-test authorization decisions and verify that denied requests return no trip data and perform no writes; code review ownership predicates on every trip endpoint as required by project-rules.md.

## Open questions
- [ ] What API endpoints, persistence schema, and exact error response shapes fit the application stack selected in `CLAUDE.md`?
- [ ] What implementation-specific timestamp representation and cascade mechanism will be used? Resolve these in the data model and implementation plan after the stack is selected.

## Decisions log
- 2026-11-16: Clarification response indicated the user was unavailable and requested pragmatic defaults → selected trimmed names of 1–100 characters, allowed duplicate names, newest-first ordering, and navigation to the new trip detail page after creation. Product Rule R8 supplies the requirement for confirmation before deletion; a standard confirmation dialog with confirm/cancel actions was used as the UI default.
