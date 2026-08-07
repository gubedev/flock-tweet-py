---
description: Development rules and workflow for The Flock challenge project.
alwaysApply: true
---

## Core Principles

- **One phase at a time**: Complete, test, and commit each phase before starting the next.
- **Test alongside features**: Tests are committed in the same phase as the feature — never deferred.
- **No hardcoded values**: Every configurable value lives in env vars (`core/config.py` or `config/env.ts`).
- **English only**: All technical artifacts use English — code, commits, comments, API docs, test names.

## Branch Protection (ABSOLUTE — never override)

- **Never commit directly to `dev` or `main`** — all work goes through feature branches and PRs.
- **PRs always target `dev`** — never open a PR against `main`.
- **`main` is untouchable** — no direct commits, no force pushes, no merges from CLI.

## Standards

- [Architecture](./docs/architecture/architecture.md)
- [Conventions](./docs/development/conventions.md) — commit format, layer rules, naming, testing rules
- [Development Flow](./docs/development/development-flow.md) — per-phase workflow, branching rules, SDD cycle

## Skills

Skills live in `.claude/skills/`. When a request matches a skill, load and follow the corresponding `skill.md` automatically.

Available skills:
- `commit` — stage, commit, push, open PR following project standards
- `code-auditing` — adversarial review: spec vs code, anti-patterns, dead code, coverage check
- `using-git-worktrees` — create an isolated git worktree before starting a phase
- `enrich-us` — enrich a user story or REQ item with full technical detail (endpoint, fields, scenarios, DoD)

## Memory

Persistent memory lives in `.claude/memory/`. Always read and write memory files there.
The index is `.claude/memory/MEMORY.md`.

## RTK

Always prefix shell commands with `rtk`. See [`.claude/rtk-reference.md`](./.claude/rtk-reference.md) for the full command catalog.
