---
name: work-with-common-project-structure
description: Contains information about the "common project structure" and how to work with it.
purpose: "Information about repository-conventions."
tags: information, conventions
version: 1.0.0
---

# General

## Build codeunits

The "common project structure" is a set of conventions and best practices for organizing and structuring a software project defined by: https://projects.aniondev.de/PublicProjects/Common/ProjectTemplates/-/raw/main/Conventions/RepositoryStructure/CommonProjectStructure/CommonProjectStructure.md
The usual pipeline-command to run all scripts (for building, linting, etc.) is `scbuildcodeunit` which runs all scripts directly on the machine.
Furthermore there is `scbuildcodeunitsc` to run scripts in a container which provides a standardized environment.
If the pipeline-command exits with 0 then everything is fine. If it exits with a non-zero exit-code then there is an error and the output of the command should be checked for details.

## Changelog

The changelog is always located in `<repository>\Other\Resources\Changelog`.
When you change something then then always update the changelog accordingly.
To find the correct changelog-file: Query the current project version using `scshowversion`.
It is important to run this command after implementing the changes, not before it.
This command show the version. The changelog-filename in the changelog-folder is then `v<version>.md`, where `<version>` is the version returned by `scshowversion`.
