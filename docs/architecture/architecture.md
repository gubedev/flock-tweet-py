# Architecture — The Flock

## Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15 (App Router) + Tailwind v4 + shadcn/ui |
| Backend | Python 3.12 + FastAPI 0.115 |
| ORM | SQLAlchemy 2.0 async |
| Migrations | Alembic |
| Database | PostgreSQL 16 |
| Auth | JWT (access 15min body + refresh 7d httpOnly cookie) |
| Server state FE | TanStack Query v5 |
| UI state FE | Zustand |
| Real-time | Server-Sent Events (SSE) |
| Testing BE | pytest + pytest-asyncio + httpx.AsyncClient + pytest-cov |
| Testing FE | Vitest + @testing-library/react + MSW |
| E2E | Playwright |
| Infra | Docker Compose (postgres + backend + frontend) |

## Backend: Clean Architecture

```
HTTP Request
    │
    ▼
presentation/api/v1/         ← FastAPI routers (thin, no logic)
    │
    ▼
presentation/dependencies/   ← DI wiring (use cases, repos, auth)
    │
    ▼
application/use_cases/       ← Business logic (one class per use case)
    │
    ▼
domain/repositories/         ← ABCs (pure Python, no framework)
    ▲
    │
infrastructure/database/     ← SQLAlchemy implementations
```

### Layer Rules
- **Domain** (`domain/`): pure Python dataclasses and ABCs. Zero imports from FastAPI, SQLAlchemy, or any infrastructure.
- **Application** (`application/use_cases/`): orchestrates domain entities and repository ABCs. No direct DB access.
- **Infrastructure** (`infrastructure/`): implements domain ABCs using SQLAlchemy. Converts models to entities via `mappers.py`.
- **Presentation** (`presentation/`): FastAPI routers, Pydantic schemas, dependency injection. Calls use cases only.

### Request Flow Example
```
POST /api/tweets
  → presentation/api/v1/tweets.py
  → presentation/dependencies/use_cases.py  (inject CreateTweetUseCase)
  → application/use_cases/tweets/create_tweet.py
  → domain/repositories/tweet_repository.py  (ABC)
  ↑ infrastructure/database/repositories/sql_tweet_repository.py
  → PostgreSQL
```

### Dependency Injection
All dependencies wired via FastAPI `Depends()`:
- `get_db_session` → async SQLAlchemy session
- `get_current_user` → parses Bearer token → calls `GetCurrentUserUseCase`
- `get_storage` → returns `LocalStorage` instance (no direct instantiation in handlers)
- `get_<use_case>` → constructs use case with its repo dependencies

## Frontend: Feature-based Architecture

```
src/
├── app/              Next.js App Router (pages + layouts)
│   ├── (auth)/       Public routes: /login, /register
│   └── (main)/       Protected routes: auth guard in layout.tsx
├── features/         Self-contained feature modules
│   ├── auth/         api, components, hooks, schemas
│   ├── tweets/       api, components, hooks
│   ├── users/        api, components, hooks
│   ├── search/       api, components, hooks
│   └── notifications/api, components, hooks
├── components/
│   ├── ui/           shadcn/ui (never edit)
│   ├── shared/       Composable atoms (button, avatar, infinite-scroll)
│   └── layout/       sidebar, mobile-nav, main
├── config/           env.ts (Zod), paths.ts, query-keys.ts
├── lib/              api-client.ts, utils.ts
├── stores/           ui-store.ts (Zustand)
└── styles/           globals.css, theme.css (CSS variables)
```

### Key Patterns
- **API client** (`lib/api-client.ts`): single fetch wrapper with URL from env, auto-refresh on 401 (with `isRetry` flag to avoid loops), no token in `localStorage`.
- **React Query**: all server state. Key factory in `config/query-keys.ts`.
- **Zustand**: sidebar open state and theme toggle only.
- **Optimistic updates**: likes and follows update UI immediately, rollback on error.
- **SSE**: `use-sse.ts` opens `EventSource`, invalidates notification query on message, closes on unmount.

## Database Schema

```sql
users          id UUID PK, username UNIQUE, email UNIQUE, password_hash,
               display_name, bio, avatar_url, created_at, updated_at

tweets         id UUID PK, content VARCHAR(280), author_id FK,
               parent_id FK NULLABLE,  image_url NULLABLE,
               likes_count INT DEFAULT 0, replies_count INT DEFAULT 0,
               created_at

follows        PK(follower_id, following_id), created_at

likes          PK(user_id, tweet_id), created_at

notifications  id UUID PK, recipient_id FK, actor_id FK,
               type ENUM(follow, like, reply), tweet_id FK NULLABLE,
               read BOOL DEFAULT FALSE, created_at
```

## Architectural Decisions

### Cursor-based Pagination
Chosen over offset pagination because it is stable under concurrent insertions — a new tweet at position 1 does not shift all subsequent pages.

### SSE vs WebSockets
SSE is server-to-client only, which is sufficient for notification delivery. It requires no protocol upgrade, works through standard HTTP/2, and needs no additional library.

### JWT in httpOnly Cookie (refresh)
The refresh token lives in an httpOnly cookie to prevent XSS access. The access token lives in memory (not localStorage) and is renewed transparently via the refresh endpoint when it expires.

### `likes_count` / `replies_count` as Columns
These are maintained with atomic SQL UPDATEs on every like/unlike/reply. This avoids a COUNT subquery on every tweet read, which would add latency on large timelines.

### `followers_count` / `following_count` as Subqueries
These are computed at query time via scalar subqueries (not stored columns). Follow/unfollow operations don't need to update a separate counter, eliminating the risk of count drift.

### `mappers.py` — Centralized Model→Entity Conversion
All `_to_entity()` logic lives in `infrastructure/database/repositories/mappers.py`. This prevents duplication across repositories and makes the conversion testable in isolation.
