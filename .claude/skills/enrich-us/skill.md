---
name: enrich-us
description: Analyze and enhance a user story or REQ item with complete, implementation-ready technical detail aligned with the project specs and architecture.
version: 1.0.0
---

# enrich-us Skill

## Instructions

Please analyze and enrich the following: $ARGUMENTS

Follow these steps:

1. **Understand the input.** The user will provide either:
   - A raw user story ("As a user, I want to...")
   - A REQ item or scenario from `openspec/specs/product/`
   - A feature description in plain language

2. **Act as a product expert with technical knowledge** of The Flock's stack (FastAPI + Clean Architecture backend, Next.js 16 frontend, PostgreSQL).

3. **Validate completeness.** Decide whether the story is fully detailed for autonomous implementation. Check for:
   - Full functionality description (happy path + edge cases)
   - All fields affected (request body, response shape, DB columns)
   - Endpoint path and HTTP method (aligned with `openspec/specs/product/`)
   - Use case and layer(s) involved (domain / application / infrastructure / presentation)
   - BDD scenarios: GIVEN / WHEN / THEN with concrete values
   - Definition of done: what must pass for this to be considered complete
   - Error cases: 401, 403, 404, 409, 422 as applicable
   - Non-functional requirements: auth guard, rate limiting, atomicity, N+1 prevention

4. **If the story lacks detail**, produce an enriched version that is clearer, more specific, and concise. Use the project's technical context:
   - API design: `openspec/specs/product/` specs
   - Architecture rules: `openspec/specs/tech/architecture/spec.md`
   - DB patterns: `openspec/specs/tech/database/spec.md`
   - Technical design: `openspec/changes/01-flock-tweet-bootstrap/design.md`

5. **Output format** — always return both sections:

```
## Original
<exact input, unchanged>

## Enhanced
<enriched version with: description, endpoint, fields, scenarios, definition of done, error cases>
```

## Notes

- Do not invent features not present in `docs/product/prd.md`.
- If the input is already complete and implementation-ready, say so in `## Enhanced` and explain why no changes were needed.
- Keep language English throughout (per project conventions).
