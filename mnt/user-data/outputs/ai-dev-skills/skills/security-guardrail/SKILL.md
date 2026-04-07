---
name: security-guardrail
description: Core security guardrail based on reversibility and blast-radius risk assessment. Use for all file, shell, and external service operations. Should be loaded globally (not just on-demand) — place in GEMINI.md for Gemini CLI, or configure via Deny List in Antigravity.
---

# Safety Protocol

## Risk Assessment Framework

Before any action, evaluate two dimensions:

| Dimension | Question | Low risk | High risk |
|---|---|---|---|
| **Reversibility** | Can this be undone easily? | Edit a file, run tests | `rm` without backup, DB migration |
| **Blast radius** | Does this affect only local project? | Local file edits | External APIs, shared branches, system paths |

Local + reversible actions → proceed freely.
Anything else → confirm with the user first.

## Require Explicit Confirmation Before

- `rm -rf` targeting any non-temporary directory (anything outside `/tmp` or OS temp dirs)
- `git push --force` to any shared branch
- `chmod` on files **outside** the project root directory
- Writing to external services (REST APIs, databases, cloud storage, email)
- Running scripts sourced from unknown or untrusted origins
- Any operation that writes outside the project root

## Data Exclusion — Never Read or Output

The following files must never be read, printed, or included in any output:

```
.env
.env.*
*.key
*.pem
*.pfx
*.p12
id_rsa
id_ed25519
*.secret
*credentials*
```

If any of the following patterns appear in output, mask them with `[REDACTED]`:
- `sk-...` (OpenAI / Anthropic API keys)
- `ghp_...` (GitHub Personal Access Tokens)
- `AKIA...` (AWS Access Key IDs)
- `ya29....` (Google OAuth tokens)
- Any string that looks like `Bearer <token>` in headers

## Path Isolation

- Default all read/write operations to project root and its subdirectories.
- Flag any operation that would write outside this boundary and ask for confirmation.
- Never follow symlinks that resolve outside the project root without explicit user approval.

## Note for Antigravity Users

In Antigravity, this skill's file exclusion rules should be reinforced via the UI's **Deny List** with the patterns above. The Deny List operates at the permission layer (before the model sees the file), which is stronger than prompt-level instructions.
