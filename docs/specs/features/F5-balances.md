# F5 — Balances

_Status: Approved · Last updated: 2026-11-17 · Source: product.md — Goals, Roles, Assumptions, Key terms, MVP scope, Business Rules R3–R5, Worked examples; F1–F4 specs; CLAUDE.md and project-rules.md_

## Summary
Show each person’s current balance for an owned trip, including the amounts they paid, their expense shares, payments made and received, and the resulting net balance. Positive balances are shown as “is owed,” negative balances as “owes,” and zero balances as “Settled up.” Balances are calculated from current trip expenses and recorded payments in whole cents and must always sum to zero.

## User stories
- As a trip owner, I want to see what each person is owed or owes so that I can understand the trip’s current financial state.
- As a trip owner, I want to see the amounts paid, shares, and payments behind each balance so that I can understand how the net was calculated.
- As a trip owner, I want balances to update when trip expenses or payments change so that the displayed values remain accurate.

## Scope
**In:** Calculate and display each person’s balance from expenses and payments; show component totals and the net status; preserve trip-addition order; and update balances when source records change. Enforce the zero-sum invariant in product.md Business Rule R5.

**Out:** Suggested payments and marking them paid (F6), editing balances, currency selection or conversion, and manual balance adjustments. Product.md defines balances as derived from expenses and payments and places suggested payments in F6.

## User flow & screens
1. The authenticated owner opens an owned trip’s balance view.
2. The app calculates and displays one row for every person on the trip, in the order they were added.
3. Each row shows amounts paid, expense shares, payments made, payments received, and the net balance. A positive net is labeled “is owed”; a negative net is labeled “owes”; a zero net is labeled “Settled up.”
4. When a person, expense, or recorded payment is added or deleted, refresh the balance view from the current trip records.
5. If the trip has no people, show an empty state and no balance rows.

No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F5.R1** Reading balances requires authentication and ownership of the parent trip, following F1. Return no balance data for an unauthenticated user or a user who does not own the trip.
- **F5.R2** Calculate each person’s balance in integer cents using product.md Business Rule R4: total amounts they paid minus their total expense shares, plus payments they made minus payments they received.
- **F5.R3** Include only recorded payments in the calculation. A suggested payment affects a balance only after it has been marked as paid, as defined by product.md Key terms and F6.
- **F5.R4** Use each expense’s cent-accurate shares from F4, including the leftover-cent allocation in F3 trip-addition order required by product.md Business Rule R3.
- **F5.R5** Show every person on the trip, including people with a zero balance, in their stable trip-addition order from F3.
- **F5.R6** Display the paid, shares, payments made, payments received, and net balance components for each person. Format amounts to two decimal places without applying currency conversion or a per-trip currency setting.
- **F5.R7** Label a positive net balance as “is owed,” a negative net balance as “owes,” and an exact zero balance as “Settled up” with a displayed value of 0.00.
- **F5.R8** Recalculate balances from the current expense and payment records whenever those records or the trip’s people change. Do not retain stale balances after an add or delete operation.
- **F5.R9** The sum of all people’s balances for a trip must equal exactly zero cents, as required by product.md Business Rule R5.
- **F5.R10** Do not allow a user to edit or manually adjust a calculated balance. Correct the underlying trip records through the add/delete behaviors in F2–F4 and F6.

## Data & contract
- Balances are derived values; the source data is the trip’s people, expenses and shares, and recorded payments. Do not treat a suggested but unpaid payment as a recorded payment.
- Calculate and return monetary components as integer cents. The API response shape and whether calculation is performed in SQL or application code belong in the API/data-model documentation once established.
- Display conversion from cents to a decimal amount must preserve the exact cent value and show two decimal places. Use no floating-point arithmetic for money.
- F4 expense deletion and F6 payment changes alter the balance inputs. They do not create a separate editable balance record.
- Balance endpoint names, response fields, and error formats are stack-specific. F1 authentication and ownership outcomes apply.

## States & errors
- **Loading balances:** indicate that the view is loading.
- **No people:** show an empty state with no balance rows.
- **Positive balance:** show the person as “is owed” the positive amount.
- **Negative balance:** show the person as “owes” the absolute value of the negative amount.
- **Zero balance:** show “Settled up” and 0.00.
- **Source-record update:** refresh all affected balances from current records after a person, expense, or recorded payment is added or deleted.
- **Calculation or request failure:** report the failure and do not present stale values as current.
- **Unauthenticated or not-owned trip request:** follow the unauthorized/not-found behavior specified by F1; do not return another user’s balances or trip data.

## Accessibility
- Provide clear labels for each balance component and the net status so the breakdown is understandable when read by assistive technology.
- Do not communicate positive, negative, or zero status by color alone; include the “is owed,” “owes,” or “Settled up” text.
- Announce balance updates and loading or empty states to assistive technology.
- Ensure the balance view and its controls are keyboard operable.

## Edge cases
- Include a person with no expenses or payments and show a zero balance.
- Include a person who paid for an expense but was not a participant; their payment increases their balance without adding an expense share, as permitted by Business Rule R2.
- A person’s unpaid suggested payment does not change their balance; once marked paid in F6, the resulting payment is included.
- A deleted expense no longer contributes to paid or share totals; separately recorded payments remain part of the formula.
- Leftover cents assigned by F4 must keep all expense shares equal to the full expense amount, so they cannot make the trip’s total balances differ from zero.
- A trip with no people has no balance rows and has a total balance of zero.

## Acceptance criteria
- **F5.AC1** Given an authenticated owner opens a trip with people, expenses, and recorded payments, when balances are calculated, then each person’s net equals amounts paid minus shares plus payments made minus payments received, all in integer cents.
- **F5.AC2** Given a trip’s expenses include an equal split with leftover cents, when balances are calculated, then each person’s shares use F4’s cent-accurate allocation in F3 trip-addition order and sum to the exact expense amount.
- **F5.AC3** Given a person has no payments or expense activity, when balances are shown, then that person still appears in trip-addition order with “Settled up” and 0.00.
- **F5.AC4** Given a person has a positive balance, a negative balance, or a zero balance, when the balance view is shown, then the status reads “is owed,” “owes,” or “Settled up” respectively and the displayed amount has two decimal places.
- **F5.AC5** Given a suggested payment has not been marked paid, when balances are calculated, then it is excluded; after it is marked paid in F6, the corresponding payments made and received are included.
- **F5.AC6** Given an expense, payment, or person is added or deleted, when the balance view refreshes, then it reflects the current source records and does not show stale results.
- **F5.AC7** Given any trip has people, when all their balances are calculated, then the sum is exactly zero cents; when the trip has no people, then the empty balance set also sums to zero.
- **F5.AC8** Given a request to read balances is unauthenticated or targets a trip not owned by the requester, when processed, then it is rejected according to F1 and no balance or trip data is returned.

## Dependencies
- F1 provides authentication and trip-ownership enforcement.
- F2 provides the parent trip.
- F3 provides people and their stable addition order.
- F4 provides expense totals and cent-accurate shares.
- F6 provides recorded payments and suggested-payment completion behavior.
- No external service is required.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F5.AC1–F5.AC2:** Unit-test the balance formula and cent-accurate expense-share inputs using fakes; include product.md Worked examples 1 and 2.
- **F5.AC3–F5.AC4:** Unit-test zero, positive, and negative labels, two-decimal formatting, and trip-addition ordering.
- **F5.AC5–F5.AC6:** Unit-test excluding unpaid suggestions, including marked-paid transactions, and refreshing results after source-record changes.
- **F5.AC7:** Unit-test the zero-sum invariant for empty, ordinary, and rounding-remainder trip data.
- **F5.AC8:** Unit-test authorization decisions and verify denied requests return no balance data and perform no writes; code review ownership predicates as required by project-rules.md.

## Open questions
- [ ] What API endpoint path, response shape, and error status/body conventions fit the selected app stack?

## Decisions log
- 2026-11-17: User chose “Settled up” with 0.00 for zero balances, a full paid/share/payment breakdown, trip-addition order for the list, and immediate recalculation when trip records change.
- 2026-11-17: User explicitly approved this specification.
