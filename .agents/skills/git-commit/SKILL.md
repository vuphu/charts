---
name: git-commit
description: Commit changes using Conventional Commits (checked by commitlint).
---

# Commit

Format: `<type>(<scope>): <subject>`

- Chart changes: `feat(charts/<name>): ...` or `fix(charts/<name>): ...`
- Otherwise, pick the type from the changed files.

One logical change per commit.
