# The Flock — Claude Workflow

## Project

Full-stack Twitter clone challenge. Backend: FastAPI + Clean Architecture. Frontend: Next.js 15 + Feature-based. Evaluated by an auditor on process quality (SDD cycle) and code quality.

## Workflow (SDD Cycle)

```
propose → apply → review → archive
```

All work lives inside `openspec/changes/01-twitter-clone/`. Artifacts were created before any product code (FASE 0). Implementation proceeds phase by phase per `tasks.md`.

## Rules

- **One phase at a time.** Complete, test, and commit before moving to the next.
- **Tests alongside features.** Never defer tests to end.
- **No hardcoded values.** Every configurable value lives in env vars via `backend/app/core/config.py` or `frontend/src/config/env.ts`.
- **English only** for all technical artifacts: code, commits, comments, API docs.
- **Never push to main.** All commits go to `dev`.

## Commits

Format: `type(scope): description`

Types: `feat`, `fix`, `test`, `chore`, `docs`, `refactor`, `perf`

Use the `/commit` skill for structured commits and PRs.

## Skills

- `/commit` — stage, commit, push, open PR
- `/code-auditing` — adversarial review (spec vs code, anti-patterns, dead code)

Skills are in `.claude/skills/`.

## Reference

- Solution: `/Users/gustavobenitez/Documents/development/labs/agentic-twitter-clone/`
- Challenge plan: `ignore/plans/challenge-plan.md`
- Challenge spec: `ignore/TwitterClone_Challenge_TheFlock.docx`
- OpenSpec change: `openspec/changes/01-twitter-clone/`

## RTK

Prefix all shell commands with `rtk` for token savings:
```bash
rtk git status
rtk pytest
rtk tsc
```
