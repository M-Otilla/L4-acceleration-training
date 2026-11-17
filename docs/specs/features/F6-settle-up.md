# F6 — Settle up

_Status: Approved · Last updated: 2026-11-17 · Source: product.md — Overview, Goals, Roles, Assumptions, Key terms, MVP scope, Business Rules R4–R7, Worked examples; F1–F5 specs; CLAUDE.md and project-rules.md_

## Summary
Show the payments that would settle an owned trip’s current balances, and let the owner mark a suggested payment as paid after making the money transfer outside the app. Each marked suggestion becomes a recorded payment that changes balances and causes suggestions to be recalculated. The owner can delete a recorded payment after confirmation; this changes the app’s records and balances but does not reverse money moved outside the app.

## User stories
- As a trip owner, I want the app to suggest who should pay whom and how much so that the trip can be settled with as few payments as possible.
- As a trip owner, I want to mark a suggested payment as paid after making it outside the app so that balances reflect the transfer.
- As a trip owner, I want to delete a recorded payment after confirmation so that I can correct the app’s ledger if I recorded a payment by mistake.

## Scope
**In:** Calculate suggested payments using product.md Business Rule R6, mark a current suggestion as paid, list recorded payments separately from outstanding suggestions, and delete a recorded payment after confirmation. Keep payment amounts in whole cents and update balances and suggestions from current trip records.

**Out:** Transferring money, payment-provider integrations, editing a payment, currencies or currency conversion, and manual payment creation that is not based on a current suggestion. Product.md Assumptions state that money moves outside the app and that the app only records a payment that was made.

## User flow & screens
1. The authenticated owner opens the settle-up view for an owned trip.
2. The app shows outstanding suggestions in the order produced by the settlement algorithm, each as a payer, recipient, and amount. Recorded payments appear separately, newest first.
3. After the owner has made a suggested transfer outside the app, they choose its “Mark as paid” action. This action itself is the owner’s confirmation; no second dialog is shown.
4. The app records the payment, updates balances, and recalculates outstanding suggestions from those new balances. It also shows the payment in the recorded-payments section.
5. To delete a recorded payment, the owner chooses delete. A confirmation dialog identifies the payer, recipient, and amount and warns that deleting the record changes balances. Cancel leaves it unchanged; confirmation deletes the record and recalculates balances and suggestions.
6. When all balances are zero, show no outstanding suggestions and indicate that the trip is settled.

No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F6.R1** Reading suggestions or recorded payments, marking a suggestion as paid, or deleting a payment requires authentication and ownership of the parent trip, following F1.
- **F6.R2** Calculate suggestions from the current F5 balances. Repeatedly select the person who owes the most and the person who is owed the most; suggest that the debtor pays the creditor the lesser of the amount owed and the amount due. Continue until all balances are zero, as required by product.md Business Rule R6.
- **F6.R3** If multiple people are tied for the largest amount owed or owing, use F3 trip-addition order to choose among the tied people. Apply the same deterministic rule at every iteration.
- **F6.R4** A suggested payment is not included in F5 balances until the owner marks it as paid. The “Mark as paid” action is one explicit action and does not require a second confirmation dialog.
- **F6.R5** When a current suggestion is marked as paid, record exactly one payment with the suggested payer, recipient, and amount in integer cents. Recalculate balances and derive a fresh set of suggestions from the updated records; do not retain stale suggestions as current.
- **F6.R6** Show outstanding suggestions separately from recorded payments. Show each recorded payment’s payer, recipient, and amount, newest recorded first.
- **F6.R7** Deleting a recorded payment requires explicit confirmation. The dialog identifies the payer, recipient, and amount and warns that deletion changes balances. Canceling or dismissing confirmation makes no changes.
- **F6.R8** On confirmation, delete the payment record and recalculate balances and suggestions. This removes the payment’s effect from the app’s ledger but does not reverse the external transfer.
- **F6.R9** Payment amounts are positive whole cents. Each payment is between two different people in the same trip, and the payer and recipient are the people in the current suggestion being marked paid.
- **F6.R10** After all current suggestions have been marked paid, every person’s balance must be exactly zero, as required by the F6 done condition and product.md Business Rule R5.
- **F6.R11** A payment record is not editable. Correcting a recorded payment means deleting it and, if appropriate, marking a newly calculated suggestion as paid.

## Data & contract
- Suggested payments are derived from current balances; they are not recorded payments until marked paid.
- A recorded payment needs a stable identifier, parent trip reference, payer person reference, recipient person reference, positive integer amount in cents, and a UTC recorded/paid timestamp.
- Marking a suggestion records the payment once. The server must validate that the payer, recipient, amount, and trip still match a current suggestion before recording it, so a stale view cannot record an obsolete suggestion.
- Derive balances and suggestions from current expenses and recorded payments. Deleting a payment removes only that payment record; it does not create or alter expenses.
- Use integer cents and parameterized MySQL queries through `mysql2`. A payment operation that changes multiple records must be performed in one transaction, consistent with project-rules.md and CLAUDE.md.
- Endpoint names, response shapes, error formats, and the maximum supported integer-cent amount belong in the API contract and data model once established.

## States & errors
- **Outstanding suggestions:** show each current suggested payer, recipient, and amount with a “Mark as paid” action.
- **All settled:** when all balances are zero, show no suggestions and indicate that the trip is settled.
- **Recorded payments:** show payer, recipient, and amount separately from outstanding suggestions, newest first.
- **Mark as paid:** after success, move the payment to recorded payments and refresh balances and suggestions. If the request fails or the suggestion is no longer current, do not show it as paid; refresh from current trip data and report the failure.
- **Delete confirmation:** identify the payment and warn that deletion changes balances; provide explicit confirm and cancel actions.
- **Canceled deletion:** close the dialog and leave the payment unchanged.
- **Delete failure:** keep the payment visible and report that it was not deleted.
- **Unauthenticated or not-owned trip request:** follow the unauthorized/not-found behavior specified by F1; do not return or change another user’s trip data.

## Accessibility
- Label each suggestion with its payer, recipient, and amount, and make the “Mark as paid” action keyboard operable.
- Distinguish outstanding suggestions from recorded payments with headings and semantic grouping, not color alone.
- The delete confirmation dialog must have an accessible name, identify the payment, move focus into the dialog when opened, keep keyboard focus within it while open, and return focus to its trigger when canceled or closed.
- Announce payment status changes, balance/suggestion updates, and settled or empty states to assistive technology.

## Edge cases
- If all balances are zero, there are no outstanding suggestions.
- Tied largest debtor or creditor choices are resolved by trip-addition order, so the algorithm is deterministic.
- After marking one suggestion paid, recalculate before showing or marking the next suggestion. In product.md Worked example 3, recording “Jo pays Sam 40.00” leaves “Alex pays Sam 10.00.”
- A stale suggestion must not create a payment if expense or payment changes have changed the current balances.
- Deleting a recorded payment removes its ledger effect, may make the recorded payer owe more or the recipient be owed more, and can generate new suggestions.
- A person referenced by a recorded payment cannot be removed under product.md Business Rule R7 until that payment record is deleted.
- Deleting a payment in the app does not undo the external transfer; the confirmation must make the ledger/balance effect clear.
- Marking a payment changes one person’s balance by the same number of cents that it changes the other person’s balance in the opposite direction, preserving the zero-sum invariant.

## Acceptance criteria
- **F6.AC1** Given a trip has positive and negative balances, when suggestions are calculated, then the person who owes the most is paired with the person owed the most and the suggested amount is the lesser of those absolute balances, repeated until all balances are zero.
- **F6.AC2** Given multiple people are tied for the largest debt or credit, when the next suggestion is chosen, then trip-addition order deterministically resolves the tie.
- **F6.AC3** Given product.md Worked example 1’s trip, when suggestions are shown, then they include “Jo pays Sam 40.00” and “Alex pays Sam 10.00.”
- **F6.AC4** Given a current suggestion exists, when the owner marks it as paid after making the external transfer, then exactly one payment is recorded and balances and outstanding suggestions are recalculated without a second confirmation dialog.
- **F6.AC5** Given product.md Worked example 1 and suggested “Jo pays Sam 40.00,” when the owner marks it as paid, then the only remaining suggestion is “Alex pays Sam 10.00,” as in Worked example 3.
- **F6.AC6** Given outstanding suggestions and recorded payments exist, when the settle-up view is shown, then they appear in separate sections and recorded payments are ordered newest first.
- **F6.AC7** Given the owner starts deleting a recorded payment, when they cancel or dismiss confirmation, then the payment and balances remain unchanged; when they confirm, then the record is deleted and balances and suggestions reflect its removal.
- **F6.AC8** Given all current suggestions have been marked paid, when balances are recalculated, then every person’s balance is exactly zero.
- **F6.AC9** Given a request to read, mark, or delete payments is unauthenticated or targets a trip not owned by the requester, when processed, then it is rejected according to F1 and no unauthorized trip or payment data is returned or changed.

## Dependencies
- F1 provides authentication and trip-ownership enforcement.
- F2 provides the parent trip.
- F3 provides people and their stable addition order for tie-breaking; payment records also prevent removal under Business Rule R7.
- F4 provides expenses and splits that contribute to F5 balances.
- F5 provides the current balances used to derive suggestions and includes recorded payments in each person’s net balance.
- No external payment service is required; money moves outside the app.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F6.AC1–F6.AC2:** Unit-test the greedy suggestion algorithm, payment amount selection, termination at zero, and deterministic tie-breaking using integer cents.
- **F6.AC3–F6.AC5:** Unit-test the product.md Worked examples 1 and 3, one-record mark-paid behavior, and regeneration of suggestions after the payment changes balances.
- **F6.AC6:** Unit-test separation of suggestion and recorded-payment sections and newest-first payment ordering.
- **F6.AC7:** Unit-test confirmation/cancel outcomes, payment deletion, and recalculation from current records.
- **F6.AC8:** Unit-test that marking every current suggestion paid leaves each balance at zero and the aggregate remains zero.
- **F6.AC9:** Unit-test authorization decisions and verify denied requests return no payment data and perform no writes; code review ownership predicates on every payment endpoint as required by project-rules.md.

## Open questions
- [ ] What API endpoint paths, request/response shapes, and error status/body conventions fit the selected app stack?
- [ ] What maximum MySQL integer-cent amount and payment timestamp field representation will the data model support?

## Decisions log
- 2026-11-17: User chose trip-addition order to resolve tied debtors/creditors, a single explicit “Mark as paid” action with no second dialog, confirmation before deleting a recorded payment, separate sections for outstanding suggestions and recorded payments, newest-first order for recorded payments, and a deletion warning that identifies payer, recipient, amount, and balance impact.
- 2026-11-17: User explicitly approved this specification.
