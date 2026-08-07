---
name: code-auditing
description: Adversarial review — spec vs code, architecture compliance, dead code, security gaps, and coverage check.
version: 2.0.0
---

# Code Auditing Skill

Comprehensive adversarial review comparing the delivered code against the project specs, architecture rules, and challenge criteria.

## When to Use

- Before merging any phase branch (lightweight per-phase check)
- At the end of all implementation phases (full audit before SDD archive)
- When a layer violation or anti-pattern is suspected

---

## Phase 0: Setup

1. Read `openspec/specs/product/` — understand what each feature must do.
2. Read `openspec/specs/tech/architecture/spec.md` and `openspec/specs/tech/database/spec.md` — understand the structural rules.
3. Read `openspec/changes/*/proposal.md` and `design.md` for the active change — understand intent and key design decisions.
4. Run tests as baseline:
   ```bash
   rtk pytest --cov=app --cov-report=term-missing
   rtk vitest run
   ```

---

## Phase 1: Spec vs Code

For each REQ item in `openspec/specs/product/`:
- Verify the scenario's happy path is implemented.
- Verify each error scenario has a corresponding test.
- Flag: REQ with no implementation, REQ with no test, test with no REQ.

---

## Phase 2: Architecture Compliance

Verify the rules in `openspec/specs/tech/architecture/spec.md` are enforced:

- **Domain purity**: no third-party imports in `app/domain/`.
- **Application purity**: no SQLAlchemy sessions or FastAPI imports in `app/application/`.
- **DI contract**: no repository instantiation inside routers; all wired via `Depends()`.
- **mappers.py**: no `_to_entity()` methods inside individual repositories.
- **No localStorage**: access token never written to `localStorage` or `sessionStorage`.
- **Feature isolation**: no cross-feature imports in `frontend/src/features/`.

Check each design decision in `design.md` — verify each is reflected in the code.

---

## Phase 3: File-by-File Analysis

For each source file, check:

**Dead code**
- Unused imports, functions, variables.
- Tools: `deadcode . --dry` (Python), `npx knip` (TypeScript).
- Verify findings before reporting (dynamic imports, re-exports, entry points).

**Anti-patterns**
- N+1 queries (any loop that issues a DB call per iteration).
- Hardcoded values (strings, numbers, URLs that should be env vars).
- Direct repository access inside routers (bypasses use case layer).

**Security**
- Unvalidated user input reaching DB or filesystem.
- Missing auth guards on protected endpoints.
- Sensitive data in logs or error responses.
- Rate limiting present on auth endpoints.

**Async correctness**
- Missing `await` on async calls.
- Unhandled promise rejections (frontend).
- SSE `EventSource` closed on unmount.

---

## Phase 4: Coverage Check

```bash
rtk pytest --cov=app --cov-report=term-missing
```

- Target: ≥ 85% backend coverage.
- List uncovered lines that represent untested business logic (not boilerplate).
- Frontend: verify unit tests cover optimistic update + rollback paths.

---

## Phase 5: Report

**Verdict:**
- **PASS** — all specs covered, no critical issues, coverage ≥ 85%.
- **PASS WITH GAPS** — minor issues only, no blockers; list gaps with priority.
- **FAIL** — critical issue or coverage < 85%; list what must be fixed before archiving.

**Issue priority:**
- **Critical** — broken functionality, security vulnerability, spec requirement not implemented.
- **High** — layer violation, N+1 query, missing auth guard.
- **Medium** — code quality, missing edge case test, hardcoded value.
- **Low** — style, minor improvement, dead import.
