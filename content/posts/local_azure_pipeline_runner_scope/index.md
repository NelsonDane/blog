---
date: '2025-06-13T12:08:00-04:00'
draft: false
title: 'Dev Journal 1: Local Azure DevOps Pipelines Runner (Project Concept)'
description: 'Scoping out a project to run Azure DevOps YAML pipelines locally'
series: ['Local Azure DevOps Pipeline Runner']
series_order: 1
tags: ['azure', 'devops', 'yaml', 'golang', 'pipelines']
---

## Introduction
This is the start of a new project that I came up with after spending too much time using `Azure DevOps` at work. One common issue with DevOps pipelines (and previously GitHub Actions), is that the only way to develop and test them is to continually commit and push changes to the remote repository, then wait and see if the pipeline works. This is a slow process that clutters commit history with a bunch of random commits as you try to fix (usually small) issues that pop up. Wouldn't it be nice if you could just run the pipeline locally?

Wouldn't it be nice if there was something like [`act`](https://github.com/nektos/act) for DevOps pipelines?

This exact question was asked back in 2023 on the [Azure DevOps Developer Community](https://developercommunity.visualstudio.com/t/Ability-to-run-Azure-DevOps-pipeline-loc/10366436?space=41&q=Ability+to+build+%26+run+windows+wpf+applications) forum, but didn't gain much traction and was closed by Microsoft. So we need to take matters into our own hands.

## Project Scope
Requirements:
- Support any YAML-based Azure DevOps pipeline (not classic/web-created).
- Support all major features of Azure DevOps pipelines (jobs, steps, variables, templates, etc).
- Possibly support running pipelines with multiple stages in parallel.
- Support running with custom plugins (e.g. `node` scripts)
- Support running pipelines with custom Docker images.

Core Features:
- Be able to lint/validate pipelines before running them.
- Resolve built-in variables and secrets using a `PAT` (Personal Access Token) (or infer them if used/not provided).
- Download and cache pipeline plugins from Azure DevOps.
- As much feature parity with `act` as possible.

Tech Stack:
- Run as a CLI tool written in Go (to match `act`).
- Use Docker to run the pipeline jobs and steps in isolated ephemeral containers.

Restrictions:
- Only support Linux-based agents (maybe Windows if running on a Windows host?).

So, the big milestones for the project are:
- Lint/Validate the pipeline YAML
- Parse the pipeline YAML into a Go struct
- Resolve variables and secrets from Azure DevOps
- Setup a Docker environment to run the pipeline
- Execute the jobs and steps in the pipeline
- Show the output of the pipeline in a user-friendly way

## Program Flow
{{< mermaid >}}
flowchart TD
    A[Start] --> B[Load Pipeline YAML]
    B --> C{Is YAML Valid?}
    C -->|No| D[Show Errors and Exit]
    C -->|Yes| E[Parse Pipeline Structure]
    E --> F[Resolve Variables and Secrets]
    F --> G[Setup Docker Environment]
    G --> H[Execute Jobs and Steps]
    H --> I{Did All Steps Succeed?}
    I -->|No| J[Show Errors and Exit]
    I -->|Yes| K[Show Success Message and Exit]

{{< /mermaid >}}

## Project Status
- Local Lint/Validation from CLI: Done (See below repo) ✅
{{< github repo="NelsonDane/az-pipeline-linter" showThumbnail=false >}}

- Parse Pipeline YAML: In Progress 🛠️
- Resolve Variables and Secrets: Not Started ❌
- Setup Docker Environment: Not Started ❌
- Execute Jobs and Steps: Not Started ❌
- Show Output: Not Started ❌
