---
name: analyze-changes
description: Analyze the changes on the current branch.
---

Summarize the uncommitted git-changes of this branch in short and write them to the current changelog-file if a changelog exists in the current repository and if they are not already listed there.
Creating a changelog-entry makes only sense if the current branch is not "main" or if there are changes. Otherwise do not do anything but print a hint about the wrong branch.
In the end, as last output-line write a suggestion for a good git-commit-command for all changes which are still uncommitted including a good commit-message for it. The commit-subject should be a short summary of the changes in this branch. Optionally you can add a commit-body which describes the changes in more detail. Use the commitskill-skill for that.
The commit-message should be written in english and should not contain any special characters like umlauts, single-quotes, double-quotes, back-slashs or non-ascii-characters.
