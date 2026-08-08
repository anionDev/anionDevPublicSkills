---
name: root-cause
description: Finds the root cause of a problem.
purpose: "Problem diagnosis and analysis."
tags: problem-solving, analysis
version: 1.0.0
---

# Skill root-cause

Find the root cause of a problem which is described by the context.

Definition of root-cause: The root cause is the fundamental reason for the occurrence of a problem.

If there are processing-chains which lead to the problem then the root cause is the first element in the chain which is responsible for the problem.

There may be services which ues other services which are all developed by us.

## Data-issues

The root-cause of a data-set-problem is the original reason why a wrong data-set comes into our system.

## Program-issues

The root-cause of a program-problem is the original reason why a bug is introduced into the system.
When searching for bugs in the source-code, then always look into the git-history.
Unless other information are given, assume that it was already working before and some change (in source-code or in data) introduced the bug.
So if a related file was changed recently then treat it as higher probability that the bug was introduced in this change and investigate this further.

## General summary

When you find the "usual root cause" of a problem, then try to find the real transitive root cause of that problem regarding if it is a data-issue or a program-issue.
Then build an explanation using a root-cause-chain.
A root-cause-chain is a chain of root-cause of oa root-cause of a root-cause and so on.
The first findable root-cause is the "original root cause".
Explain this to the user and show the entire root-cause-chain in a plain-text english summary without any link or markdown-syntax.

## How to fix

Make fix-suggestions or explain things which should be done to fix the root-cause.
Never suggest workarounds, unless the user requests it.
If a proper fix is not possible, then explain why it is not possible.
