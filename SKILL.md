---
name: architect-thinker
description: Ensures complete understanding of project context before any modification. Use when starting work on an unfamiliar codebase, planning a large refactor, or before modifying any shared/core logic. Based on Claude Code's verified perception logic from the 2026/03/31 source leak.
---

# Core Perception Protocol

## Step 1: Zero-Assumption Exploration

- **No modification without confirmation**: Never generate code changes without first verifying file paths and actual file content.
- **Bounded directory scan**: Use `find . -maxdepth 3 -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/__pycache__/*'` to map the project tree. Avoid `ls -R` on large repos — it produces unmanageable output.
- **Tech stack identification**: Read the relevant dependency manifest before touching any logic:
  - JS/TS: `package.json`
  - Python: `requirements.txt`, `pyproject.toml`, or `Pipfile`
  - Go: `go.mod`
  - Java/Kotlin: `pom.xml` or `build.gradle`

## Step 2: Reference Tracing

- **Call-site search before modification**: Use `grep -rn "functionName" --include="*.ts"` (adjust extension) to find all usages of a function before modifying it.
- **Side-effect analysis**: Before writing any change, produce an explicit "Analysis" section in your response that lists potential side effects and affected call sites.

## Step 3: Trade-off Decision

- **Two-solution minimum**: For any non-trivial problem, produce at least two candidate solutions (e.g., performance-focused vs. readability-focused).
- **Decision record**: State which solution was chosen and why, inline in the response, before writing the code.
- **Minimal footprint**: Per Claude Code's core philosophy — don't add features, refactor, or "improve" beyond what was explicitly asked. A bug fix doesn't need surrounding code cleaned up.
