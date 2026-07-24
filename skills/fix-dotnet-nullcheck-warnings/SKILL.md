---
name: fix-dotnet-nullcheck-warnings
description: Fixes warnings regarding null and null-references in .NET-projects.
purpose: "Maintenance for repositories."
version: 1.0.0
---

# Skill fix-dotnet-nullcheck-warnings

## Goal
Fixes nullable and NullReference warnings in C# projects systematically and in a technically correct way.

## When to use
- When warnings like `CS8600` to `CS8777` occur during the build.
- When a user explicitly wants to "fix NullReference warnings".
- When nullable annotations are inconsistent and lead to potential runtime `NullReferenceException`s.

## Core principle
First determine all warnings completely, then decide per warning on a technical basis:
1. Can `null` actually occur from a domain point of view?
2. If yes: model nullability correctly (`?`) and ensure null-safe usage.
3. If no: tighten the API/types so that `null` is no longer possible (remove nullability, add guards), without introducing errors.

## Inputs
- Path to the solution or the project.
- Current compiler warnings (build output).
- Domain rules on whether a value is actually optional.

## Workflow
1. Collect nullable warnings
- Run the build with warnings (e.g. `dotnet build`).
- Extract the relevant warnings from the build output.
- Map the warning codes against the Microsoft documentation:
	https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/nullable-warnings

2. Locate the cause per warning
- Open the file and line.
- Check the data flow: where does a potential `null` come from, where is it dereferenced or assigned?
- Do not suppress with `!` before there is a clean justification.

3. Choose a technically correct fix strategy
- Strategy A: `null` is allowed from a domain point of view
	- Set the nullable annotation correctly (`string?`, `MyType?`).
	- Add null checks (`if (x is null) ...`, `ArgumentNullException.ThrowIfNull(...)` where appropriate).
	- Propagate consistently to call sites and return types.

- Strategy B: `null` is not allowed from a domain point of view (preferred)
	- Remove nullability from signatures when the value must never be `null`.
	- Add early guards so that the contract is clear.
	- Guarantee initialization (constructor, required members, sensible defaults).

4. Validate
- Rebuild the project until the relevant nullable warnings are eliminated.
- Ensure that no new compiler errors have been introduced.
- Optionally run tests and check for regressions.

## Decision rules
- Prefer tightening the model over "silencing the warning".
- `!` (null-forgiving operator) only as a last resort and only with a clear invariant.
- Changes must respect API contracts; be especially careful with public APIs.
- With multiple possible fixes: choose the one with the least domain ambiguity.

## Typical fix patterns
- Secure dereferencing:
	- Before: `value.Length`
	- After: null check or safe alternative (`value?.Length`, if technically correct)

- Tighten the parameter contract:
	- If `null` is not allowed: `string? input` -> `string input` + guard
	- If `null` is allowed: `string input` -> `string? input` + defensive usage

- Correct the return contract:
	- If the method can return `null`, mark the return type as nullable.
	- If not, adjust the implementation so that `null` is never returned.

## Anti-patterns
- Setting `#nullable disable` across the board.
- Suppressing warnings instead of fixing the cause.
- Blindly adding `?` everywhere even though non-null holds from a domain point of view.

## Short checklist for the agent
- Warnings pulled from the build?
- Checked against the Microsoft reference?
- Decided per location on a technical basis: allow or remove nullability?
- Null checks/guards added consistently?
- Build clean afterwards?
