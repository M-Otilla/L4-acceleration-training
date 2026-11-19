# Trip Expense Tracker — Claude Code Guidance

## Read before making changes

- Read `docs/specs/product.md` and `docs/rules/project-rules.md` before implementing features.
- Read the relevant feature spec in `docs/specs/features/` and its plan in `docs/plans/` before changing application code.
- The product spec and project rules are the source of truth. Do not add behavior from the “After the core” or “Out of scope” sections without approval.
- Keep `docs/specs/api.md` and `docs/specs/data-model.md` current when API or data contracts are introduced.

## Stack

- Frontend: Vue 3, Vite, and JavaScript.
- Backend: Node.js, Express, and JavaScript.
- Database: MySQL, accessed from the API with `mysql2`.
- Package manager: npm.
- Unit tests: Vitest for both the frontend and API.

Use JavaScript, not TypeScript, unless the user changes this stack decision.

## Repository layout

Keep application code separated by runtime:

```text
app/
  api/          Express API and API unit tests
  web/          Vue application and frontend unit tests
docs/
  specs/
    features/   Feature specifications
    product.md
    api.md      API contract, once established
    data-model.md
  plans/        Feature implementation plans
  rules/
    project-rules.md
```

Keep each app's npm commands in its own package manifest. Run commands from the corresponding app directory.

## Development and test commands

The app directories and package manifests do not exist yet. When they are scaffolded, provide these scripts:

Frontend, from `app/web/`:

```sh
npm run dev
npm test
npm run build
```

Backend, from `app/api/`:

```sh
npm run dev
npm test
```

`npm test` must run Vitest in non-watch mode. These are expected commands, not currently runnable commands; update this section if the scripts are named differently when the apps are created.

## Implementation rules

- Build only behavior required by the product and approved feature scope. Do not edit trips, people, expenses, or payments; the product says mistakes are corrected by deleting and adding again.
- Use Express routes for API behavior and Vue components for UI behavior. Keep the API responsible for validating requests and enforcing authorization; never trust a user or trip owner identifier supplied by the client.
- Every trip-related API action must authenticate the user and verify trip ownership before returning or changing trip data.
- Access MySQL through `mysql2` using parameterized queries. Never interpolate user input into SQL.
- Load database credentials and other secrets from environment configuration. Never commit secrets or include them in logs, prompts, tests, or screenshots.
- Use integer cents for money, never floating-point amounts. Store times in UTC.
- Perform multi-record writes in a single database transaction. Deleting a trip must follow product Business Rule R8 and remove everything in the trip after confirmation.
- Use the project's logger when one is established. Do not leave `console.log`, `console.error`, or other print statements in application code. Never log passwords, tokens, keys, or personal data; do not swallow errors.
- Keep UI labels, validation feedback, focus behavior, and error states accessible by keyboard and assistive technology.

## Testing and review

- Every code change includes focused unit tests. Use Vitest for both apps; API tests must use fakes and must not require a live MySQL server, network, or running app.
- Name each test with the full acceptance-criterion ID it covers, or include that ID in a comment.
- Do not add integration or end-to-end tests unless the user changes the project rules.
- Use code review for ownership enforcement in database queries, database constraints, and cascading deletes, as specified in `docs/rules/project-rules.md`.
- Never delete, skip, or weaken an existing test to make a run pass.
- Run the smallest relevant test command after a code change and report failures explicitly.
