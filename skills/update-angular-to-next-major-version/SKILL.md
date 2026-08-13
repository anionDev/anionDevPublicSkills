---
name: update-angular-to-next-major-version
description: Updates an Angular project to the next major Angular version, based on the official changelog- and migration-information from the Angular websites.
purpose: "Maintenance for repositories."
tags: maintenance, angular, dependency-update
version: 1.0.0
---

# Skill update-angular-to-next-major-version

## Goal
Update an Angular project from its current major version to the next major version, in a reproducible way and based on the official information of the Angular project.

## When to use
- When a user wants to update an Angular project to a newer major version.
- When the Angular version used in a repository is out of support.

## Core principle
Never update based on assumed knowledge about a specific Angular version.
The concrete steps, breaking changes and migrations of a major version are always retrieved from the official Angular sources at the time of the update.
Update exactly one major version per run. If several major versions have to be skipped, repeat the whole workflow for each major version separately.

## Inputs
- Path to the repository and to the Angular project (`angular.json`, `package.json`).
- The currently used Angular major version.
- The target major version (usually current + 1).

## Workflow

1. Determine the current state
- Read the used versions of `@angular/core`, `@angular/cli` and all further Angular packages from `package.json`.
- Determine the used Node.js-, TypeScript- and package-manager-version.
- Ensure that the working directory is clean and that the project builds and its tests pass before the update. This is the reference state.

2. Retrieve the official information (mandatory)
- Retrieve the migration-information for the concrete transition "current major version -> target major version" from the official Angular update guide: https://angular.dev/update-guide
- Retrieve the release- and support-information (including the required Node.js- and TypeScript-versions) from: https://angular.dev/reference/releases
- Retrieve the changelog with the detailed list of changes and breaking changes from the official Angular repositories, at least:
	- https://github.com/angular/angular/blob/main/CHANGELOG.md
	- https://github.com/angular/angular-cli/blob/main/CHANGELOG.md
- Retrieve the documentation of the automatic migrations (schematics) which the target version provides, from https://angular.dev
- If further Angular-related packages are used (for example Angular Material/CDK), retrieve their own migration- and changelog-information from their official pages as well.
- Summarize the retrieved information as a concrete, project-specific task list before changing anything.

3. Check the prerequisites
- Compare the required Node.js-, TypeScript- and package-manager-versions with the versions used in the repository and in the build-pipeline.
- Update these prerequisites first if the target version requires it, otherwise the update will fail.

4. Perform the update
- Use the official update-mechanism of the Angular CLI (`ng update`) with explicitly pinned target versions instead of implicit "latest"-resolutions.
- Let the automatic migrations of the target version run and review their result.
- Update the further Angular-related dependencies to the versions which are compatible with the target version.

5. Handle the breaking changes manually
- Work through the breaking changes from the retrieved changelog one by one and check for each one whether the project is affected.
- Fix the affected places manually, following the recommended replacement from the official documentation.
- Do not keep deprecated constructs alive with workarounds if the official documentation provides a clean replacement.

6. Validate
- Build the project and run all tests.
- Check that no new build- or lint-warnings were introduced.
- Verify the application manually in the relevant scenarios if the changes affect runtime-behavior.
- Compare the result against the reference state from step 1.

7. Document
- Document in the repository which major version was reached, which manual adjustments were necessary and which sources were used.

## Decision rules
- Official Angular sources always take precedence over assumed knowledge and over third-party articles.
- One major version per run, with a working and validated state in between.
- Always pin versions explicitly, so that the used version is visible in the source-code.
- If an automatic migration produces a result which is unclear or wrong, correct it manually instead of accepting it silently.
- If the update cannot be completed, keep the repository in the last consistent state and report what is blocking.

## Anti-patterns
- Skipping several major versions in one step.
- Updating dependencies without reading the breaking changes.
- Suppressing errors or deactivating strictness-settings to make the build green.
- Mixing the version-update with unrelated refactorings in the same change.

## Short checklist for the agent
- Reference state (build and tests green) established?
- Update guide, releases-page and changelogs retrieved from the official Angular sources?
- Prerequisites (Node.js, TypeScript, package manager) fulfilled?
- Update executed via the official CLI-mechanism with pinned versions?
- All relevant breaking changes checked and handled?
- Build, tests and manual verification successful?
- Result documented?
