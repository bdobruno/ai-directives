# Core coding directives

These apply to every task, in every language.

## Environment
- Never read `.env` files.

## Requirements → tests
- Align on the requirements with the user before implementing anything. List them, resolve ambiguities, and get confirmation.
- Every requirement in the spec has a test or a group of tests linked to it.
- Give each requirement an ID (e.g. `REQ-012`) and reference it in the test name or docstring, so the link can be found by searching for the ID.
- A requirement with no linked test is not done. A test with no linked requirement must be justified or removed.

## Test-driven development
- Red before green: write the failing test first, run it, and confirm it fails for the expected reason. Then write only enough code to make it pass. No speculative features.
- Work in vertical slices: one test → one minimal implementation → repeat. Never write all the tests up front and then all the implementation.
- Refactor as a separate step, once the tests are green.
- Every bug fix starts with a test that reproduces the bug.

## What a good test is
- Tests verify behavior through public interfaces, not implementation details. They should survive a refactor that doesn't change behavior.
- Test names describe *what* the code does ("user can checkout with valid cart"), not *how*.
- Expected values come from an independent source of truth: a known literal, a worked example, the spec. Never recompute the expected value the way the code does.
- Verify through the interface, not through a side channel (e.g. read back with `getUser`, don't query the database directly).
- Red flags: testing private methods, asserting on call counts or call order, a test that breaks on a refactor with no behavior change.

## Mocking
- Mock only at system boundaries: external APIs, time, randomness, and sometimes the database or filesystem (prefer a real test DB).
- Never mock your own modules or internal collaborators.
- Pass external dependencies in (dependency injection) rather than creating them inside the function.
- Wrap external services in specific functions per operation (`getUser`, `createOrder`), not one generic `fetch(endpoint, options)`, so each mock returns one shape.

## Small, testable functions
- One function, one responsibility. Aim for ~30 lines or fewer.
- Leaf functions are pure where possible: inputs in, output out, no I/O, no hidden state.
- Leaf functions get fast tests with no mocks.

## Orchestrators
- Each major process has exactly one orchestrator.
- An orchestrator only sequences calls to leaf functions and handles I/O, errors, and logging. It contains no business logic of its own.
- Orchestrators are covered by integration tests, mocking only at system boundaries.

## Orchestrators of orchestrators
- Larger flows compose orchestrators; they never call leaf functions directly.
- Keep the layers strict: leaf → orchestrator → orchestrator of orchestrators. A layer only calls the layer directly below it.
- The top-level entry point should read like a table of contents of the process.
