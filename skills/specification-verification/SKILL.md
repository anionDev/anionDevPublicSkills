---
name: specification-verification
description: Contains instructions about how to verify specifications.
purpose: "Guidance for verifying specifications."
tags: verification, guidance
version: 1.0.0
---

# Skill specification-verification

Check whether all specifications of this project are fulfilled, if the project has any.

- First check whether the project contains specifications at all. A common example are specifications managed by openspec (typically in an `openspec`-folder). Other examples are requirement-documents, specification-files in the documentation-folder or acceptance-criteria in issues.
- If the project does not contain any specifications then there is nothing to verify and this skill is done.
- If specifications exist then check for each single specification whether the current implementation fulfills it.
- Report for each specification whether it is fulfilled, not fulfilled or only partially fulfilled. For specifications which are not (or only partially) fulfilled explain concretely what is missing.
