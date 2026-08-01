---
name: ai-session-save
description: Exports the context of the current AI-session (knowledge, decisions, state, permissions, next steps) as one copy-pasteable block so that it can be imported later into a new session using the ai-session-restore-skill.
purpose: "Session-management for AI-sessions."
tags: session-management
version: 1.0.0
---

# Skill ai-session-save

## Goal

Produce **one single copy-pasteable block** which contains everything a fresh AI-session needs to
continue the current work as if it had been there from the beginning: what the task is, what was
decided and why, what is already done, what was learned about this codebase, which commands work,
which permissions and tools were used, and what the next steps are.

The block is the deliverable. It is not a summary for the user to read — it is an artifact for
another agent to consume via the `ai-session-restore`-skill. Optimize it for that.

## When to use

- The user wants to end a session (the context gets long, the day is over, the tool is restarted)
  and continue later without losing the accumulated context.
- The user wants to hand the work over to another session, another machine or another person.

## Workflow

### 1. Collect the facts of the session

Go through the whole conversation of the current session and collect:

- The task the user actually asked for, in their words, including scope-corrections made later.
- Every decision that was made, together with its reason. A decision without its reason is useless
  for the next session, because it will simply be re-litigated there.
- Everything that was learned about the codebase which is **not** obvious from reading the code:
  quirks, traps, things that look wrong but are intentional, dependencies between components.
- Every approach that was tried and rejected, and why. This keeps the next session out of the same
  dead end.
- The state of the work: what is finished and verified, what is half-done, what is untouched.
- Preferences and corrections the user gave about *how* to work (language, style, verbosity, which
  tools to use or to avoid).

### 2. Collect the verifiable environment-facts

Read these from the system, not from memory:

- Repository, current branch, current commit (`git rev-parse HEAD`) and whether the working tree is
  clean (`git status --porcelain`).
- The build-, test- and run-commands which were actually executed successfully in this session,
  verbatim. Never write a command into the export which was not verified — a wrong command costs the
  next session more time than a missing one.
- The relevant files, with their paths and one line each about why they matter.

### 3. Collect tooling and permissions

The next session does not inherit the runtime-permissions of this one. Record therefore:

- Which skills were used, and in which order.
- Which tools and MCP-servers were used.
- Which concrete commands or tool-calls the user approved during this session.
- Which additional working-directories outside the primary one were accessed.
- Which permission-requests the user **denied**, and what was done instead. This is as important as
  the granted ones.

State in the export that these are *records*, not grants: a permission approved in this session must
be approved again in the new one, unless it is persisted in the settings.

### 4. Redact

Never put into the export:

- Secrets, tokens, passwords, private keys, connection-strings with credentials, or content from a
  `SensitiveInformation`-folder.
- Personal data which is not needed to continue the work.
- Large code-dumps. Reference files by path and line instead — the new session can read them itself.

If a secret is relevant for the work, write only *that* it is needed and *where* it comes from
(e.g. "the API-key is read from the environment-variable X"), never its value.

### 5. Write the export

Print the export as one fenced block, fenced with **four** backticks so that inner code-fences
survive the copy-pasting. Print nothing between the fences except the export itself.

Use exactly this structure and keep the headings unchanged — `ai-session-restore` relies on them:

````
# AI-Session-Export

- Format-version: 1
- Exported-at: <ISO-8601-date-and-time>
- Model: <model which created this export>
- Primary-working-directory: <absolute path>
- Repository: <remote-url or "none">
- Branch: <branch>
- Commit: <full commit-hash>
- Working-tree: <clean | dirty, with a short list of the modified paths>

## 1. Task

<What the user wants, in their words. Include scope-corrections and explicitly out-of-scope items.>

## 2. Status

- Done and verified: <list, each with how it was verified>
- In progress: <list, each with where exactly it stands>
- Not started: <list>

## 3. Decisions

<Per decision: what was decided, why, and which alternatives were rejected for which reason.>

## 4. Knowledge about this codebase

<Non-obvious facts, traps, invariants, cross-component-dependencies. Nothing which is trivially
visible in the code.>

## 5. Relevant files

<path - why it matters. One line per file.>

## 6. Verified commands

<Verbatim commands which were executed successfully in this session, each with its purpose.
Only verified ones.>

## 7. Tooling and permissions

- Skills used: <list>
- Tools and MCP-servers used: <list>
- Approved in this session: <concrete commands and tool-calls the user approved>
- Denied in this session: <what was denied, and what was done instead>
- Additional working-directories: <list>

## 8. Dead ends

<Approaches which were tried and rejected, with the reason. Do not repeat these.>

## 9. Open questions

<Questions which need the user, and what is blocked by each of them.>

## 10. Next steps

<Ordered list, concrete enough to be started without asking anything back.>

## 11. Working-preferences of the user

<How the user wants to be worked with: language, style, verbosity, what to ask about, what to just
do, corrections they gave.>
````

Leave out a section only if it is genuinely empty, and then write `- none` under its heading instead
of deleting the heading.

### 6. Offer persistence

After printing the block, offer — do not do it unasked — to additionally write it into a file. If
the user wants that:

- Write it outside the repository, or into a path which is git-ignored, and say explicitly that this
  file can contain project-internals and should not be committed.
- Use a name which sorts chronologically, e.g. `ai-session-export-<yyyy-MM-dd-HHmm>.md`.

## Rules

- The export must be self-contained. Assume the next session has read nothing of this conversation.
- Write facts, not impressions. Every claim must be traceable to something which happened in this
  session or to a command whose output you saw.
- Mark uncertainty as uncertainty. `<assumed>` is fine, silently guessing is not.
- Do not invent progress. What was not verified belongs under "In progress", not under
  "Done and verified".
- Do not shorten to save space at the cost of the reason behind a decision.
- Print the block in one piece, without interleaved commentary, so that the user can select it in
  one go.

## Checklist for the agent

- Is the block self-contained and understandable without this conversation?
- Are branch, commit and working-tree-state read from git and not from memory?
- Is every listed command one which actually ran successfully?
- Does every decision carry its reason?
- Are dead ends and denied permissions recorded, not only the successes?
- Are secrets and personal data excluded?
- Is the structure exactly the one above, so that `ai-session-restore` can read it?
