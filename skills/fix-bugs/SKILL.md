---
name: fix-bugs
description: Fixes bugs in a project.
purpose: "Maintenance for repositories."
version: 1.1.0
---

# Skill fix-bugs

## Goal

Find bugs in a source-code repository and, for each one, take exactly one of two paths:

- **Fix it properly** — the root cause is understood and can be removed cleanly, without workarounds
  and without creating technical debt.
- **Document it only** — a clean fix is not possible right now (or not possible without decisions
  that are not yours to make). Then the bug is reported and *nothing is changed*.

There is no third path. A partial fix, a symptom patch, a `try`/`catch` around the symptom, a
special case for the failing input, a disabled test or an added `TODO` in place of a fix are all
violations of this skill. If in doubt: document, do not touch.

## When to use

- The user asks to find and fix bugs in a repository or in a part of it.
- A known defect exists and should be repaired at its root cause.
- Not for: security vulnerabilities and CVEs (use the security-related skills), lint/style issues
  (use `project-linting`), missing tests (use `write-tests`), broad technical debt
  (use `repository-debt-analysis`).

## Definition of a bug

A bug is a **provable deviation between the intended behaviour and the actual behaviour**. The
intent must come from a source, not from your taste: specification, documentation, doc comments,
tests, type/API contracts, an invariant the surrounding code relies on, or unambiguous domain logic.

The following count as bugs as well, even when the program itself runs without an error:

- **Broken references inside the repository.** A file references another file of the same repository
  and that target does not exist. This covers all kinds of files, not only source code: relative
  links in Markdown and other documentation, paths in configuration, build, CI and container files,
  script and tooling paths, includes/imports of repository-local files, and referenced images or
  other assets. The reference is the intent; a missing target is a provable deviation from it.
  References to targets *outside* the repository (external URLs, files provided by the environment,
  files generated during the build) are not part of this — treat them as out of scope unless the
  repository itself claims they are checked in.
- **Divergence between code and documentation.** The documented behaviour and the implemented
  behaviour disagree. "Documentation" here means both doc comments (for example XML doc comments,
  docstrings, Javadoc) and documentation as text files (`README`, reference documentation, manuals,
  specifications inside the repository). Typical cases: a documented parameter, return value,
  exception, default value, unit or range that the code does not implement; a documented option,
  command, environment variable or configuration key that does not exist (or exists under another
  name); a doc comment that still describes an earlier version of the function; documented behaviour
  for an edge case that the code handles differently.

Not a bug, and therefore out of scope: code you find ugly, a style you would have written
differently, a missing feature, a deliberate documented trade-off, or a hypothetical problem you
cannot show a concrete failing path for. Documentation that is merely *incomplete* — it says nothing
about a behaviour — is not a divergence and belongs to `write-documentation`; only a documented
statement that contradicts the code is a bug here.

## Workflow

### 1. Establish the scope

Determine what is actually the repository's own maintained code. Exclude — and say that you
excluded — submodules, generated code, vendored third-party code and ignored/binary files. A defect
in generated code belongs to its generator; report it there.

Get the build and test commands of the repository working *before* changing anything, so you have a
baseline. If the repository follows the "common project structure", use the
`work-with-common-project-structure` skill for the correct commands.

### 2. Find candidate bugs

Look where bugs actually live:

- Failing or skipped/ignored tests, and tests whose assertions do not match their name.
- Existing warnings in the build output.
- Error handling: swallowed exceptions, catch-all blocks, error paths that cannot be reached.
- Boundaries: null/empty/`0`/negative values, off-by-one, first/last element, empty collections,
  overflow, division, encoding, time zones, culture-dependent parsing and formatting.
- Resource and lifetime handling: undisposed resources, use after dispose, leaks.
- Concurrency: shared mutable state, missing awaits, blocking calls in async paths, races.
- State machines and invariants: is every reachable state actually handled?
- Copy-paste sites that drifted apart — the same rule implemented twice with different behaviour.
- Repository-local references: collect the referenced paths (links in Markdown, paths in
  configuration/build/CI files, script paths, repository-local includes, referenced assets) and
  check for each whether the target exists — including case, which matters on Linux even when the
  repository is developed on Windows. Resolve relative paths against the referencing file.
- Documentation against code: read the doc comment of a function together with its implementation,
  and the documented options/commands/configuration keys of the reference documentation against
  what the code actually offers. Places where code changed recently but the surrounding
  documentation did not are the most productive spots.
- `git log --grep='fix\|bug\|hotfix\|workaround' -i` — where fixes cluster, defects cluster.

### 3. Verify each candidate before doing anything

For every candidate, produce **evidence**:

- The exact location (file and line).
- The concrete input or state that triggers it.
- The resulting wrong behaviour, and the source that says it is wrong.

Then check that it is reachable at all. If you cannot show a concrete failing path, it is a
suspicion, not a finding — list it as such, and never fix a suspicion.

For a broken reference the evidence is: the referencing file and line, the resolved target path, and
the proof that this target does not exist in the repository — plus the check that it is really meant
to be a repository-local file and not something external or generated.

For a code/documentation divergence the evidence is: the documenting location (file and line of the
doc comment or documentation file), the implementing location, and the exact statement that
contradicts the implementation. Additionally determine which of the two sides is wrong — that is
part of the finding, not of the fix.

### 4. Find the root cause

Follow the defect to where the wrong behaviour *originates*, not to where it becomes visible.
Ask what has to be true for this bug to be impossible, and check whether a fix at that point would
also remove sibling bugs. If the same defect appears in several places, treat the shared cause as
the finding.

### 5. Decide: fix or document

Fix it properly only if **all** of the following hold:

- The root cause is identified and understood — not guessed.
- It can be removed within the existing design, without a workaround and without leaving debt.
- The correct intended behaviour is unambiguous.
- The blast radius is assessable, and a regression would be caught (existing test, or a test you add).
- The change stays inside the scope the user asked for.

Document it only if **any** of the following holds:

- The clean fix requires an architectural change, an API break or a redesign.
- The intended behaviour is ambiguous and needs a decision by the user or a domain owner.
- The fix belongs in another repository (then say which one and what has to change there — do not
  work around it locally).
- You cannot verify the fix, because there is no way to build or test the affected code.
- The only fix you can see would be a workaround.

Where the decision is not obvious, ask the user, and prefer documenting over guessing.

### 6. Implement the fixes

Per bug, one focused change:

- Fix the cause, then remove the symptom-level scaffolding that existed for it (dead guards, stale
  comments) if it is now provably unnecessary.
- Add or adjust a test that fails before the fix and passes after it. If the repository has no test
  setup for that code, say so instead of silently skipping the test.
- Keep the style, naming and idioms of the surrounding code.
- For a broken reference: fix it at its cause. If the target was moved or renamed, point the
  reference at the actual target. If the reference belongs to something that no longer exists,
  remove the reference together with the statement that carries it — do not silently delete a link
  and leave a sentence behind that now claims something untrue. If the target is missing because a
  file was lost, say so and document it; do not invent a replacement file.
- For a code/documentation divergence: correct the side that is wrong. If the documentation
  describes the intended behaviour and the code does not implement it, the code is the bug. If the
  code is correct and the documentation is stale, the documentation is the bug and gets updated. If
  it is not decidable which side represents the intent, do not choose — document it and ask.
- Do not mix in refactorings, reformatting or unrelated improvements.
- Run the build and the tests, and compare against the baseline from step 1. If the fix breaks
  something you cannot resolve cleanly, revert it and move the bug to the documented list.

Keep fixes separable — one bug per commit — so a single fix can be reverted on its own.

### 7. Report

Deliver one report with two clearly separated parts:

**Fixed** — per bug: location, what was wrong, the root cause, what was changed and why this is the
cause and not the symptom, the test that covers it, and the build/test result.

**Documented, not fixed** — per bug: location, evidence (trigger, wrong behaviour, source of the
intent), root cause as far as it is known, why it was not fixed here, what a clean fix would
require, severity, and confidence (confirmed / likely / suspicion).

Also state explicitly what was examined and found healthy, and report the state of the build and
tests honestly — if something fails or was skipped, say so with the output.

## Anti-patterns

- Catching or suppressing an exception so the symptom disappears.
- Special-casing the failing input instead of correcting the logic.
- Adding a retry, a sleep, a flag or a fallback to hide a race or a broken contract.
- Disabling, skipping, weakening or rewriting a test so it passes.
- Suppressing a warning instead of fixing what it points at.
- Deleting a broken reference (or the documentation containing it) instead of pointing it at the
  target that was actually meant.
- Creating an empty placeholder file so that a broken reference resolves.
- Adjusting the documentation to the code although the code is what deviates from the intent — or
  the other way round — without having established which side is right.
- Leaving a `TODO`/`FIXME` as a substitute for either a fix or a report entry.
- Working around a defect that is caused by another repository.
- Bundling opportunistic refactorings into a bug fix.
- Fixing many things at once so that no single change can be reverted.
- Reporting a suspicion as a confirmed bug.

## Checklist for the agent

- Scope stated, exclusions named, build/test baseline established?
- Every finding backed by location, trigger and the source of the intended behaviour?
- Repository-local references checked for existing targets (including case), external and generated
  targets excluded?
- Doc comments and documentation files compared against the implementation, and for every divergence
  stated which of the two sides is wrong?
- Root cause identified for each fix — not only the symptom location?
- Every fix free of workarounds and free of new technical debt?
- Every fix covered by a test that failed before?
- Build and tests run, result reported honestly?
- Everything not cleanly fixable documented instead of patched?
