---
name: commit
description: Create focused commits and pull requests following repository standards.
version: 1.0.0
---

# commit Skill

## Instructions

# Role

You are an expert in version control. You create clear, comprehensive commits and Pull Requests aligned with project standards.

# Arguments

**Optional.** `$ARGUMENTS` may contain:

- **Nothing**: Stage and commit all relevant changes, then open a PR.
- **Feature/scope identifiers**: Stage and commit only the changes belonging to that scope.
- **No-git mode**: If user says "only message", "dry run", "no PR" — output the staging plan and commit message only, no git commands.

# Process

## 1. Inspect state
- `git status` + `git diff` to list all changes.
- Identify current branch. If not on a feature branch, warn.

## 2. Resolve scope
- No arguments → stage all relevant changes (exclude `.env`, build artifacts).
- With arguments → stage only changes tied to that scope; leave others unstaged.

## 3. Commit message
- **English only** (per `docs/development/conventions.md`).
- Format: `type(scope): short imperative description`
- Types: `feat`, `fix`, `test`, `chore`, `docs`, `refactor`, `perf`
- Body (if needed): bullet points explaining what and why.
- Never commit secrets, `.env`, or generated files.

## 4. Commit and push
- Create the commit.
- Push to remote with `-u` if branch is new.

## 5. Pull Request
- Use `gh pr create` with clear title and description.
- Target branch: `dev` (never `main`).

## 6. Summary
- Report files committed and PR URL.

# Notes
- Never `--force` push unless user explicitly requests it.
- Never skip hooks (`--no-verify`).
