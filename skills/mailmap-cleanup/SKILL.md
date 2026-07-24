---
name: mailmap-cleanup
description: Builds or cleans up a git .mailmap file for a repository, consolidating duplicate author identities (typo domains, GitHub noreply emails, AD usernames, case variants) into canonical names/emails. Use when the user asks to deduplicate git authors, "clean up" or "aufräumen" the .mailmap, or normalize `git log`/`git shortlog` author lists.
purpose: "Maintenance for repositories."
version: 1.0.0
---

# Git `.mailmap` Cleanup

Consolidates the many raw author identities that accumulate in a repository's git history (typo'd
email domains, GitHub noreply addresses, AD/Windows usernames, mixed-case duplicates, "Lastname,
Firstname" formatting) into one canonical `Name <email>` per real person, via a `.mailmap` file at
the repository root.

## Completion criterion

A first draft is done when every distinct identity in `git log --format='%aN|%aE' | sort -u` has
either been merged into a canonical identity, or deliberately left unmapped because no confident
identity resolution was possible. The task is fully done only after the user has reviewed and
confirmed the uncertain merges (see Step 4).

## Step 1 — Gather the raw identity list

```bash
git log --format='%aN|%aE' | sort -u
```

This is the ground truth. Re-run it after every batch of edits — see Step 5.

## Step 2 — Establish the canonical rules with the user

Do not invent domain-preference or formatting rules — ask, unless the user already stated them.
Typical questions to clarify upfront:

- **Canonical email preference order**, e.g. "primary corporate domain > secondary corporate
  domain > everything else". Get the actual domain(s) from the user or infer from the most common
  domain in the raw list, but confirm before applying broadly.
- **Casing rule** for canonical emails (commonly: always fully lowercase).
- **Identity-merging rule**: typically, merge two raw identities if they clearly belong to the same
  real person (same name spelled differently, same email with a typo'd domain, a GitHub noreply
  address whose raw commit author name already matches a known person, an AD username that maps
  1:1 to a known person via other evidence).
- **Unresolvable-identity rule**: if no confident real name can be derived from an entry (bare
  usernames, opaque AD IDs, cryptic logins with no corroborating name/email evidence), leave it
  unmapped rather than guessing. Do not silently drop it either — surface it to the user as an open
  question.
- **Bots**: exclude bot accounts (e.g. `dependabot[bot]`, `*-agent[bot]`, CI service accounts) from
  person-identity merging entirely — leave them out of the mailmap.
- **Name formatting**: normalize raw "Lastname, Firstname (department/suffix)" style names to
  "Firstname Lastname" in the canonical entry.
- **Comments**: ask whether explanatory comments are allowed inside `.mailmap`. Some users
  explicitly forbid them (keep the file itself clean) and want all reasoning delivered only in
  chat — default to no comments if unsure, since comments are easy to add later but annoying to
  strip out.

## Step 3 — Know the `.mailmap` line formats

| Format | Syntax | Effect |
|---|---|---|
| 1 | `Proper Name <commit-email>` | Rewrites only the **name** for any commit using that email (email match is case-insensitive). Does **not** change the email's casing in output. |
| 2 | `<proper-email> <commit-email>` | Rewrites only the **email**, keeps the original commit name. |
| 3 | `Proper Name <proper-email> <commit-email>` | Rewrites **both** name and email for any commit matching `commit-email`. This is the only form that forces a canonical (e.g. lowercase) email to appear in output. |
| 4 | `Proper Name <proper-email> Commit Name <commit-email>` | Most specific: matches only when **both** the commit's name and email match. Rarely needed — use only when the same email is shared by genuinely different people and the name is the only disambiguator. |

**Critical gotcha (lesson learned)**: git's mailmap email matching is case-insensitive, but a
format-1 rule (name-only) does **not** normalize the email's casing in output — it only fixes the
displayed name. If the raw history contains a mixed-case variant of an otherwise-correct email
(e.g. `John.Doe@example.com` vs `john.doe@example.com`), a format-1 rule alone leaves the mixed-case
variant as a separate line in `git log` output. Fix this by adding an explicit format-3 redirect
line (`Proper Name <canonical-lowercase-email> <Mixed.Case.Variant@example.com>`) for every
case-variant found.

## Step 4 — Build the draft, separating confident merges from guesses

For each raw identity:

1. Try to resolve it to a real name using available evidence: an identical/near-identical name on
   another commit, a corporate email with the same local-part, a GitHub noreply email whose numeric
   ID or login correlates with a known contributor, etc.
2. **High-confidence merges** (clear name match, matching email local-part, obvious typo'd domain):
   apply directly using format 1–3 as appropriate.
3. **Low-confidence merges** (username-similarity only, one-off broken self-referential emails,
   surname-only matches, disputed or ambiguous accounts): do **not** merge silently. Flag each one
   explicitly to the user (e.g. as a short table: raw identity → proposed person → evidence/reason)
   and wait for confirmation or correction before writing the merge into `.mailmap`.
4. If the user disputes a proposed merge, remove it and leave that identity unmapped until they
   give an explicit resolution — do not re-propose the same guess again without new evidence.
5. Never fabricate a name for a raw identity that carries no name evidence at all; leave it out of
   the mailmap and flag it as unresolved instead.

Apply confirmed corrections one at a time as the user reports them (this is typically an iterative,
interactive process — the user may independently check `git log --author="<raw-identity>"`
themselves and report back the real name).

## Step 5 — Re-verify after every batch of edits

After writing or changing `.mailmap` entries, always re-run:

```bash
git log --format='%aN|%aE' | sort -u
```

and check that:

- Every merged identity has fully disappeared as a separate line (folded into its canonical form).
- No canonical email appears twice in different casing (the Step 3 gotcha).
- The remaining unmapped/flagged identities are exactly the ones still pending user confirmation.

Report the before/after counts concisely (e.g. "41 → 39 unique identities") rather than dumping
the full list, unless the user asks for detail.

## Non-goals

- Do not add comments to `.mailmap` unless the user explicitly allows it — keep explanations in the
  chat response only, by default.
- Do not commit the `.mailmap` change unless the user explicitly asks for it.
- Do not merge two identities on a guess when the user has not confirmed it, even if the guess
  seems obvious — mailmap identity resolution is about real people and should not be based on
  unverifiable inference alone.
