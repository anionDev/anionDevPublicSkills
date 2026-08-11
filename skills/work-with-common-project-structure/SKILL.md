---
name: work-with-common-project-structure
description: Contains information about the "common project structure" and how to work with it.
purpose: "Information about repository-conventions."
tags: information, conventions
version: 1.0.0
---

# General

## General information

The "common project structure" is a set of conventions and best practices for organizing and structuring a software project defined by: https://projects.aniondev.de/PublicProjects/Common/ProjectTemplates/-/raw/main/Conventions/RepositoryStructure/CommonProjectStructure/CommonProjectStructure.md
It follows convention-over-configuration, so most things are located at a well-defined place and do not have to be searched for.

### Codeunits

A repository contains one or more `codeunit`s. A codeunit is an independently compilable part of the product (for example a library, a backend, a frontend or an image-definition). Small products have exactly one codeunit, larger ones (monorepos) have several.

- A codeunit is located in `<repository>/<codeunit>`, its source-code usually in `<repository>/<codeunit>/<codeunit>` and its testcases in `<repository>/<codeunit>/<codeunit>Tests`.
- Testcases belong to the codeunit and are a mandatory part of it (as long as the codeunit has testable source-code).
- A codeunit must not include the source-code of another codeunit of the same repository directly. Dependent codeunits are consumed as artifacts via `<codeunit>/Other/Resources/DependentCodeUnits`.
- Each codeunit has its own version, which is independent of the product-version.
- Build-results are placed in `<codeunit>/Other/Artifacts/<artifactname>` (git-ignored). Common artifacts: `BuildResult_<TargetEnvironment>`, `TestCoverage`, `TestCoverageReport`, `Reference`, `BOM`, `APISpecification`.

### Automation

A lot happens automatically, because the whole build-pipeline is implemented as python-scripts at defined locations inside the codeunit. These scripts are mostly thin wrappers which delegate to the `ScriptCollection`-package (see https://github.com/anionDev/ScriptCollection), so the actual logic is usually not duplicated in the repository.

The scripts of a codeunit and what they do:

| Script | Purpose |
|---|---|
| `<codeunit>/Other/CommonTasks.py` | Preparation-tasks, for example updating the version and building/copying dependent codeunits. |
| `<codeunit>/Other/UpdateDependencies.py` | Updates the dependencies of the codeunit. |
| `<codeunit>/Other/QualityCheck/Linting.py` | Runs the linting. Exits non-zero on linting-issues. |
| `<codeunit>/Other/QualityCheck/RunTestcases.py` | Runs the testcases, generates the test-coverage and checks the minimal-code-coverage-threshold. |
| `<codeunit>/Other/Reference/GenerateReference.py` | Generates the html-reference from `<codeunit>/Other/Reference/ReferenceContent`. |
| `<codeunit>/Other/Build/Build.py` | Builds the productive build-artifacts (without unit-tests). |
| `<codeunit>/Other/OnBuildingFinished.py` | Optional script for tasks which run after all other scripts. |

All of these scripts are runnable unattended on every developer-machine, they exit with 0 on success and with a non-zero exit-code on failure. Running the whole pipeline is idempotent: it must not change any not-git-ignored file.

## Defined source of information

Because of the conventions, information about the product and its codeunits does not have to be guessed. It can be read from defined files:

`<repository>/<codeunit>/<codeunit>.codeunit.xml` (must be valid against `codeunit.xsd`) defines per codeunit:

- `enabled`-attribute: whether the codeunit is built and processed at all. Disabled codeunits are skipped by the pipeline.
- `codeunitspecificationversion`-attribute: the version of the common-project-structure-specification the codeunit implements.
- `name` and `version` of the codeunit.
- `codeunitownername` and `codeunitowneremailaddress` as well as the `developerteam` (name and email-address of each developer).
- `properties/@description`: short description of the codeunit.
- `properties/@developmentstate`: for example "Active development", "Maintenance updates only" or "Inactive".
- `properties/@codeunithastestablesourcecode`: whether the codeunit has testable source-code at all (if `false`, there are no testcases and no `RunTestcases.py`).
- `properties/@codeunithasupdatabledependencies`: whether the codeunit has dependencies which can be updated (if `false`, there is no `UpdateDependencies.py`).
- `properties/testsettings/@minimalcodecoverageinpercent`: the minimum test-coverage-threshold which the testcases must reach.
- `properties/pipelinedemands`: tools (optionally with version) which must be available to build the codeunit.
- `properties/updatesettings/ignoreddependencies`: dependencies which are deliberately not updated.
- `dependentcodeunits`: the names of all codeunits of the same repository this codeunit depends on.

`<repository>/.ScriptCollection/ProductInformation.xml` (must be valid against `productinformation.xsd`) defines per product/repository:

- `producttitle`: the name of the product.
- `remoteaddress`: the address of the remote-repository.
- `requiredenvironmentvariables`: the names (not the values) of the environment-variables which must be set to build the product.

Further defined sources are the `ReadMe.md`-files (product- and codeunit-level, including the development-state), `<codeunit>/Other/Reference/ReferenceContent/Hints.md` (requirements to run the scripts) and `HowToBuild.md`, and `GitVersion.yml` for the versioning.

## Build codeunits

The usual pipeline-command to run all scripts (for building, linting, etc.) is `scbuildcodeunits` which runs all scripts directly on the machine.
`scbuildcodeunits` also has a `-c`-switch (`--runincontainer`) which runs the scripts in a container which provides a standardized environment.
If the pipeline-command exits with 0 then everything is fine. If it exits with a non-zero exit-code then there is an error and the output of the command should be checked for details.

## Changelog

The changelog is always located in `<repository>\Other\Resources\Changelog`.
When you change something then then always update the changelog accordingly.
To find the correct changelog-file: Query the current project version using `scshowversion`.
It is important to run this command after implementing the changes, not before it.
This command show the version. The changelog-filename in the changelog-folder is then `v<version>.md`, where `<version>` is the version returned by `scshowversion`.
