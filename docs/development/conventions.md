# Development Conventions

## Language

- **English only**: code, commits, comments, API documentation, test names, file names.
- Variable names, function names, class names: English.

## Commits

Format: `type(scope): short imperative description`

| Type | When |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `test` | Adding or updating tests |
| `chore` | Tooling, config, scaffolding |
| `docs` | Documentation only |
| `refactor` | No behavior change |
| `perf` | Performance improvement |

Examples:
```
feat(auth): add JWT refresh token endpoint
test(tweets): verify cursor pagination returns distinct pages
fix(users): compute followers_count from subquery instead of hardcoded 0
chore: scaffold backend and frontend with Docker Compose
```

## Backend (Python / FastAPI)

- **No hardcoded values**: every configurable value via `app/core/config.py` (`pydantic-settings`).
- **Layer discipline**: domain never imports from infrastructure. Handlers never import repos directly.
- **One use case per file**: `application/use_cases/<domain>/<action>.py`.
- **Mappers**: model→entity conversion only in `infrastructure/database/repositories/mappers.py`.
- **DI**: all dependencies via `Depends()`. No direct instantiation inside handlers.
- **Error handling**: domain exceptions raised in use cases; caught in FastAPI exception handlers.
- **Async everywhere**: all DB operations use `async with session` via `sqlalchemy.ext.asyncio`.
- **Tests alongside features**: unit tests in `tests/unit/`, integration tests in `tests/integration/`.
- **Test DB**: async SQLite in-memory via `conftest.py`. Never hit a real PostgreSQL in tests.

## Frontend (TypeScript / Next.js)

- **No hardcoded values**: all env vars validated via `src/config/env.ts` (Zod).
- **Feature isolation**: each feature in `src/features/<name>/` with its own `api/`, `components/`, `hooks/`, `schemas/`.
- **shadcn/ui**: never edit files in `src/components/ui/`. Compose in `src/components/shared/`.
- **React Query for server state**: no `useState` for async data. Key factory in `src/config/query-keys.ts`.
- **Zustand for UI state only**: sidebar and theme. Not for server data.
- **Forms**: React Hook Form + Zod resolver. No uncontrolled inputs.
- **Imports**: use path aliases (`@/`) not relative paths (`../../`).
- **No `any`**: TypeScript strict mode. Type everything.
- **Optimistic updates**: mutations update cache immediately with rollback on error.

## File Naming

- Python: `snake_case.py`
- TypeScript: `kebab-case.tsx`, `kebab-case.ts`
- No `index.ts` barrel files unless explicitly needed for public API of a package.

## Environment Variables

- Backend: declared in `backend/app/core/config.py` as `pydantic-settings` fields.
- Frontend: declared in `frontend/src/config/env.ts` via `z.object({...}).parse(process.env)`.
- All `.env` files are gitignored. Only `.env.example` is committed.
- Docker Compose root `.env` only contains infra variables (ports, DB credentials).

## Testing

- **Backend**: `pytest --cov=app` must report ≥ 85%.
- **Frontend unit**: Vitest + MSW for API mocking.
- **E2E**: Playwright against the running Docker stack.
- **Never defer tests**: each phase commits its tests alongside the feature code.
