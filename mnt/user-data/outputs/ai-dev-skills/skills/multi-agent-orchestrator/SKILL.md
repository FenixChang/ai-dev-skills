---
name: multi-agent-orchestrator
description: Task decomposition and multi-role verification for large or complex engineering tasks. Use when a task involves 3+ file edits, backend/API changes, or infrastructure changes. Works in both true multi-agent environments (Antigravity Manager, Claude Code) and single-agent tools (Gemini CLI) via sequential role simulation.
---

# Orchestration Protocol

## Task Decomposition

Break large requirements into three ordered layers before writing any code:

1. **Infrastructure layer**: Data models, database schemas, API contracts, configuration files.
2. **Logic layer**: Business logic, algorithms, service implementations.
3. **Interface layer**: UI components, CLI interfaces, output formatting, API consumers.

**Dependency rule**: Always define the contract (API shape, data model) before implementing consumers. The interface layer must never be started before the logic layer contract is stable.

## Sequential Role Simulation

In single-agent tools (Gemini CLI, etc.), simulate the following roles sequentially. Do not skip phases.

### Phase 1 — Coder
Write the primary implementation. Focus only on making it correct and working. Do not optimise prematurely.

### Phase 2 — Tester
Write unit tests for the Phase 1 implementation **without modifying the implementation**. Tests should cover:
- Happy path
- Edge cases
- At least one failure/error case

### Phase 3 — Reviewer
Audit the implementation against:
- The original requirements (did we build what was asked?)
- Security standards (does it follow the `security-guardrail` rules?)
- Code quality (is it the minimal, readable solution?)

Explicitly output: `Reviewer findings: [list of issues] / None found.`

## Mandatory Verification Trigger

If the task involves **any** of the following, a verification pass is required before reporting completion:
- 3 or more file edits
- Backend or API changes
- Infrastructure or configuration changes
- Database migrations

Verification output format:
```
VERIFICATION PASS
Changes made: [list of files and what changed]
Tests status: [pass / fail / none available]
Reviewer findings: [issues found / none]
Confidence: [high / medium / low — explain if not high]
```

## True Multi-Agent Mode (Antigravity / Claude Code)

When the tool supports parallel sub-agents:

- Use **background/fork mode** for long-running tasks that should not block the main context.
- Use **Explore sub-agent** for broad codebase research instead of running Glob/Grep in the main context.
- Use **Verification sub-agent** after any non-trivial implementation (same threshold as above).
- **Final integration**: The orchestrating agent must run a full integration test after all sub-agent outputs are merged. Never report completion before this step.
