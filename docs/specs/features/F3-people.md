# F3 — People

_Status: Approved · Last updated: 2026-11-16 · Source: product.md — Goals, Roles, Key terms, MVP scope, Assumptions, Business Rules R3 and R7; F1 and F2 specs_

## Summary
An authenticated trip owner can add and remove named people on their trip. People are trip-local names, not accounts, and each name is unique within its trip after trimming and case-insensitive comparison. The app preserves the order people were added because Business Rule R3 uses that order to allocate leftover cents, and it prevents removal while a person is referenced by any expense or payment.

## User stories
- As a trip owner, I want to add the people on my trip so that I can include them in expenses and balances.
- As a trip owner, I want to remove a person who is no longer needed, so that the trip contains only its current participants.
- As a trip owner, I want the app to explain why an in-use person cannot be removed so that existing expense and payment records remain valid.

## Scope
**In:** Add and remove names on an owned trip, enforce trip-local name uniqueness, retain each person’s addition order, and prevent removal when any expense or payment refers to the person.

**Out:** Person accounts, editing a person’s name, and managing expenses or payments. People are names only under product.md Roles and Key terms; expenses and payments are handled by F4 and F6.

## User flow & screens
1. The authenticated owner opens a trip detail page (F2) and sees the people on that trip in the order they were added.
2. The owner enters a person’s name and submits it. The app trims surrounding whitespace and rejects an empty or duplicate name.
3. On success, the new person appears in the list after existing people, with a stable position for expense split rounding.
4. If a person is referenced by an expense or payment, the remove action is disabled and the UI explains why.
5. To remove an unused person, the owner chooses remove and confirms in a dialog identifying that person. Cancel leaves the person unchanged; confirmation removes the person from the trip.

No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F3.R1** Adding, listing, or removing people requires authentication and ownership of the parent trip, following F1. A person must belong to exactly one trip.
- **F3.R2** Adding a person requires a name. Trim surrounding whitespace and reject a name that is empty after trimming. No product-level maximum name length is set.
- **F3.R3** A person’s name must be unique within its trip after trimming and case-insensitive comparison. The same name may be used on a different trip.
- **F3.R4** A person is represented by a trip-local name and identifier; they do not have or require an account.
- **F3.R5** Preserve each person’s stable order of addition within the trip. Present people in that order and make the order available to F4 so Business Rule R3 can assign leftover cents to participants in their trip-addition order.
- **F3.R6** A person referenced as payer or participant by any expense, or as sender or recipient by any payment, cannot be removed. The server must enforce this even if the UI’s displayed state is stale.
- **F3.R7** Disable removal for a person known to be referenced and explain the restriction. If a removal request is rejected because a reference exists, show the same explanation and make no changes.
- **F3.R8** Removing an unused person requires explicit confirmation that identifies the person. Canceling or dismissing confirmation makes no changes; confirming removes the person from the trip.
- **F3.R9** Person names are personal data and must not be written to logs, consistent with project-rules.md.

## Data & contract
- A person needs a stable identifier, parent trip reference, display name, and stable per-trip addition order. Exact field names, normalization representation, maximum storage size, and order-generation mechanism are stack-specific.
- Name uniqueness must be enforced per trip using the trimmed, case-insensitive comparison in F3.R3, including against concurrent add requests.
- The list for a trip returns its people in addition order. F4 must use this order when allocating leftover cents under product.md Business Rule R3.
- Add and remove endpoint names, request/response fields, and error shapes belong in the API contract after the stack is selected. F1’s authentication and ownership outcomes apply to all person endpoints.
- A rejected removal must leave both the person and all related trip data unchanged.

## States & errors
- **Loading people:** indicate that the list is loading.
- **No people:** show an empty state and an action to add a person.
- **Blank name:** show “Enter a person’s name.” Do not create a person.
- **Duplicate name:** show “A person with this name is already on this trip.” Do not create a person.
- **Person in use:** disable removal and show “Delete linked expenses and payments before removing this person.” If the server rejects a stale removal request, show the same message.
- **Removal confirmation:** identify the selected person and provide explicit confirm and cancel actions.
- **Canceled removal:** close the dialog and leave the person unchanged.
- **Request failure:** report the failure without presenting a success state or changing the displayed person list as if the operation succeeded.
- **Unauthenticated or not-owned trip request:** follow the unauthorized/not-found behavior specified by F1; do not return or change another user’s trip data.

## Accessibility
- Provide an associated label for the person-name field and make validation errors available to assistive technology.
- The confirmation dialog must have an accessible name, move focus into the dialog when opened, keep keyboard focus within it while open, and return focus to its trigger when canceled or closed.
- A disabled removal action must have an accessible explanation of why the person cannot be removed.
- Announce successful additions/removals and loading or empty states to assistive technology; do not communicate errors by color alone.

## Edge cases
- A name containing only whitespace is invalid; surrounding whitespace on a valid name is removed before storing or comparing it.
- Names differing only by case or surrounding whitespace are duplicates within the same trip.
- The same normalized name can be used on separate trips.
- A person used as an expense payer or participant, or as either side of a payment, cannot be removed.
- If an expense or payment begins referencing a person after the UI loads, the server still rejects a later removal and preserves all records.
- Removing a person does not renumber the relative addition order of the remaining people.

## Acceptance criteria
- **F3.AC1** Given an authenticated owner opens their trip and submits a name that is non-empty after trimming, when the name is not already in use on that trip, then one person is added after existing people and appears in the trip’s people list.
- **F3.AC2** Given a submitted name is empty after trimming, when it is added, then the app shows “Enter a person’s name.” and creates no person.
- **F3.AC3** Given a trip already has a person whose trimmed name matches case-insensitively, when another matching name is submitted, then the app shows “A person with this name is already on this trip.” and creates no person; the same name remains valid on another trip.
- **F3.AC4** Given a trip has multiple people, when its people are listed and used by F4, then their stable order of addition is preserved for the remainder-cent allocation required by Business Rule R3.
- **F3.AC5** Given a person has no associated expenses or payments, when the owner starts removal and cancels or dismisses confirmation, then the person remains unchanged; when the owner confirms, then that person is removed.
- **F3.AC6** Given a person is referenced as an expense payer or participant, or by a payment, when the owner views or attempts removal, then the UI explains “Delete linked expenses and payments before removing this person.”, the server rejects removal, and the person and related records remain unchanged.
- **F3.AC7** Given a request to add, list, or remove people is unauthenticated or targets a trip not owned by the requester, when the request is processed, then it is rejected according to F1 and no unauthorized trip or person data is returned or changed.
- **F3.AC8** Given any person operation is handled, when application logging occurs, then person names are absent from logs.

## Dependencies
- F1 provides authentication and trip-ownership enforcement.
- F2 provides the parent trip and trip detail page.
- F4 consumes people and their stable addition order for expense participants and rounding.
- F6 payments can prevent person removal under product.md Business Rule R7.
- No external service is required.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F3.AC1–F3.AC3:** Unit-test trimming, empty-name validation, per-trip case-insensitive uniqueness, cross-trip name reuse, and persistence behavior using fakes.
- **F3.AC4:** Unit-test stable person ordering and its availability to expense split logic; use Business Rule R3 reference cases when F4 is implemented.
- **F3.AC5–F3.AC6:** Unit-test confirmation/cancel outcomes and removal rejection when a person is referenced by an expense or payment. Code review the query or database constraint that prevents deletion while references exist.
- **F3.AC7:** Unit-test authorization decisions and verify that denied requests return no person data and perform no writes; code review ownership predicates on every person endpoint.
- **F3.AC8:** Unit-test or inspect logging paths with representative person names and verify names are excluded.

## Open questions
- [ ] What API endpoint paths, request/response shapes, and error status/body conventions fit the selected app stack?
- [ ] What safe database storage limit and per-trip addition-order representation will be used, given that the product defines no person-name length maximum?
- [ ] What exact confirmation-dialog prompt should be used when removing an unused person?

## Decisions log
- 2026-11-16: User chose to trim person names and require them to be non-empty, without setting a product-level maximum; compare names case-insensitively after trimming; require confirmation to remove an unused person; disable removal and explain when a person is referenced; and use concise descriptive messages for blank names, duplicates, and blocked removal.
- 2026-11-16: User explicitly approved this specification.
