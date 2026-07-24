---
name: work-with-common-project-structure
description: Contains information about the "common project structure" and how to work with it.
---
The "common project structure" is a set of conventions and best practices for organizing and structuring a software project defined by: https://projects.aniondev.de/PublicProjects/Common/ProjectTemplates/-/raw/main/Conventions/RepositoryStructure/CommonProjectStructure/CommonProjectStructure.md
The usual pipeline-command to run all scripts (for building, linting, etc.) is `scbuildcodeunit` which runs all scripts directly on the machine.
Furthermore there is `scbuildcodeunitsc` to run scripts in a container which provides a standardized environment.
If the pipeline-command exits with 0 then everything is fine. If it exits with a non-zero exit-code then there is an error and the output of the command should be checked for details.
