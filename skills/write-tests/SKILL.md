---
name: write-tests
description: Ensures that there are tests for the most important functions of the project, by finding the untested code that actually carries risk, writing independent and atomic tests for it (including defined crash-behavior and abuse cases), and verifying that the new tests run green in the repository's own test setup. Use when the user asks to write, add or complete unit-tests, or when another skill requires test coverage.
purpose: "Development for repositories."
version: 1.1.0
---

# Write Tests

Adds unit-tests for the most important non-trivial functions that do not have tests yet.

The goal is **not** a coverage number. The goal is that the next change to this repository breaks a
test instead of breaking production. Every test written must be able to answer: *which realistic
defect does this test catch?* If there is no answer, the test is not worth its maintenance cost.

## Completion criterion

Done when:

1. The selected functions have tests that would actually fail if the behavior regressed.
2. All tests — the new ones **and** the pre-existing ones — run green via the repository's own test
   command (Step 6).
3. Nothing in the production code was changed to make a test pass (see "Non-goals"), and any bug
   found on the way was reported to the user rather than silently encoded as expected behavior.

## Step 1 — Learn the repository's test setup before writing anything

Never invent a test style. Find the existing one:

- Which test framework, assertion library and mocking library are already used? Use exactly those.
- Where do test projects/files live, and what is the naming convention (`*Tests`, `test_*.py`,
  `*.spec.ts`)? Put new tests exactly where the existing ones live.
- How is a test named in this repository (`MethodName_Scenario_ExpectedResult` or otherwise)?
  Follow it.
- What test utilities already exist (builders, fixtures, fakes, base classes, test containers)?
  Reuse them instead of writing new ad-hoc setup. Repositories following the common project
  structure often have a dedicated `TestUtilities` area — check it first.
- **If the repository defines rules or scripts for tests, follow them.** For repositories using the
  common project structure, the test run is part of `scbuildcodeunit` — see the
  `work-with-common-project-structure` skill.

A test that is technically correct but stylistically foreign to the repository is a maintenance
burden, no matter how good it is.

## Step 2 — Find what is untested *and* worth testing

Two filters, applied in this order:

**Filter 1 — is it already covered?** Search the existing tests for the function, and for its
callers. Do not add a second test for behavior that is already asserted somewhere else; extend the
existing test instead if a case is missing.

**Filter 2 — does it carry risk?** Prioritize by what actually hurts when it breaks:

1. Code that handles untrusted input (parsers, deserializers, request handlers, file/path handling,
   anything reachable from outside).
2. Security-relevant logic: authentication, authorization, permission checks, token/credential
   handling, cryptographic operations, sanitization/escaping.
3. Correctness-critical domain logic: money, quantities, dates/time zones, state machines,
   permissions, business rules with several branches.
4. Complex or branch-heavy code, and code with a history of bug-fix commits.
5. Public API surface — anything other code depends on and that must not silently change.

Explicitly **skip** trivial code: plain getters/setters, pure delegations, generated code, DTOs
without logic, and thin wrappers whose only behavior comes from a library. Testing them adds cost
and catches nothing.

State briefly which functions were selected and why, before writing the tests.

## Step 3 — Determine the expected behavior from the contract, not from the implementation

This is the step that decides whether the tests are worth anything.

Derive the expected result from the **specification**: doc-comments, the function's name and
contract, the domain rules, the issue or commit that introduced it, the callers' expectations. Do
**not** derive it by running the code and asserting whatever came out — that only cements the
current implementation, bugs included, and produces tests that break on every refactoring while
catching no defects.

If implementation and apparent intent disagree, you have found a bug. Do not write a test that
asserts the buggy behavior. Report it to the user and let them decide, and either leave that case
out or write the test for the correct behavior only if the user confirms the fix.

## Step 4 — Write tests that are independent and atomic

Hard rules:

- **Independent**: no test may depend on another test having run before it, and the tests must pass
  in any order and when run individually.
- **Atomic**: one test verifies one behavior. Multiple assertions are fine when they describe the
  same behavior; a test that walks through five scenarios is five tests.
- **Deterministic**: no dependence on wall-clock time, time zone, locale, random values, network,
  machine-specific paths, environment variables or leftover state from an earlier run. Inject or
  fake these. A flaky test is worse than no test, because it trains everyone to ignore red.
- **Self-contained state**: create what the test needs, and clean it up. Never rely on a shared
  mutable fixture that other tests also write to. Any test touching a database, file system or
  container must leave the environment as it found it.
- **Clear structure**: arrange / act / assert, with a name that states scenario and expectation.
  The failure message must make the cause obvious without opening a debugger.
- **Assert the meaningful thing**: the observable behavior and the contract — not internal call
  order or private state, unless the interaction *is* the contract.
- **Mock only what you must**: external systems, non-determinism and slow dependencies. Do not mock
  the code under test's own collaborators when the real ones are cheap and deterministic — over-
  mocked tests assert that the code calls itself the way it currently does, and nothing more.

Cover per selected function: the normal case, the meaningful boundary values (empty, zero, one,
maximum, off-by-one edges), and the invalid/hostile inputs.

## Step 5 — Cover failure behavior and abuse cases explicitly

Two categories that are routinely forgotten and are explicitly in this skill's scope:

**Defined crash-behavior.** If the code under test can throw, there must be a test for it. It must
verify that:

- the failure is the *defined* one (the documented exception type / error result), not an
  incidental `NullReferenceException` from three layers down,
- the failure happens at the right boundary (invalid input is rejected early, not half-processed),
- the error message and any exposed detail **reveal nothing sensitive** — no credentials, tokens,
  connection strings, internal paths, stack traces exposed to an untrusted caller, no data of other
  users,
- the system is left in a consistent state afterwards (no half-written file, no leaked resource, no
  partially applied transaction).

**Abuse cases.** For anything reachable by an attacker, write tests that assert an attacker *cannot*
succeed — as positive tests of the guard, not as an afterthought:

- authorization: a caller without the required permission is rejected (test the *denial*, not only
  the happy path with an admin),
- injection and traversal: `../` and absolute paths in file arguments, SQL/command/template
  metacharacters, oversized and deeply nested input,
- identifier tampering: accessing another tenant's/user's object by ID,
- input that should be rejected before it reaches expensive processing.

Each of these tests must fail if the guard is removed. Verify that mentally — a "security test"
that passes with the check deleted is theater.

## Step 6 — Run the tests and make sure the suite is green

- Run the repository's own test command (for the common project structure: `scbuildcodeunit`; exit
  code 0 means fine, non-zero means read the output).
- Run the **whole** suite, not just the new tests: new tests can break existing ones through shared
  state, and a pre-existing failure must be reported, not inherited silently.
- Verify a new test actually can fail — if a test passes no matter what the code does, it asserts
  nothing.
- If a new test fails because the production code is wrong: **report it, do not fix it silently**
  and do not weaken the assertion. Fixing it is the `small-fix` or `fix-error` skill's job, on the
  user's decision.

Report at the end: which functions got tests, which cases each covers, what was deliberately not
tested and why, and any bug or suspicious behavior found along the way.

## Non-goals

- Do **not** change production code to make a test pass — not even "just a small adjustment". If
  the code must change, say so and let the user decide.
- Do **not** weaken, comment out, skip or delete an existing test to get a green run. A failing
  pre-existing test is a finding to report.
- Do **not** write tests that assert the current implementation's behavior when that behavior looks
  wrong (Step 3).
- Do **not** write tests for trivial code, and do not chase a coverage percentage.
- Do **not** write tests that depend on timing, sleeps, execution order or external services.
- Do **not** introduce a new test framework, assertion style or mocking library into a repository
  that already has one.
- Do **not** commit or push unless the user explicitly asks.
- Write more tests than the criteria above select only if the user explicitly asks for it.
