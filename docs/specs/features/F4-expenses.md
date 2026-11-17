# F4 — Expenses

_Status: Draft · Last updated: 2026-11-17 · Source: product.md — Overview, Goals, Roles, Assumptions, Key terms, MVP scope, Business Rules R1–R3 and R7, Worked examples; F1–F3 specs; CLAUDE.md and project-rules.md_

## Summary
An authenticated trip owner can add, list, and delete expenses in their trip. Each expense records a description, an amount in whole cents, one payer, and at least one participant; everyone on the trip is selected by default. Expenses are split equally, with leftover cents allocated in the order participants were added to the trip.

## User stories
- As a trip owner, I want to record what was paid, how much, who paid, and who shares it so that the trip’s expenses are tracked.
- As a trip owner, I want everyone selected as a participant by default, while still being able to choose a subset, so that recording common expenses takes fewer steps.
- As a trip owner, I want to see and delete expenses so that I can correct a mistake by removing it and adding the correct entry.

## Scope
**In:** Add an expense with a description, amount, payer, and participants; default participants to everyone on the trip; list expenses; and delete an expense after confirmation. Enforce the business rules for amount, payer, participants, equal split, and leftover cents.

**Out:** Editing expenses, uneven splits, currency selection or conversion, and adding a manually chosen date or other expense fields. Editing and uneven splits are listed under “After the core” in product.md; the product specifies one currency and the four expense inputs.

## User flow & screens
1. The authenticated owner opens an owned trip and its expense list.
2. The owner chooses to add an expense and enters a description and amount, selects one payer, and reviews the participant selection. All people on the trip are selected initially; the owner may deselect people but must leave at least one selected.
3. On submission, the server verifies the trip, payer, and participants belong to the same owned trip, converts the positive amount to whole cents using the selected rounding rule, and records the expense.
4. The expense list refreshes with the new expense at the top. Each entry shows its description, total amount, payer, and participants.
5. To delete an expense, the owner chooses delete and confirms in a dialog that says, “Delete ‘{description}’ ({amount})? This cannot be undone.” Cancel leaves it unchanged; confirmation removes it and returns the owner to the expense list.

If a trip has no people, the owner cannot create an expense until people are added (F3). No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F4.R1** Adding, listing, or deleting expenses requires authentication and ownership of the parent trip, following F1. The server must check ownership before returning or changing expense data.
- **F4.R2** An expense requires a description. Trim surrounding whitespace and reject a description that is empty after trimming. No product-level maximum description length is set.
- **F4.R3** An expense amount must be positive and is stored as a whole number of cents, never as a floating-point amount. Accept decimal amounts with additional fractional digits and round to the nearest cent; exact half-cent ties round up. Reject an amount if its rounded result is zero or less. There is no product-level maximum amount.
- **F4.R4** Each expense has exactly one payer and at least one participant. The payer and all participants must belong to the same trip as the expense. The payer is not required to be a participant.
- **F4.R5** When adding an expense, select every person on the trip as a participant by default. The owner may change the selection, but submission is unavailable while no participants are selected. Participant ordering for split allocation follows F3’s stable trip-addition order, not the order in which checkboxes were selected.
- **F4.R6** Split the total amount equally among the selected participants: each starts with the amount divided by the participant count and rounded down to a cent. Give leftover cents one at a time to participants in their order of addition to the trip, as required by product.md Business Rule R3.
- **F4.R7** List only expenses in the authenticated owner’s trip, newest-added first. Each list item includes its description, total amount, payer, and participants.
- **F4.R8** Deleting an expense requires explicit confirmation that identifies the expense and warns that deletion cannot be undone. Show “Delete ‘{description}’ ({amount})? This cannot be undone.” Canceling or dismissing confirmation makes no changes. On confirmation, delete the expense and its participant associations atomically, consistent with project-rules.md.
- **F4.R9** Expense records are not editable. Deleting an expense removes that expense’s contribution to the balances; separately recorded payments remain, and F5/F6 calculations use the remaining expenses and payments according to product.md Business Rule R4.

## Data & contract
- An expense requires a stable identifier, parent trip reference, trimmed description, integer amount in cents, one payer reference, participant references, and a creation timestamp. Store timestamps in UTC as required by project-rules.md.
- Store participant associations so the split calculation can retrieve participants in their stable F3 person-addition order.
- Expense creation and deletion, including associated participant rows, must be atomic. Access MySQL through `mysql2` using parameterized SQL as required by CLAUDE.md.
- Parse and round the entered decimal amount using exact decimal/integer arithmetic; do not use floating-point arithmetic for money.
- Endpoint names, request/response fields, maximum MySQL storage bounds, and error response shapes belong in the API contract and data model once established. The product-level rules in this spec still apply.

## States & errors
- **Loading expenses:** indicate that the list is loading.
- **No expenses:** show an empty state and an action to add an expense.
- **No people on trip:** explain that people must be added before an expense can be recorded.
- **Blank description:** show “Enter a description.” Do not create an expense.
- **Invalid or rounded-to-zero amount:** show “Enter an amount that rounds to at least 0.01.” Do not create an expense.
- **No participants selected:** disable submission and explain that at least one participant is required.
- **Create failure:** keep entered values available for correction or retry and do not present a success state.
- **Delete confirmation:** show “Delete ‘{description}’ ({amount})? This cannot be undone.” Provide explicit confirm and cancel actions.
- **Canceled deletion:** close the dialog and leave the expense unchanged.
- **Delete failure:** keep the expense visible and report that it was not deleted.
- **Unauthenticated or not-owned trip request:** follow the unauthorized/not-found behavior specified by F1; do not return or change another user’s trip data.

## Accessibility
- Provide associated labels for description, amount, payer, and participant controls. Identify required fields, validation errors, and the at-least-one-participant constraint to assistive technology.
- Use numeric decimal input semantics for the amount without allowing browser floating-point conversion to alter the stored value.
- Participant controls must be keyboard operable and expose their selected state.
- The delete confirmation dialog must have an accessible name, move focus into the dialog when opened, keep keyboard focus within it while open, and return focus to its trigger when canceled or closed.
- Announce successful additions/deletions and loading or empty states to assistive technology; do not communicate errors by color alone.

## Edge cases
- Blank or whitespace-only descriptions are invalid after trimming.
- A positive input smaller than half a cent may round to zero; reject it because Business Rule R1 requires an amount greater than zero.
- For a positive exact half-cent, round up to the next cent (for example, 1.005 becomes 1.01).
- A payer may be excluded from the participant selection; the payer still must be a person on the same trip.
- With no people, or after deselecting everyone, no expense can be submitted.
- When a trip has three participants and an amount of 10.00, allocate 3.34 to the participant added first and 3.33 to each of the next two, as in product.md Worked example 2.
- Deleting an expense does not delete independently recorded payments. Balances and suggestions are recalculated from the remaining expense and payment records.
- A user cannot read, add, or delete an expense in another user’s trip by changing identifiers or calling the API directly.

## Acceptance criteria
- **F4.AC1** Given an authenticated owner submits a non-empty trimmed description, a positive amount, a payer, and at least one participant from their trip, when creation succeeds, then one expense is recorded with its amount in integer cents and the selected payer and participants.
- **F4.AC2** Given an amount contains more than two fractional digits, when it is converted to cents, then it is rounded to the nearest cent with exact half-cent ties rounded up (for example, 1.005 becomes 1.01).
- **F4.AC3** Given a description is blank after trimming, or an amount is non-positive or rounds to zero, when creation is submitted, then the corresponding validation message is shown and no expense is created.
- **F4.AC4** Given a trip has people, when the owner opens the add-expense form, then everyone is selected by default; the owner can change the participants and payer independently, but cannot submit with zero participants.
- **F4.AC5** Given a valid expense has multiple participants, when its shares are calculated, then the shares sum exactly to the total cents, are equal except for leftover cents, and leftover cents go in the participants’ trip-addition order as required by Business Rule R3.
- **F4.AC6** Given an owner has expenses in a trip, when they open the expense list, then only that trip’s expenses are shown newest-added first with description, amount, payer, and participants.
- **F4.AC7** Given an owner starts deleting an expense, when they cancel or dismiss confirmation, then the expense and its participant associations remain; when they confirm, then those records are deleted atomically and balances use the remaining expenses and payments.
- **F4.AC8** Given a request to add, list, or delete expenses is unauthenticated or targets a trip not owned by the requester, when the request is processed, then it is rejected according to F1 and no unauthorized trip or expense data is returned or changed.

## Dependencies
- F1 provides authentication and trip-ownership enforcement.
- F2 provides the parent trip.
- F3 provides people, payer/participant identities, and their stable addition order.
- F5 uses expense amounts and splits to calculate balances; F6 uses those balances for suggested payments.
- No external service is required.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F4.AC1–F4.AC3:** Unit-test description and amount validation, exact decimal-to-cent rounding including half-cent ties and values that round to zero, and record creation/no creation using fakes.
- **F4.AC4–F4.AC5:** Unit-test default-all participant selection, independent payer selection, minimum participant validation, equal shares, and leftover-cent allocation in trip-addition order. Use product.md Worked examples 1 and 2 as reference cases.
- **F4.AC6:** Unit-test owner-scoped listing, newest-first order, and list contents.
- **F4.AC7:** Unit-test confirmation/cancel outcomes and transaction behavior using fakes. Code review the database constraints and transaction to verify expense and participant records are removed atomically.
- **F4.AC8:** Unit-test authorization decisions and verify denied requests return no expense data and perform no writes; code review ownership predicates on every expense endpoint as required by project-rules.md.

## Open questions
None.

## Decisions log
- 2026-11-17: User chose to trim and require a non-empty description without setting a product-level maximum; round additional fractional digits to the nearest cent with ties rounded up; set no product-level amount cap; require confirmation before deleting; list newest-added expenses first; use descriptive validation messages; identify the expense and warn that deletion cannot be undone; and disable submission with an explanation when no participants are selected.
- 2026-11-17: User explicitly approved this specification.
- 2026-11-17: Reopened the approved spec to resolve its remaining confirmation-copy question; the spec returns to Draft because the confirmed wording changes its substance. The user was unavailable, so selected “Delete ‘{description}’ ({amount})? This cannot be undone.” to identify the expense clearly and warn about the destructive action. API routes, error shapes, physical MySQL storage limits, and accepted decimal precision remain implementation/data-model details; no product-level description or amount maximum is introduced.
