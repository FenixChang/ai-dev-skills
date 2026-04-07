---
name: atomic-patcher
description: Implements precise patching logic to avoid large-scale rewriting. Use when editing existing files, fixing bugs, or making targeted changes to production code. Based on Claude Code's verified string-replacement editing strategy from the 2026/03/31 source leak.
---

# Precision Patching

## Core Principle: Read Before Edit

Always read the target file immediately before editing. Stale context causes string-match failures — if the file has changed since it was last read, the replacement will silently fail or produce incorrect results.

## Modification Guidelines

- **Minimum change principle**: Use exact string replacement targeting only the lines that need to change. Never rewrite an entire file to change a few lines.
- **Context preservation**: Retain at least 3 lines of original code above and below the modified section to anchor the replacement correctly.
- **One concern per edit**: Each edit should address exactly one logical change. If multiple changes are needed, apply them sequentially with verification between each.

## Pre-flight Verification (check BEFORE editing, not after)

Run the appropriate syntax checker for the language **before** generating edits, to confirm the baseline is clean:

| Language | Syntax Check | Auto-format (separate, optional step) |
|---|---|---|
| JS/TS | `eslint src/` or `tsc --noEmit` | `prettier --write` |
| Python | `ruff check .` | `ruff format .` |
| Go | `go vet ./...` | `go fmt ./...` |
| Rust | `cargo check` | `cargo fmt` |

**Important**: `eslint --fix`, `ruff format`, and `go fmt` are auto-fixers, not validators. Always run the check command first. Only run auto-format as a separate, explicit step after the check passes.

## Post-edit Confirmation

For compiled languages (Go, Rust, Java, C++), confirm the modified file passes compilation before reporting success:
- Go: `go build ./...`
- Rust: `cargo build`
- TypeScript: `tsc --noEmit`
