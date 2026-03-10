# Test Implementation Plan

## Objective
Create meaningful automated tests for both the backend (Express API) and frontend (React + Vite) packages so that `npm run test` exercises the core boilerplate functionality and guards against regressions.

## Current State
- **Backend**: Single `src/index.ts` file starts an Express server but nothing is exported for testing. `vitest` is listed as a dev dependency, yet there are no test files. Running tests today would effectively be a no-op.
- **Frontend**: Minimal React app rendering static content. Vitest is installed but not configured (no `test` block in `vite.config.ts`, no jsdom setup, no test files).

## Proposed Work
### 1. Backend: make the Express app testable
- Split the server bootstrap logic so the Express `app` instance can be imported without opening a listening socket.
  - Create `backend/src/app.ts` exporting a `createApp()` factory (wraps middleware + routes) **and** a reusable `app` instance for the simple case.
  - Update `backend/src/index.ts` so it imports `{ app }` from `./app` and only calls `app.listen(...)`. This keeps runtime behavior unchanged but lets Vitest import the app without side effects.

### 2. Backend: add HTTP tests with Supertest
- Add `supertest` + `@types/supertest` to `devDependencies`.
- Update `backend/tsconfig.json` to include `"types": ["vitest/globals"]` (or extend existing array) so the new tests can use `describe`, `it`, etc. without additional imports.
- Create `backend/src/app.test.ts` that:
  1. Imports the exported `app` and wraps it with `supertest(app)`.
  2. Confirms `GET /health` responds with status 200 and JSON `{ status: 'ok' }`.
  3. Optionally verifies the `Content-Type` header and that unknown routes return 404 (using Express default handler) so the router contract is covered.
- Ensure tests run via `npm run test -w backend`.

### 3. Frontend: configure Vitest for React
- Install testing utilities: `@testing-library/react`, `@testing-library/jest-dom`, and (optionally) `@testing-library/user-event` for future interaction tests.
- Add `frontend/src/setupTests.ts` that imports `@testing-library/jest-dom/vitest` to extend matchers.
- Update `vite.config.ts` to include a `test` section:
  ```ts
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/setupTests.ts'
  }
  ```
- Update `frontend/tsconfig.json` to include `"types": ["vitest/globals"]` so TypeScript recognizes the global test APIs.

### 4. Frontend: author component tests
- Add `frontend/src/App.test.tsx` that renders `<App />` via Testing Library and asserts:
  - The `<h1>` contains "Habit Tracker".
  - The supporting paragraph text is present.
  - (Optional) Snapshot or semantic checks ensuring the main landmark exists, preparing the ground for future UI changes.
- If future stateful components are added, this file can grow or new test files can be created alongside features.

### 5. Repository-level validation
- Document in the README (or a new short TESTING.md) how to execute the test suites (`npm run test`, workspace-specific commands) so contributors know how to run them.
- Run `npm run test` at the root to ensure both workspaces succeed sequentially.

## Acceptance Criteria
- `npm run test` runs frontend + backend tests successfully on a clean checkout.
- Backend tests cover the `/health` endpoint and fail if the JSON contract changes.
- Frontend tests verify the key UI copy renders; failing when the copy or structure changes unexpectedly.
- No server starts during backend tests (port remains free); frontend tests run under jsdom without accessing the network.

## Risks & Mitigations
- **Express app refactor could introduce regressions**: keep runtime behavior identical by only extracting middleware/route creation into `createApp()` and reusing it in `index.ts`.
- **Vitest configuration conflicts**: ensure new `test` config in `vite.config.ts` does not interfere with existing build settings; use `defineConfig` merge pattern.
- **Dependency bloat**: limit new packages to widely used testing libraries; they remain dev-only.

## Validation Plan
1. Install new dev dependencies via `npm install`.
2. Run `npm run test -w backend` to confirm backend Vitest suite passes.
3. Run `npm run test -w frontend` to confirm frontend suite passes under jsdom.
4. Run `npm run test` at repo root (ensuring combined script works).
5. Optionally run `npm run build` to ensure the refactor didn’t break production builds.
