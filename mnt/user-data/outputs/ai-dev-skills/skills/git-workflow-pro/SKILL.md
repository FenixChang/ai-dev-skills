---
name: git-workflow-pro
description: Git workflow standards for commits, staging verification, and PR creation. Use when committing code, creating pull requests, or reviewing staged changes. Based on Claude Code's verified git operation logic from the 2026/03/31 source leak.
---

# Git Standards

## Commit Logic

- **Semantic commits**: Follow [Conventional Commits](https://conventionalcommits.org) format:
  - `feat:` — new feature
  - `fix:` — bug fix
  - `refactor:` — code restructuring without behaviour change
  - `docs:` — documentation only
  - `test:` — adding or fixing tests
  - `chore:` — build process, dependency updates

- **Atomic scope**: One commit = one logical change. If a task touches unrelated concerns, split into separate commits before pushing.

## Pre-Commit Checks (run in parallel)

Run both commands simultaneously before staging anything:

```bash
git status          # identify all untracked and modified files
git diff --staged   # review exactly what is currently staged
```

Then verify:
1. **Scope check**: Every staged file must be related to the current task. Unstage unrelated files with `git restore --staged <file>`.
2. **Debug artifact check**: Confirm no `console.log`, `print()`, `debugger`, or `TODO` markers were accidentally staged.
3. **Message from diff**: Write the commit message based on `git diff --staged` output, not from memory.

## Pre-Push Checks

- For shared branches: run `git pull --rebase` before pushing to avoid merge noise.
- For force-push (`git push --force`): always confirm with the user first. Prefer `--force-with-lease` which fails safely if the remote has new commits.

## PR Creation Checklist

When creating a pull request:
1. Run `git diff main...HEAD` (or relevant base branch) to review all changes since divergence.
2. Confirm the PR title follows Conventional Commits format.
3. PR body should include: what changed, why, and how to test it.
4. Check for unintended file inclusions (lock files, build artifacts, IDE configs).
