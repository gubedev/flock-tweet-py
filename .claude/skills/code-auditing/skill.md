---
name: code-auditing
description: Adversarial review — spec vs code, anti-patterns, dead code, security gaps.
version: 1.0.0
---

# Code Auditing Skill

Use after completing all implementation phases (FASE 18 — SDD Archive). Compares the delivered code against the openspec specs and the challenge criteria.

## Audit Phases

### Phase 0: Setup
1. Read `openspec/changes/01-twitter-clone/proposal.md` and `design.md` — understand the spec.
2. Read `ignore/plans/challenge-plan.md` — understand the improvement targets over v2.
3. Run existing tests as baseline: `pytest --cov=app` and `npm test`.

### Phase 1: Spec vs Code
For each capability in `proposal.md`:
- Verify it is implemented in the code.
- Verify it is covered by at least one test.
- Flag any spec capability with no implementation or no test.

### Phase 2: v2 Bug Fixes Verification
Verify each improvement listed in `challenge-plan.md` was actually fixed:
- [ ] Reply notifications dispatched from `CreateTweetUseCase`
- [ ] `followers_count` computed via subquery (not hardcoded 0)
- [ ] `entrypoint.sh` has no `--reload` in production
- [ ] Rate limiter uses `X-Forwarded-For`
- [ ] No access token in `localStorage`
- [ ] Password min 8 chars validated in schema
- [ ] `GET /auth/me` uses `GetCurrentUserUseCase`
- [ ] `LocalStorage` injected via `Depends()`

### Phase 3: File-by-File Analysis
For each file, check:
- Dead code (unused functions, imports, variables)
- Anti-patterns (N+1 queries, hardcoded values, direct repo access in handlers)
- Security issues (unvalidated input, missing auth checks, XSS surfaces)
- Layer violations (domain importing infrastructure, handlers bypassing use cases)

### Phase 4: Coverage Check
- `pytest --cov=app --cov-report=term-missing` → must be ≥ 85%.
- List any uncovered lines that represent untested business logic.

### Phase 5: Report

Verdict options:
- **PASS** — all specs covered, no critical issues, coverage ≥ 85%.
- **PASS WITH GAPS** — minor issues, no blockers, list gaps.
- **FAIL** — critical issues or coverage < 85%. List what must be fixed before archiving.

## Issue Priority
- **Critical** — broken functionality, security vulnerability, spec not implemented
- **High** — performance issue, layer violation
- **Medium** — code quality, missing test coverage
- **Low** — style, minor improvements
