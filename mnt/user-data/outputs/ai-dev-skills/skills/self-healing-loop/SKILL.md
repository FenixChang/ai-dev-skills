---
name: self-healing-loop
description: Implements automated testing and error self-healing loop with bounded recursion. Use when a project has an existing test suite and you want the agent to automatically verify and repair its own changes. Activate for projects with npm test, pytest, go test, or equivalent.
---

# Self-Healing Protocol

## The Loop

After every code modification, execute the following cycle:

1. **Run tests**: Execute the project's existing test suite:
   - JS/TS: `npm test` or `npx vitest run`
   - Python: `pytest` or `python -m pytest`
   - Go: `go test ./...`
   - Rust: `cargo test`

2. **Capture errors verbatim**: If tests fail, read the full output from `stderr` and the test report. Do not summarise — capture the exact error messages.

3. **Diagnose before fixing**: In one sentence, state the diagnosed root cause before writing any fix. Do not write code until the cause is clearly identified.

4. **Apply minimal fix**: Produce the smallest change that addresses the diagnosed cause only. Do not fix unrelated issues in the same pass.

5. **Recursive verification**: Repeat from step 1 until tests pass or the loop limit is reached.

## Loop Limits and Exit Conditions

- **Maximum iterations: 3**. After 3 failed repair attempts, stop and report.
- **Identical error exit**: If the exact same error appears across 2 consecutive loops, stop immediately. The fix strategy is wrong, not the code — further attempts will waste cycles.
- **On failure**: Output the full `stderr`/test report verbatim. List each attempted fix. Do NOT claim the issue is resolved.

## Hard Rules

- **Never modify test files** unless explicitly instructed. Failing tests are evidence about the implementation, not bugs in the tests.
- **Never fake a passing state** by commenting out assertions or skipping tests to make the suite green.

## Escalation

If all 3 loops fail, output:
```
SELF-HEALING FAILED after 3 attempts.
Root cause hypothesis: [your diagnosis]
Attempted fixes: [list]
Full error output: [verbatim stderr]
Recommended next step: [human review / different approach]
```
