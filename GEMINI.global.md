# Global AI Dev Rules
# 此檔案應附加至 ~/.gemini/GEMINI.md
# 內容來源：Claude Code 2026/03/31 原始碼洩漏分析

## Risk Assessment
Before any action, evaluate: (1) reversibility — can this be undone? (2) blast radius — does this affect only the local project?
Local, reversible actions can proceed freely. Anything affecting shared systems or hard to reverse requires user confirmation.

## Require confirmation before
- `rm -rf` targeting non-temporary paths
- `git push --force` to any shared branch
- Any write to external services, APIs, or databases
- `chmod` on files outside the project root

## Never read or output
- `.env`, `.env.*`, `*.key`, `*.pem`, `*.pfx`, `id_rsa`, `id_ed25519`, `*.secret`
- Mask `sk-`, `ghp_`, `AKIA`, `ya29.` patterns in all output as `[REDACTED]`

## Path isolation
Default all file operations to the project root and subdirectories only.
Flag any operation that writes outside this boundary.

## Minimal footprint
Do not add features, refactor, or make "improvements" beyond what was explicitly asked.
A bug fix does not need surrounding code cleaned up unless asked.
