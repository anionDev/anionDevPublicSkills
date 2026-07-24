---
name: write-documentation
description: Writes and updates the non-business-logic documentation of a repository (readme, doc-comments for non-trivial functions, reference-folder documentation), verifies that the documented instructions actually work, and removes documentation that has become wrong. Use when the user asks to write, complete, update or check the documentation of a repository.
purpose: "Development for repositories."
version: 1.1.0
---

# Write Documentation

Ensures that the documentation of the repository is complete, correct and up to date.

The measure of documentation quality is not its length. It is whether a competent person who does
not know this repository can, using only the documentation: understand what the repository is for,
get it running, change something safely, and contribute in the expected way.

Two rules override everything else:

1. **Wrong documentation is worse than none.** A README command that no longer works costs a reader
   more time than an empty README. Verifying existing documentation therefore comes before writing
   new documentation.
2. **Document what cannot be read off the code**: purpose, contracts, constraints, and the *why*
   behind non-obvious decisions. Never paraphrase the implementation — that kind of text is wrong
   after the first refactoring and nobody notices.

## Completion criterion

Done when:

1. Every documented instruction that can be verified has been verified (Step 2).
2. The readme, the doc-comments of the non-trivial functions and the reference-folder documentation
   are complete for what this repository actually contains (Steps 3–5).
3. Documentation found to be wrong was corrected or removed — not left standing next to the new
   text.
4. Anything that could not be documented because the intent was unclear has been reported to the
   user as an open question rather than guessed.

## Step 1 — Take stock, and follow the repository's own rules

Before writing:

- Inventory what exists: readme, reference folder, architecture/decision documents, changelog,
  contribution guide, doc-comments, generated API documentation, comments in configuration and
  build scripts.
- **If the repository defines rules or scripts for documentation, follow them.** For repositories
  using the common project structure, documentation lives in the defined folders and the
  documentation build is part of `scbuildcodeunit` — see the `work-with-common-project-structure`
  skill.
- Detect and keep the existing conventions: language (do not mix languages — match what the
  repository already uses), doc-comment style (XML-doc, JSDoc, docstrings), heading structure,
  terminology. If the repository maintains a domain model or glossary, use exactly its terms
  (see the `domain-modeling` skill).
- Determine the audience per document: the readme is for users and newcomers, the reference folder
  is for maintainers and operators, doc-comments are for callers. Do not merge these.
- Note what is **generated** (API docs from doc-comments, tables from configuration): fix the
  source, never the generated artifact.

## Step 2 — Verify the existing documentation before adding to it

Go through the existing documentation and check every verifiable claim against the repository:

- Do the setup/build/run/test commands still exist and still work? Actually run them where that is
  safe and cheap; where it is not, at least confirm the referenced scripts, targets and files exist.
- Do referenced paths, file names, modules, classes, endpoints, configuration keys and environment
  variables still exist under those names?
- Are the stated prerequisites (runtime versions, tools, services) still the ones the build
  actually requires?
- Do internal links and external URLs still resolve?
- Do documented defaults match the real defaults in the code/configuration?
- Does the described architecture still match the module layout?

Every mismatch is a finding: correct it. If the correct answer cannot be determined, mark the spot
and ask the user — do not invent a plausible-sounding replacement.

Delete documentation that describes things which no longer exist. Removing a dead section is a
documentation improvement, not a loss.

## Step 3 — The readme

The readme must let a newcomer get from zero to a running state. Cover, in this order and only as
far as it applies to this repository:

- **What this is and what it is for** — in the first few lines, in plain language, including what it
  is explicitly *not* for if that is a likely misunderstanding.
- **Status and audience** — is this a library, a service, a CLI, a template? Who is it for?
- **Prerequisites** — runtimes, tools, services, accounts, with the versions that are actually
  required.
- **Installation / getting started** — the shortest path to a working state, as commands that can be
  copied and that were verified in Step 2.
- **Usage** — the two or three realistic entry-level examples, not an exhaustive API dump. Examples
  must be runnable as written.
- **Configuration** — the relevant options, their defaults and their effect; secrets handling
  described without ever containing a real secret.
- **Building, testing and releasing** — the repository's own commands (for the common project
  structure: `scbuildcodeunit`).
- **Project structure** — a short map of the top-level directories, if the layout is not obvious.
- **Contributing** — branch/commit conventions, review expectations, what must pass before a change
  is accepted. May be a link to a separate file.
- **License and contact/support**.

Keep it scannable: short sections, working links, no wall of text. Detail belongs in the reference
folder, linked from the readme — the readme's job is orientation, not completeness.

## Step 4 — Doc-comments in the source code

Write doc-comments for the **non-trivial** functions and types, and there specifically about
behavior and design decisions:

- **The contract**: what it does (in terms of the caller's world, not the implementation's),
  parameter meanings and valid ranges, what it returns, what it does *not* do.
- **Failure behavior**: which exceptions/error results occur under which conditions — this is the
  part callers most often need and most rarely find.
- **Constraints and guarantees**: thread-safety, ordering, idempotency, side effects, resource
  ownership (who disposes what), performance characteristics where they matter, units and time
  zones for numeric and temporal values.
- **Design decisions and their reasons**: why this approach, which alternative was rejected and
  why, which external constraint forced it. This is the highest-value comment in any codebase,
  because it is the only information that is truly unrecoverable from the code.
- **Non-obvious preconditions** and anything a caller could plausibly get wrong.

Do **not**:

- restate the signature in prose (`/// Gets the name. Returns the name.`),
- document trivial members, generated code or self-evident private helpers,
- write comments that will silently rot (line-by-line narration of the implementation),
- leave a comment standing that contradicts the code — correct or delete it.

Where an existing comment is wrong, fixing it takes priority over adding new ones.

## Step 5 — The reference folder

The reference documentation is for maintainers and operators — everything too detailed for the
readme:

- Architecture overview: components, responsibilities, dependency directions, and the trust/process
  boundaries; a diagram if it helps, but a correct paragraph beats a stale diagram.
- Design decisions / ADRs: decision, context, alternatives, consequences, date. Never rewrite the
  history of a decision — supersede it with a new entry (see the `domain-modeling` skill if the
  repository keeps a domain model or glossary).
- Operations: deployment, configuration, environments, logging/monitoring, backup and restore,
  known failure modes and their handling.
- Development: how to set up a development environment, how to run the tests, how to debug the
  typical scenario, how a release is produced.
- Integration points: external services, APIs, data formats, protocols, versioning and
  compatibility guarantees.
- Glossary of the domain terms used in the code, if the repository has a non-obvious domain.

Keep each document focused on one subject, and link between them instead of duplicating text.
Duplicated documentation diverges — always.

## Step 6 — Verify and report

- Re-run the verifiable instructions after editing (Step 2 again, on the new text): every command
  in the readme must have been executed or at least confirmed to exist.
- Where the repository builds its documentation, run that build (common project structure:
  `scbuildcodeunit`) and make sure it passes.
- Check that all links, including the new ones, resolve.
- Report: what was added, what was corrected, what was deleted and why, and every open question
  where the intent could not be determined.

## Non-goals

- Do **not** invent facts. If behavior, intent or a rationale cannot be determined from the code,
  the history or the user, ask — a plausible-sounding wrong sentence in documentation is worse than
  a gap, because readers trust it.
- Do **not** document behavior that does not exist yet, and do not describe planned features as if
  they were implemented.
- Do **not** put credentials, tokens, internal URLs or other sensitive information into
  documentation or examples.
- Do **not** change source code other than its comments; if the code needs a fix, report it
  (`small-fix` / `fix-error`).
- Do **not** edit generated documentation artifacts — change their source.
- Do **not** pad documents with generic boilerplate that is not specific to this repository.
- Do **not** commit or push unless the user explicitly asks.
