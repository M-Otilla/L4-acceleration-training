# F1 — Registration and login

_Status: Draft · Last updated: 2026-11-16 · Source: product.md — Goals, Roles, Assumptions, MVP scope_

## Summary
Users create an account with an email address and password, then log in and out. Passwords are stored only as hashes. Every trip endpoint requires an authenticated user and enforces that user’s ownership of the trip, so authentication alone never grants access to another user’s data.

## User stories
- As a visitor, I want to register with my email and password so that I can use the app.
- As a registered user, I want to log in and out so that I can control access to my account.
- As a trip owner, I want every trip request to enforce my ownership so that other users cannot see or change my trips.

## Scope
**In:** Account registration, email and password login, logout, secure password hashing, authentication on every trip endpoint, and trip ownership checks.

**Out:** Email verification, password reset, social login, multiple roles, and sharing trips with other users. Password reset and sharing are explicitly out of scope in product.md; email verification is excluded by its Assumptions section.

## User flow & screens
1. A logged-out visitor can open the registration or login screen.
2. The visitor submits an email and password to register. On success, the account is created, the user is signed in, and the app opens the trip list (provided by F2).
3. A registered user submits their credentials to log in. On success, the app opens the trip list.
4. The user logs out. The active authentication is ended and subsequent trip requests are treated as unauthenticated.
5. A logged-out request to a trip endpoint is rejected. An authenticated request for another user’s trip reveals no trip data and makes no changes.

No screen mockups or application routes exist yet; exact navigation and endpoint names are to be established with the chosen stack.

## Requirements
- **F1.R1** Registration requires a syntactically valid email address and a password of at least 12 characters. Do not impose character-class composition rules. Trim surrounding whitespace from email addresses and treat them case-insensitively for uniqueness and login.
- **F1.R2** Registering an email address that is already in use after normalization must not create another account.
- **F1.R3** A successful registration signs the new user in and takes them to the trip list.
- **F1.R4** A user can log in with the registered email and password. Email normalization is identical to registration. A failed login uses the same user-facing message whether the email is unknown or the password is incorrect: “Invalid email or password.”
- **F1.R5** Logging out ends the active authentication. Any later request to a protected trip endpoint must be rejected unless the user authenticates again.
- **F1.R6** Every trip endpoint requires an authenticated user. For a trip-specific request, the server must check that the trip belongs to that user before returning or changing trip data. Creating a trip must associate it with the authenticated user.
- **F1.R7** Passwords must be stored as password hashes, never as plaintext. Passwords, hashes, authentication tokens, and session secrets must not be logged.
- **F1.R8** An unauthenticated trip request returns an unauthorized response. A request for another user’s trip returns a not-found response and must not reveal or modify the trip. The status for an unknown trip is also not found.

## Data & contract
- An account needs a stable identifier, a normalized unique email, and a password hash. The underlying data model and exact field names are stack-specific.
- Authentication state must be established after successful registration or login and invalidated on logout. The session or token representation, persistence, and lifetime depend on the selected application stack.
- Authentication endpoint names, request/response schemas, and error response shapes belong in the API contract once the stack is selected. The externally observable outcomes and status behavior in this spec still apply.
- Password-hashing algorithm and parameters must be selected for the chosen stack using its secure, maintained password-hashing support.

## States & errors
- **Registration in progress:** prevent duplicate submissions while the request is pending.
- **Registration validation failure:** show an error associated with the invalid email or password; do not create an account.
- **Email already registered:** registration fails without creating an account. Exact wording and whether the response is generic remain open.
- **Login failure:** show the same message for an unknown email and a wrong password: “Invalid email or password.”
- **Unauthenticated trip request:** return unauthorized and do not return trip data.
- **Trip not found or not owned by the user:** return not found and do not return trip data or apply changes.
- **Logout:** clear the current authentication state; repeating logout leaves the user logged out.

## Accessibility
- Provide programmatically associated labels for email and password fields and identify required fields and validation errors to assistive technology.
- Use appropriate email and password input semantics, preserve keyboard operation, and move focus to an error summary or the first invalid field after a failed submission.
- Ensure authentication status and errors are not communicated by color alone.

## Edge cases
- Email values differing only by surrounding whitespace or letter case refer to the same account.
- A request with missing or malformed credentials fails without creating an account or authenticating the user.
- Logging out or making a trip request with expired/invalid authentication must not expose trip data. The specific expiry and renewal behavior is stack-dependent and remains open.
- A valid session for one user does not authorize access to another user’s trip, including through direct API requests.

## Acceptance criteria
- **F1.AC1** Given a visitor submits a valid unused email and a password of at least 12 characters, when registration succeeds, then exactly one account exists with a password hash rather than the submitted password, and the visitor is authenticated and taken to the trip list.
- **F1.AC2** Given a visitor submits an invalid email, a password shorter than 12 characters, or an email already in use after normalization, when registration is submitted, then no additional account is created and the request reports failure.
- **F1.AC3** Given an account exists, when its owner logs in using the registered email with surrounding whitespace or different letter case and the correct password, then login succeeds.
- **F1.AC4** Given login is attempted with an unknown email or an incorrect password, when the request fails, then both cases produce the same user-facing message and do not authenticate the requester.
- **F1.AC5** Given a user is logged in, when they log out and then request any protected trip endpoint, then the request is rejected as unauthorized and returns no trip data.
- **F1.AC6** Given user A owns a trip and user B is authenticated, when user B requests to read or change that trip through any trip endpoint, then the response is not found, no trip data is returned, and no data is changed.
- **F1.AC7** Given any trip endpoint is called without valid authentication, when the request is processed, then it is rejected before trip data is returned or changed.
- **F1.AC8** Given any registration, login, or trip request is handled, when application logging occurs, then plaintext passwords, password hashes, tokens, and session secrets are absent from logs.

## Dependencies
- F1 precedes F2–F6 because all trip data is owned by an authenticated user.
- The application stack must be selected before choosing the session mechanism, password-hashing implementation, API shapes, and data model.
- No external authentication provider is required; the product specifies email and password accounts.

## Test plan
Every test must include the full acceptance-criterion ID in its name or a comment, as required by project-rules.md.

- **F1.AC1–F1.AC2:** Unit-test registration validation, normalized uniqueness, and account creation using fakes; verify that persisted credentials are hashes and invalid registration creates no account.
- **F1.AC3–F1.AC4:** Unit-test email normalization, correct-password login, and identical user-facing failures for unknown email and incorrect password.
- **F1.AC5:** Unit-test logout/session invalidation and unauthenticated access to each protected trip endpoint.
- **F1.AC6–F1.AC7:** Code review the ownership predicate on every trip endpoint as required by project-rules.md; unit-test authorization decisions and verify that denied requests do not return data or perform writes.
- **F1.AC8:** Unit-test or inspect logging paths with representative credentials and verify secrets are redacted or excluded.

## Open questions
- [ ] Which authentication/session mechanism and expiration/renewal policy fit the application stack selected in `CLAUDE.md`?
- [ ] What are the exact API endpoints, request/response schemas, persistence fields, and password-hashing algorithm/parameters for the selected stack?
- [ ] What feedback should registration show when an email is already in use, balancing usability and account-enumeration risk?
- [ ] What maximum password length and email-validation rules beyond basic syntax should apply?

## Decisions log
- 2026-11-16: Clarification response indicated the user was unavailable and requested pragmatic defaults → selected a 12-character minimum with no composition rules, trimmed case-insensitive email uniqueness, automatic sign-in after registration, a generic login-failure message, and not-found responses for trips not owned by the requester. These are draft defaults and may be revised.
