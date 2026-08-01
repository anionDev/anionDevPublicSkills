---
name: ai-session-restore
description: Imports an AI-session-export which was created with the ai-session-save-skill into the current session, verifies it against the actual state of the repository and continues the work from there.
purpose: "Session-management for AI-sessions."
tags: session-management
version: 1.0.0
---

# Skill ai-session-restore

## Goal

Take an export created by the `ai-session-save`-skill and bring the current session to the same
level of knowledge as the session which produced it — task, decisions, state, codebase-knowledge,
verified commands, tooling, dead ends and next steps — **without** trusting the export blindly.

An export is a snapshot of a past state. The repository may have moved on since then. The value of
this skill lies as much in detecting that drift as in loading the context.

## When to use

- The user pastes an "AI-Session-Export" block, or points at a file containing one.
- The user says they want to continue the work of an earlier session.
- Not for: understanding an unknown repository from scratch (use `repository-summary`), or reading
  project-documentation.

## Workflow

### 1. Obtain the export

In this order:

1. If the user pasted the export into the prompt, use it.
2. If the user named a file, read it with the `read-file-content`-skill.
3. If neither, ask the user for the export. Do **not** try to reconstruct it from git-history,
   changelogs or guesswork — a fabricated context is worse than no context.

Check that the block starts with `# AI-Session-Export` and carries a `Format-version`. If the
format-version is higher than 1, say so and read it as best as possible, but tell the user that
sections may be unknown to you.

### 2. Verify the export against reality

Before acting on anything in the export, compare it with the actual current state:

- Is the primary working-directory the one named in the export? If not, ask before continuing —
  applying a context to the wrong repository is the most damaging failure-mode of this skill.
- Current branch versus the branch in the export.
- Current commit versus the commit in the export. If they differ, look at what happened in between
  (`git log <export-commit>..HEAD --oneline`).
- Working-tree-state versus the one in the export.
- Do the files listed under "Relevant files" still exist at those paths?

Do not silently repair mismatches. Collect them.

### 3. Report the drift

Give the user a short report before starting any work:

- What was loaded (task, number of decisions, next steps).
- What has changed since the export: commits, moved or deleted files, branch-switches,
  now-dirty or now-clean working tree.
- Which items from "Done and verified" may no longer hold because the code they touched has changed
  since then.
- Which items from "Next steps" have possibly already been done by someone else in the meantime.

If the drift is large enough that the plan in the export no longer fits, say that plainly and
propose an adjusted plan instead of executing an outdated one.

### 4. Adopt the context

Adopt from the export:

- **Task and scope** — including what was explicitly out of scope. Do not widen it.
- **Decisions** — treat them as already made. Do not re-open a decision because you would have
  chosen differently; only re-open one whose stated reason is provably invalidated by the drift
  found in step 2, and then say why.
- **Codebase-knowledge** — as context, but verify a claim before you rely on it for a change. The
  export describes the code as it was at the export-commit.
- **Dead ends** — do not retry them. If you think a dead end deserves a second attempt, ask first.
- **Working-preferences of the user** — apply them immediately, for this whole session.

For anything the export marks as `<assumed>` or uncertain: treat it as a hypothesis, verify it
before building on it.

### 5. Handle commands and permissions

- The "Verified commands" were verified at export-time. Use them, but check the output of the first
  run instead of assuming success.
- The "Tooling and permissions"-section is a **record, not a grant**. The current session has its own
  permissions. Expect to be asked for approval again for the listed commands and tools, and do not
  try to circumvent a permission-prompt because the export says it was approved before.
- If the export lists permissions which will obviously be needed repeatedly, offer to persist them
  in the settings (via the `update-config`-skill) instead of prompting every time. Offer it, do not
  do it unasked.
- If the export lists **denied** permissions, respect them for this session too. Do not retry a
  denied approach.
- If the export names additional working-directories which are not available in this session, say
  which ones are missing and what is blocked by that.
- If the export refers to a skill which is not available in this session, do not work around it:
  abort the affected part and tell the user that this skill has to be made available.
- If the export refers to secrets or to a `SensitiveInformation`-folder which is not available, say
  so and tell the user that they can make it available.

### 6. Continue the work

Present the "Next steps" from the export, adjusted for the drift found in step 2, and ask the user
whether to start with the first one — unless the user already said to just continue, in which case
start.

From here on, work normally. The restore is finished; do not keep referring back to the export.

## Rules

- Never invent context which is not in the export. A missing section stays missing; ask the user.
- Never treat the export as authoritative about the *current* code. It is authoritative about
  intent, decisions and history — the repository is authoritative about its own state.
- Never use the export as a justification to skip a permission-prompt or a safety-check.
- Report honestly what could not be restored (missing directories, missing skills, deleted files),
  and what is blocked by it.

## Checklist for the agent

- Export obtained from the user, not reconstructed?
- Working-directory, branch and commit compared with the export?
- Drift reported to the user before starting work?
- Decisions adopted instead of re-litigated, dead ends not retried?
- Permissions treated as a record, not as a grant?
- Missing skills, directories or files reported instead of worked around?
- Next steps presented in an up-to-date form?
