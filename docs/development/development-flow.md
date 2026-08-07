# Development Flow — The Flock

Phase-by-phase workflow for building the challenge.

---

## Source of Truth

**`openspec/changes/01-flock-tweet-bootstrap/`** is the operational source of truth.
- `proposal.md` — why and what is being built
- `design.md` — how it is technically designed
- `tasks.md` — executable checklist of all phases (marked ✅ on completion)

---

## Per-Phase Flow

```
dev (empty)
 │
 └─ 1. Create branch from dev
 │     git checkout dev
 │     git checkout -b <branch>
 │
 └─ 2. (Optional) Create isolated worktree
 │     /using-git-worktrees
 │
 └─ 3. Implement the phase
 │     Follow tasks.md in order
 │     Code + tests in the same commit
 │
 └─ 4. Commit on the branch
 │     /commit
 │     Format: type(scope): description
 │
 └─ 5. PR to dev
 │     gh pr create --base dev
 │
 └─ 6. Merge to dev
 │     PR merged → dev advances
 │
 └─ 7. Mark task in tasks.md as ✅
       Next phase: back to step 1
```

---

## Non-Negotiable Rules

1. **Never commit directly to `dev` or `main`** — always via branch + PR.
2. **Tests in the same commit as the code** — never at the end.
3. **No hardcoded values** — everything via env vars.
4. **One phase at a time** — complete, test, and merge before starting the next.

---

## Branch Naming

```
feat/<scope>     new functionality
test/<scope>     tests only
chore/<scope>    scaffolding, configuration
docs/<scope>     documentation
sdd/<scope>      SDD foundation artifacts
```

Full branch plan: `ignore/plans/branching-strategy.md` (non-versioned, local only).

---

## Worktrees

Worktrees live in `.claude/temp/worktrees/` (git-ignored).
Use the `/using-git-worktrees` skill to create and clean up.

---

## SDD Archive Cycle (at project end)

When all phases are complete:

1. Run `/code-auditing` — adversarial review
2. Fix identified gaps
3. Confirm `pytest --cov=app` ≥ 85%
4. Archive: move `openspec/changes/01-flock-tweet-bootstrap/` → `openspec/changes/archive/YYYY-MM-DD-flock-tweet-bootstrap/`
5. Create `openspec/specs/` with final product specs

See the last phase in `openspec/changes/01-flock-tweet-bootstrap/tasks.md`.

---

## Available Skills

| Skill | When to use |
|---|---|
| `/commit` | Create commit and PR with correct format |
| `/code-auditing` | Adversarial review at the end |
| `/using-git-worktrees` | Create isolated workspace before a phase |
| `/enrich-us` | Enrich a user story or REQ item with full technical detail |
