# Local Environment Setup

This guide explains how to prepare a local FinFlow development environment and keep code quality checks consistent.

## Assumptions

Run all commands from the repository root using PowerShell (Windows) unless noted otherwise.

## Pre-Commit Installation

Install the pre-commit framework and register the Git hook:

```powershell
pip install pre-commit
pre-commit install
```

## What Pre-Commit Does

Pre-commit executes the configured hooks (formatting, linting, security, etc.) defined in `.pre-commit-config.yaml` before each commit. If any hook fails, the commit is blocked until issues are fixed.

### Automatic Run on Commit

After installation the hooks run every time you commit. No extra action required.

### Run Hooks Manually (Full Scan)

Execute all hooks against the entire codebase (not just staged changes):

```powershell
task pre-commit
```

Use this before larger refactors or prior to opening a PR.

### Update Hook Versions

Refresh all hook repositories to their latest declared revisions:

```powershell
task pre-commit:update
```

Run periodically to stay current with upstream improvements.

## Common Troubleshooting

| Issue                         | Cause                             | Fix                                                                        |
| ----------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| Hook keeps reformatting files | Formatter config mismatch         | Re-run commit; accept formatting or adjust config in the hook repo section |
| Long execution time           | Full scan on large changes        | Prefer incremental commits; use manual full scan sparingly                 |
| Python package import errors  | Missing local virtual environment | Create and activate venv before running hooks                              |

## Recommended Workflow

1. Make changes
1. Run `task pre-commit` (optional full scan)
1. Stage & commit (hooks auto-run)
1. Fix any reported issues and recommit

## Next Steps

Set up additional FinFlow tooling (n8n workflows, parser modules) after ensuring pre-commit is working consistently.
