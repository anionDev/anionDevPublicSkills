---
name: fix-merge-conflicts
description: Resolves complicated merge conflicts in a git repository (merge, rebase, cherry-pick or stash-pop conflicts), by reconstructing the intent of both sides from the history instead of guessing from the conflict markers. Use when the user asks to resolve/fix merge conflicts, when a merge, rebase or cherry-pick stopped with conflicts, or when a conflict is too large or too semantic to resolve by simply picking one side.
purpose: "Maintenance for repositories."
version: 1.0.0
---

# Fix Merge Conflicts

Resolves conflicts that arise from `git merge`, `git rebase`, `git cherry-pick`, `git revert`,
`git stash pop` or `git am`. The guiding principle: **a conflict is a question about intent, not
about text.** Both sides changed the same region because both sides wanted something. The
resolution must preserve both intents, or explicitly and knowingly drop one of them.

Never resolve a conflict by picking whichever side makes the markers disappear fastest.

## Completion criterion

The task is done when:

1. `git status` reports no unmerged paths and no remaining conflict markers exist anywhere in the
   working tree (see Step 7).
2. The project builds and its tests pass (see Step 8).
3. Every non-obvious resolution decision — especially every place where one side's change was
   intentionally dropped — has been reported to the user.

The merge/rebase is **not** finalized (no `git commit`, no `git rebase --continue`, no
`git push`) unless the user explicitly asks for it. See "Non-goals".

## Step 0 — Never destroy the user's work

Before touching anything, protect the ability to start over:

```bash
git status
git stash list
git rev-parse HEAD MERGE_HEAD 2>/dev/null
```

Rules:

- **Do not change what is checked out.** No `git checkout <commit|branch>`, no `git switch`, no
  `git reset` in any form. The conflict state itself is the thing being worked on; moving HEAD
  away from it destroys exactly the state that has to be resolved.
- **Do not create commits.** No `git commit`, no `git merge/rebase/cherry-pick --continue`, no
  `git stash` (stashing creates commit objects and empties the working tree), no `git tag`. The
  resolution is delivered as a resolved working tree, nothing more.
- **Do not** run `git merge --abort`, `git rebase --abort`, `git reset --hard`, `git checkout
  --theirs/--ours <file>` on a whole file, or `git checkout -- <file>` without explicit user
  consent. Each of these can silently discard hours of work.
- The only git commands that may modify anything are: writing resolved file content, `git add
  <file>` for a resolved path, `git rm`/`git add` for delete-conflicts, and re-materializing the
  conflict inside a single file (`git checkout --conflict=… -- <file>`, see Step 3). Everything
  else in this skill is read-only inspection.
- If the repository has uncommitted unrelated changes on top of the conflict state, point this out
  to the user before proceeding — that combination is the most common way work gets lost.
- If the user asks for a rescue point, `git branch backup/pre-conflict-fix` is safe: it only adds a
  label to the current commit and touches neither HEAD nor the working tree.

## Step 1 — Determine which operation is in progress

The correct interpretation of "ours" and "theirs" depends entirely on the operation, and this is
the single most frequent source of wrong resolutions:

| Operation | `--ours` / `HEAD` / left marker side | `--theirs` / right marker side |
|---|---|---|
| `git merge feature` (on `main`) | `main` — the branch you are on | `feature` — the branch being merged in |
| `git rebase main` (rebasing `feature`) | **`main`** — the upstream you replay onto | **`feature`** — your own commits being replayed |
| `git cherry-pick <c>` | the current branch | the cherry-picked commit |
| `git revert <c>` | the current branch | the inverse of the reverted commit |
| `git stash pop` | the working tree | the stashed changes |

**During a rebase, "ours" and "theirs" are inverted relative to intuition.** "Theirs" is your own
work. Verify with:

```bash
ls .git/MERGE_HEAD .git/rebase-merge .git/rebase-apply .git/CHERRY_PICK_HEAD 2>/dev/null
```

State the detected operation and the ours/theirs mapping explicitly before resolving anything.

## Step 2 — Get the full list of conflicts and classify them

```bash
git diff --name-only --diff-filter=U
git status --short
```

Classify every conflicted path into one of these buckets, because the correct treatment differs:

- **Generated / lock / build artifacts** (`package-lock.json`, `*.csproj.user`, `*.min.js`,
  compiled output, `poetry.lock`, migration bundles): do not hand-merge. Take one side and
  **regenerate** the file with the project's own tooling. Hand-merging a lockfile produces a file
  that is internally inconsistent.
- **Append-only / list-like files** (changelogs, `.gitignore`, resource lists, DI registration
  blocks, `using`/`import` blocks): usually **both sides** are wanted. The conflict is only about
  ordering.
- **Version / metadata files** (version numbers, `.csproj` versions, `setup.py`): take the higher
  or the intended release version — never blindly the incoming one. If in doubt, ask.
- **Real source-code conflicts**: the actual work, handled by Steps 3–6.
- **Rename/delete and delete/modify conflicts** (`git status` shows `DU`, `UD`, `AA`): these have
  no textual merge at all. The decision is "does this file still exist, and under which name?" —
  resolve with `git rm` / `git add` after determining intent, and check for stale references to
  the old path elsewhere in the repository.

Handle the easy buckets first so the remaining set is small and purely semantic.

## Step 3 — Reconstruct the intent of both sides

This is the core of the skill and the part that must not be skipped for complicated conflicts.
For each real source conflict, answer three questions before writing a single line:

1. **What did each side change, and why?**

   ```bash
   git log --oneline --left-right --merge -- <file>   # commits from both sides touching this file
   git log -p HEAD...MERGE_HEAD -- <file>             # what actually changed on each side
   git log --format='%h %s%n%b' -1 <commit>           # the commit message = the stated intent
   ```

   The commit messages, linked issues and PR titles are the primary evidence of intent. Read them.

2. **What did the common ancestor look like?**

   ```bash
   git merge-base HEAD MERGE_HEAD
   git show :1:<file> > /tmp/base   # stage 1 = base, 2 = ours, 3 = theirs
   git show :2:<file> > /tmp/ours
   git show :3:<file> > /tmp/theirs
   ```

   Diffing base→ours and base→theirs separately is far more informative than staring at the merged
   conflict markers, especially when the hunks are large or interleaved.

3. **Are the two changes actually related?** Often two sides touch adjacent lines for entirely
   unrelated reasons. Then the resolution is trivially "both". A conflict is only genuinely hard
   when both sides changed the *same behavior*.

Enable `diff3`/`zdiff3` style to see the base inline — it turns most "impossible" conflicts into
obvious ones:

```bash
git config merge.conflictStyle zdiff3
git checkout --conflict=zdiff3 -- <file>   # re-materialize the conflict with base included
```

The second command is path-scoped (`-- <file>`) and only rewrites that one still-unresolved file
from its stages — it does not check out a commit and does not move HEAD. Run it only on files
whose conflict has not been edited yet, since it discards manual edits in that file.

## Step 4 — Resolve by intent, using the decision order

For each conflict hunk, apply in this order:

1. **Both changes are compatible** → integrate both. This is the correct answer far more often than
   people expect. Do not drop one side merely because integrating is more work.
2. **One side supersedes the other** (a refactoring that already covers the other side's fix, a
   later rewrite of the same function) → take the superseding side, but verify the subsumed side's
   *behavior* is truly present in the result, not just its general area of code.
3. **The changes are semantically incompatible** (both changed the same logic in different
   directions) → this is a genuine design decision. Do **not** invent a compromise silently. Ask
   the user, presenting: what each side does, why (per commit message), and your recommendation.
4. **You cannot determine intent from the history** → ask. Guessing produces code that compiles,
   passes review, and is wrong.

Additional rules while resolving:

- Adapt one side's change to the other side's new structure. If `theirs` fixed a bug in a method
  that `ours` renamed/extracted, the fix must be **re-applied inside the new structure**, not
  pasted back as a duplicate of the old method.
- Never leave both variants "just in case" (duplicated methods, both old and new call, commented-out
  alternative). That is not a resolution.
- Preserve formatting/indentation of the surviving structure — do not import the other side's
  formatting only because it came from the conflict block.
- Watch for conflicts whose two sides are pure formatting/whitespace/line-ending noise; check with
  `git diff -w` before treating them as real. On Windows repositories, CRLF/LF differences are a
  frequent cause of whole-file "conflicts".

## Step 5 — Look beyond the conflict markers (semantic conflicts)

A conflict-marker-free file is **not** a merged file. Git only detects textual overlap. The most
dangerous merge damage is where git merged cleanly but the result is wrong:

- Side A renamed a symbol; side B added new usages of the old name in a different file → clean
  merge, broken build.
- Side A changed a method signature/contract; side B added a caller with the old signature.
- Side A deleted a helper/config key; side B started using it.
- Both sides added a member/route/DI registration/enum value with the same name in different files.
- Both sides added a database migration with the same parent — clean merge, broken migration chain.
- Both sides bumped the same dependency to different versions in different files.

After resolving markers, actively grep for the symbols touched by both sides across the *whole*
repository, not only the conflicted files. Then rely on the build (Step 8) as the second net.

## Step 6 — Use git's own machinery where it helps

- `git rerere`: if the same conflict keeps reappearing (long-running rebases, repeated merges of the
  same branch), suggest `git config rerere.enabled true` to the user so resolutions are recorded and
  replayed automatically.
- `git checkout --ours|--theirs -- <file>`: acceptable **only** for the generated/lock bucket from
  Step 2, or when the user explicitly decided one side wins for that file. Always path-scoped,
  never as a default strategy for source code.
- For a very large, painful rebase, the cheapest fix for a conflict is sometimes not having it: a
  merge instead of replaying dozens of commits, or `git rebase --onto` to skip commits that are
  already upstream. **Only describe this to the user** — restarting the operation differently means
  discarding the current conflict state, which is the user's decision, not this skill's.

## Step 7 — Verify no conflict remains

```bash
git diff --name-only --diff-filter=U          # must be empty
git grep -nE '^(<{7}|={7}|>{7})( |$)'          # must find nothing (also checks non-conflicted files)
git status --short
```

The `git grep` is mandatory: conflict markers routinely get committed inside strings, tests,
documentation or files that were resolved by hand earlier in the session.

Then stage the resolved files explicitly with `git add <file>` — per file, never `git add -A`,
so that unrelated working-tree changes are not swept into the merge.

## Step 8 — Build and test before declaring success

Run the repository's own build and test commands (the ones the project defines — see the
`work-with-common-project-structure` skill for repositories following the common project
structure). A merge resolution that was not compiled and tested is unverified.

If the build fails, treat the failure as part of this task — it is almost always a Step 5 semantic
conflict, not an unrelated problem.

## Step 9 — Report

Report to the user, concisely:

- The operation and the ours/theirs mapping used.
- Per conflicted file: one line stating how it was resolved (both integrated / ours / theirs /
  rewritten / regenerated).
- **Explicitly and prominently: every change from either side that was intentionally dropped**, and
  why. This is the information the user cannot recover by reading the diff.
- Any semantic conflicts found in files that were not themselves conflicted.
- Any decisions that remain open / need the user's judgement.

## Non-goals

- Do **not** create any commit: no `git commit`, no `git rebase/merge/cherry-pick --continue`, no
  `git stash`, no push — not even when the resolution looks finished and verified. The deliverable
  is the resolved working tree; finalizing it is the user's step.
- Do **not** check out anything: no `git checkout <commit|branch>`, no `git switch`, no
  `git reset`. Path-scoped file operations on still-conflicted files (`git checkout --conflict=… --
  <file>`, `git checkout --ours|--theirs -- <file>`) are the only exception, and the latter only
  under the conditions in Step 6.
- Do **not** abort the operation on your own initiative.
- Do **not** perform unrelated refactoring, reformatting or cleanup inside conflicted files — it
  makes the merge diff unreviewable, which is exactly when review matters most.
- Do **not** "resolve" a conflict by deleting the conflicting code, disabling a test, or commenting
  a block out.
- Do **not** guess an intent that the history does not support — ask instead.
