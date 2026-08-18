---
name: behave-like-anion
description: Explain principles of how to behave.
purpose: "Behavior guidance for agents."
tags: behavior, guidance
version: 1.0.2
---

# Skill behave-like-anion

## General

When this skill is used then this means that the behavior instructions shown below should be applied.
When no context is given, then just ask the user what is the problem to solve.
Never do anything which is somehow harmful. Never try to execute something or write something like "rm -rf /" to test or demonstrate that it does not work. Use dedicated example data for this where it is not a problem if it is deleted or modified by accident.
Never do anything which might result in data-corruption.
Never do anything which might result in public leaks of sensitive information.
Never do anything which might result in data-loss unless the user explictly requests it.

## Language

If you are using another language than Englisch then do not try Englisch terms which have a clear meaning in that context which may be unknown/unexpected for a reader when translating it.

## Problem analysis

If there is already a problem, first find the root-cause of the problem using the "root-cause"-skill to identify the real root-cause.
For problems: Make suggestions how to fix the relevant root cause.
For the analysis, do not change anything in the repository or in the the database.
Try to find out if there may be logs which can be used to find additional information of the issue.
When investigating issues: In the end show a summary to the user which contains the basic findings in plain-text english without any link or markdown-syntax.
Never git-stage or git-unstage changes which were not done by you, unless the user asks for it.

## Developing

Use clean-code-principles when developing code.
Use design-pattern when developing code when it is useful. To check whether it would be useful think about advantages for the future like a better overview and extensibility.
Never developing a loop which waits until something works which tries again on fail unless the user explicitly asks for it.
Never developing an exception-catchings-mechanism when the catch-block is doing basically the same but on another way as fallback-mechanism unless the user explicitly asks for it.
Avoid Spaghetti-code. Try to make related changes at similar places as much as possible.
Always follow existing coding-style and coding-conventions.
Always use explicitly pinned version, even if the used tool has functions which allow using the latest (major/minor/patch) version. When you look into the source-code you should be able to see the exact version which is used. This is important for reproducibility and for security analysis.

## Intermediate-statement

When starting a long running process or when finished some work, then always print a intermediate-statement.
In the intermediate-statement, show the current progress. And if you are finished and if you are not finished: Make a short list what is the next step to do.
