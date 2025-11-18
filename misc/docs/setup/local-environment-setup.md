# Local Environment Setup

This guide explains how to prepare a local FinFlow development environment and keep code quality checks consistent.

<!-- mdformat-toc start --slug=github --maxlevel=6 --minlevel=2 -->

- [Assumptions](#assumptions)
- [Task Installation](#task-installation)
- [Pre-Commit Installation](#pre-commit-installation)
  - [What Pre-Commit Does](#what-pre-commit-does)
    - [Automatic Run on Commit](#automatic-run-on-commit)
    - [Run Hooks Manually (Full Scan)](#run-hooks-manually-full-scan)
    - [Update Hook Versions](#update-hook-versions)

<!-- mdformat-toc end -->

## Assumptions<a name="assumptions"></a>

Run all commands from the repository root using PowerShell (Windows) unless noted otherwise.

## Task Installation<a name="task-installation"></a>

This repo uses [Task](https://taskfile.dev/) as a cross-platform command runner to manage its build steps.

- Run `task` to list all available commands.
- Run `task --summary task-name` to see details about a specific task.
- Run multiple tasks at once: `task task1 task2 …`.
- Set task variables using env-style syntax: `task sample-task MYVAR=value`.

## Pre-Commit Installation<a name="pre-commit-installation"></a>

Install the pre-commit framework and register the Git hook:

```powershell
pip install pre-commit
pre-commit install
```

### What Pre-Commit Does<a name="what-pre-commit-does"></a>

Pre-commit executes the configured hooks (formatting, linting, security, etc.) defined in `.pre-commit-config.yaml` before each commit. If any hook fails, the commit is blocked until issues are fixed.

#### Automatic Run on Commit<a name="automatic-run-on-commit"></a>

After installation the hooks run every time you commit. No extra action required.

#### Run Hooks Manually (Full Scan)<a name="run-hooks-manually-full-scan"></a>

Execute all hooks against the entire codebase (not just staged changes):

```powershell
task pre-commit:all
```

Use this before larger refactors or prior to opening a PR.

#### Update Hook Versions<a name="update-hook-versions"></a>

Refresh all hook repositories to their latest declared revisions:

```powershell
task pre-commit:update
```

Run periodically to stay current with upstream improvements.
