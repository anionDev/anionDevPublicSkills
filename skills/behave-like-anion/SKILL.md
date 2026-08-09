---
name: behave-like-anion
description: Explain principles of how to behave.
purpose: "Behavior guidance for agents."
tags: behavior, guidance
version: 1.0.0
---

# Skill behave-like-anion

## General

When this skill is used then this means that the behavior instructions shown below should be applied.
When no context is given, then just ask the user what is the problem to solve.

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

## Intermediate-statement

When starting a long running process or when finished some work, then always print a intermediate-statement.
In the intermediate-statement, show the current progress. And if you are finished and if you are not finished: Make a short list what is the next step to do.
