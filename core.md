# Core coding directives

These apply to every task, in every language.

## Environment
- Never read `.env` files.
- Don't edit files with `sed` or other shell commands. Use the inline edit tool so I can review and adjust each change.

## Requirements → tests
- Align on the requirements with the user before implementing anything. List them, resolve ambiguities, and get confirmation.
- Once the requirements are approved, write them to `spec.md`, split into vertical slices, with each slice listing its requirement IDs.
- When a slice is implemented and its tests pass, remove it from `spec.md`. The spec only holds work that is still left to do.
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

## Module architecture
- Dependency direction: app → capability → port ← adapter. Never import another module's internals; import only what a package exports.
- A capability is a package with a pure core: inputs in, a Plan or Result out. The core does no I/O and never reads the clock, environment, or disk. Pass `now` and the like in.
- The core defines its external dependencies as ports: interfaces it owns, each as narrow as that capability needs.
- An adapter implements a port for one external system. There is one adapter per external system, shared by every capability. Adapters hold no business rules.
- Translate external names and shapes in adapters only. Data crossing a module boundary has a named, typed contract (a record type), never a loose map or tuple.
- Effects follow plan → apply. The core returns a plan, a list of typed operations. A separate step applies it through the ports. Dry run, logging, and tests inspect the plan.
- Each capability has one orchestrator: read through the ports, build the plan with the core, apply it unless it's a dry run, return the plan. Test it with in-memory fakes of the ports.
- Apps (flow, CLI, server) are composition roots: read config and secrets, build the adapters, call the orchestrators. No business logic.
- Add new variants (objects, tables, sources, checks) through config, not new code paths. If adding a variant means editing code, the config is missing a field.
- CI must enforce this structure, not review. Every PR runs checks that fail the build on imports against the dependency direction, I/O imports in the core, imports between capabilities, and imports of another module's internals. Never merge on red.

### Layout
Under the language's usual source root:
```
<app entry points>   composition roots, thin
config/              variants: sources, tables, checks
<capability>/        core · ports · run (the orchestrator)
adapters/            one module per external system
tests/               mirrors the source tree; core tests are pure, no mocks
```

### Bringing an existing repo in line
- Audit the repo against the rules above and write the gaps into `spec.md` as vertical slices with requirement IDs, per Requirements → tests.
- One slice, one PR. Write the behavior that must not change as requirements too, each with a characterization test written before any code moves.
