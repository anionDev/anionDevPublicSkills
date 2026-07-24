---
name: repository-debt-analysis
description: Analyzes a repository for technical debt and non-functional weaknesses (code smells, duplication, structural erosion, missing tests, outdated dependencies, undocumented decisions) and produces a prioritized, evidence-based report. Read-only analysis, this skill does not change any file. Use when the user asks for a technical-debt analysis, a maintainability assessment, a refactoring backlog, or "where is this repository rotting".
purpose: "Analysis for repositories."
version: 1.1.0
---

# Repository Technical-Debt Analysis

Produces a prioritized inventory of the technical debt of a repository, together with an estimate
of what each item costs and what removing it costs.

Two principles drive everything below:

1. **Debt only matters where the code is touched.** Ugly code that nobody reads and nobody changes
   pays no interest. A moderate smell in the file that changes every week is far more expensive
   than a severe smell in a file untouched for five years. Prioritization must therefore be based
   on evidence from the repository's history, not on aesthetic judgement of a snapshot.
2. **A finding without evidence is an opinion.** Every finding names a concrete location, shows
   what is actually there, and states why it costs something. "Could be cleaner" is not a finding.

This skill is **read-only**: it produces a report, it does not modify, fix, refactor or reformat
anything (see "Non-goals").

## Completion criterion

The analysis is complete when:

1. The analysis scope is explicit — which parts of the repository were examined and which were
   deliberately excluded (Step 1).
2. Every finding has: location, evidence, cost of keeping it, estimated effort to fix, confidence.
3. The findings are prioritized against each other, not just listed.
4. Areas that were examined and found healthy are stated explicitly, so the report is
   interpretable as a coverage statement and not only as a list of complaints.

## Step 0 — Delimit against the neighbouring skills

Technical debt is a broad term. Do **not** duplicate work that other skills own, but do reference
their subject areas as debt items when they are relevant:

| Concern | Owning skill | Treatment here |
|---|---|---|
| Security weaknesses, OWASP categories, CVEs, exploitable bugs | `owasp-top-10-analysis`, `security-audit`, `bug-hunter`, `analyze-cve-finding` | Only note *structural* security debt (e.g. "no input-validation layer exists at all", "secrets handling is ad-hoc per module") and refer to those skills for the actual security analysis. |
| Typos, formatting, lint-rule violations | `project-linting` | Report only if the repository has **no** linting setup at all, or if the setup exists but is not enforced. Do not list individual lint findings. |
| Missing unit tests | `write-tests` | Report test-coverage debt as a structural finding (which critical areas are untested), not as a list of individual missing tests. |
| Missing/outdated documentation | `write-documentation` | Report documentation debt where it blocks maintenance (undocumented architecture, dead README instructions), not stylistic doc issues. |
| Deviation from the repository conventions | `work-with-common-project-structure` | If the repository follows the "common project structure", check it against those conventions and report deviations as structural debt. |

Everything else — code smells, duplication, architectural erosion, dependency rot, build/CI debt,
process debt — is this skill's own subject.

## Step 1 — Establish the scope, and exclude what must be excluded

Before analyzing, determine what actually belongs to the repository's own maintained code:

```bash
git ls-files | wc -l
git submodule status
```

Exclude from the analysis (but state that you excluded them):

- **Submodules** — they are separate repositories with their own debt.
- **Generated code** (parsers, API clients, `*.designer.cs`, protobuf/OpenAPI output, migrations
  scaffolded by a tool). Debt in generated code is debt in the generator or its configuration —
  report it there, if at all.
- **Vendored / third-party code** checked into the repository.
- **Git-ignored and binary files.**
- **Test fixtures and sample data**, unless the debt *is* the fixture management.

Then get the shape of what remains: languages, code units/modules, entry points, build system, test
projects, CI configuration. If the repository follows the common project structure, use that layout
to name the components consistently in the report.

State the scope explicitly at the top of the report. An analysis whose scope is unclear cannot be
acted on.

## Step 2 — Gather evidence from the history (hotspots)

This step is what separates a useful debt report from a generic code review. Find where the
repository actually hurts:

```bash
# Change frequency per file over the last year (the "interest rate")
git log --since=1.year --format=format: --name-only -- . | sort | uniq -c | sort -rn | head -40

# Files with the most distinct authors (coordination cost / unclear ownership)
git log --format='%aN' --name-only --since=2.years

# Recent bug-fix commits — where do fixes cluster?
git log --since=1.year --oneline --grep='fix\|bug\|hotfix\|workaround' -i

# Age and size of the biggest files
git ls-files | xargs wc -l 2>/dev/null | sort -rn | head -30
```

(On Windows, run these through the Bash tool, or use the PowerShell equivalents — the point is the
data, not the exact command.)

From this, derive the **hotspots**: files or modules that are simultaneously large/complex and
frequently changed. These get the deepest analysis. Files that are large but frozen get a
one-line mention at most.

Also look for history smells:

- Repeated "fix the fix" commits in the same area → the design there does not hold.
- Commits that touch many unrelated modules at once → missing module boundaries / high coupling.
- Long-lived `TODO`/`HACK`/`FIXME`/`XXX`/`workaround` markers — count them, date them via
  `git log -S`, and check whether their referenced issues still exist.

## Step 3 — Analyze the hotspots and the structure

For the hotspots from Step 2, and for the repository as a whole, assess:

**Code-level debt**

- Code smells: overly long methods/classes, deep nesting, long parameter lists, boolean/flag
  parameters, primitive obsession, feature envy, god objects, static mutable state.
- Duplication: real semantic duplication (the same rule implemented twice, which will drift), not
  incidental textual similarity. Verify a duplicate really is one before reporting it.
- Error handling: swallowed exceptions, catch-all blocks, error paths that cannot be tested,
  inconsistent failure semantics across modules.
- Dead code: unreferenced classes/methods, feature flags that are permanently on/off, commented-out
  blocks, obsolete compatibility shims. Verify "unused" by searching the whole repository
  (including reflection, DI registration, configuration files and scripts) before claiming it.
- Concurrency and resource handling debt: undisposed resources, ad-hoc locking, blocking calls in
  async paths.

**Structural / architectural debt**

- Module boundaries: does the dependency direction match the intended architecture? Are there
  cycles between modules? Does the domain layer depend on infrastructure?
- Layering violations and shortcuts ("just this once" direct database access from the UI layer).
- Abstractions that do not pay for themselves: interfaces with exactly one implementation and no
  test-double purpose, indirection layers that only forward calls.
- Missing abstractions: the same concept expressed differently in three places.
- Configuration debt: hardcoded values, environment-specific values in code, config duplicated
  between deployment and code.

**Build, dependency and process debt**

- Dependencies: outdated or unmaintained packages, multiple libraries doing the same job, pinned
  ancient versions, direct dependencies that are actually transitive. Note end-of-life runtimes and
  frameworks explicitly — those are the most expensive debt items in any repository.
- Build/CI: slow or flaky builds, manual release steps, disabled/skipped tests, suppressed warnings
  and lint rules (`#pragma warning disable`, `// eslint-disable`, `[SuppressMessage]`) — count them
  and check whether each is justified.
- Missing safety net: which critical paths have no automated test at all? Debt is much more
  expensive where there is no test to catch the consequences.
- Knowledge debt: components with a single author, undocumented non-obvious decisions, no ADRs
  (see the `domain-modeling` skill if the repository maintains a domain model).

## Step 4 — Distinguish debt from deliberate design

Before reporting anything, apply this filter — it is the main source of false positives:

- Is this pattern used **consistently** across the repository? Then it is a convention, not a smell.
  If you disagree with the convention, say so once as a single global finding, not as fifty local
  ones.
- Is there a comment, ADR, README or commit message that explains the choice? Then it is a
  deliberate trade-off. Report it only if the reason has since expired.
- Is it a known deliberate simplification for a young project (no abstraction yet, no caching yet)?
  Then it is not debt yet — it becomes debt when the code starts fighting back. Say which trigger
  would make it debt.
- Would the "fix" be a rewrite of a working, stable, rarely-touched component? Then the honest
  recommendation is usually "leave it alone".

Debt that will never be paid back is not worth listing. Prefer a short report of real items over an
exhaustive one nobody will act on.

## Step 5 — Rate and prioritize

For each finding, determine:

- **Impact / interest** (what it costs while it stays): Critical / High / Medium / Low. Justify via
  hotspot data — how often is this code touched, and what breaks or slows down because of it?
- **Effort** to remove it: Small (< 1 day) / Medium (a few days) / Large (a project of its own).
- **Risk** of the fix itself: is there test coverage that would catch a regression? A Large fix in
  an untested hotspot is a different proposition from the same fix with a good test suite.
- **Confidence**: Confirmed (evidence in the code) / Likely / Assumption (needs the user's domain
  knowledge to verify). Never present an assumption as a fact.

Prioritize primarily by **impact ÷ effort**, and pull forward anything that blocks other work
(e.g. "no test harness exists" blocks every other refactoring, so it comes first regardless of its
own score). Explicitly name the items that are *not* worth fixing.

## Step 6 — Report

Deliver:

1. **Scope statement** — what was analyzed, what was excluded and why (Step 1).
2. **Overall assessment** — a few sentences: is this repository healthy, aging, or in trouble, and
   what is the single dominant problem if there is one.
3. **Findings**, grouped by category (code / structure / dependencies / build & CI / tests /
   knowledge). Per finding: ID, title, location(s), evidence, why it costs something, impact,
   effort, risk, confidence, and a concrete recommendation.
4. **Hotspot list** — the files/modules where change frequency and complexity coincide, since these
   are where every future euro of effort will be spent.
5. **Healthy areas** — explicitly state what was examined and found in good shape.
6. **Prioritized action plan** — an ordered, ready-to-use backlog: quick wins first, then the
   structural items, with the "do not fix" list at the end and a one-line justification each.
7. **Summary table** of all findings with at least: ID, category, location, impact, effort,
   confidence.

Keep the prose tight. The report is a work list, not an essay.

## Non-goals

- Do **not** change, fix, refactor, reformat or reorganize any file. This skill produces a report
  only. If the user wants fixes afterwards, that is the `improve-product`, `small-fix` or
  `project-linting` skill's job.
- Do **not** create issues, branches or commits.
- Do **not** report findings without a concrete location and evidence.
- Do **not** pad the report with generic best-practice advice that is not tied to something
  actually present in this repository.
- Do **not** analyze submodules, generated code or vendored code as if it were the repository's own.
- Do **not** turn this into a security audit — refer to the security skills instead (Step 0).
