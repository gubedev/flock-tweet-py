# Spec: Architecture

**Status**: specified
**Phase**: All phases
**Created**: 2026-08-04

---

## Purpose

Define the structural rules that govern every backend and frontend file in the project. These rules are non-negotiable — violations are treated as bugs, not style preferences.

---

## REQ-01: Backend — Clean Architecture Layers

The backend is divided into four layers with a strict dependency direction:

```
presentation  →  application  →  domain  ←  infrastructure
```

- `domain/` has zero imports from FastAPI, SQLAlchemy, or any other external library.
- `application/` imports only from `domain/`. No DB sessions, no HTTP concepts.
- `infrastructure/` imports from `domain/` to implement its ABCs. Never imports from `application/` or `presentation/`.
- `presentation/` imports from `application/` (use cases) and `infrastructure/` only via `Depends()`.

**Scenario REQ-01-A: Domain purity check**
- GIVEN any file in `app/domain/`
- WHEN its imports are inspected
- THEN no import from `fastapi`, `sqlalchemy`, `httpx`, or any third-party library appears

**Scenario REQ-01-B: Application layer purity check**
- GIVEN any file in `app/application/`
- WHEN its imports are inspected
- THEN no import from `sqlalchemy`, no `Session` or `AsyncSession` in method signatures

---

## REQ-02: Use Case Structure

Each business action has exactly one use case class in `app/application/use_cases/<domain>/`.

```
auth/           register_user, login_user, refresh_token, logout_user, get_current_user
tweets/         create_tweet, delete_tweet, get_timeline, get_thread, create_reply
users/          get_profile, update_profile, upload_avatar
follows/        follow_user, unfollow_user
likes/          like_tweet, unlike_tweet
search/         search_users
notifications/  get_notifications, mark_notification_read, mark_all_notifications_read
```

Each use case:
- Receives repository ABCs and services as constructor arguments.
- Has a single public method (e.g., `execute()`).
- Never instantiates infrastructure classes directly.

---

## REQ-03: Dependency Injection via Depends()

All infrastructure dependencies are wired in `app/presentation/dependencies/`. No direct instantiation of repositories or services in routers.

```python
# Correct
async def create_tweet(
    use_case: CreateTweetUseCase = Depends(get_create_tweet_use_case),
    ...
)

# Forbidden
async def create_tweet(...):
    repo = SqlTweetRepository(db)  # ← violates DI rule
```

**Scenario REQ-03-A: GET /auth/me layer compliance**
- GIVEN the `GET /api/auth/me` handler
- WHEN its dependency chain is traced
- THEN it goes through `GetCurrentUserUseCase`, NOT a direct injection of `SqlUserRepository`

---

## REQ-04: mappers.py — Centralized Model→Entity Conversion

All SQLAlchemy model → domain entity conversions live in one file:

```
app/infrastructure/database/repositories/mappers.py
```

Functions:
- `user_model_to_entity(m, followers_count, following_count, viewer_is_following) → User`
- `tweet_model_to_entity(m, viewer_has_liked, author) → Tweet`
- `notification_model_to_entity(m, actor_username, actor_avatar_url) → Notification`

No `_to_entity()` methods inside individual repository classes.

---

## REQ-05: Frontend — Feature-based Architecture

```
src/
├── app/                  Next.js App Router (pages + layouts only)
│   ├── (auth)/           Public routes: /login, /register
│   └── (main)/           Protected routes (auth guard in layout.tsx)
├── features/             Self-contained modules (NO cross-feature imports)
│   ├── auth/             api/, components/, hooks/, schemas/
│   ├── tweets/           api/, components/, hooks/
│   ├── users/            api/, components/, hooks/
│   ├── search/           api/, components/, hooks/
│   └── notifications/    api/, components/, hooks/
├── components/
│   ├── ui/               shadcn/ui only — never edit these files
│   ├── shared/           Reusable atoms: avatar, infinite-scroll, character-counter
│   └── layout/           sidebar, mobile-nav, main
├── config/               env.ts (Zod), paths.ts, query-keys.ts
├── lib/                  api-client.ts, utils.ts
├── stores/               ui-store.ts (Zustand: sidebar + theme only)
└── styles/               globals.css, theme.css (CSS variables)
```

**Scenario REQ-05-A: No cross-feature imports**
- GIVEN `features/tweets/` and `features/users/`
- WHEN `features/tweets/` needs user data
- THEN it calls the API (via `users-api.ts`) or uses a shared hook — it does NOT import directly from `features/users/`

---

## REQ-06: API Client — No localStorage

The access token lives in a JavaScript module-level variable (memory). It is never stored in `localStorage` or `sessionStorage`.

```typescript
let accessToken: string | null = null  // module scope, cleared on page unload
```

On page reload, the token is re-acquired via `POST /api/auth/refresh` (which reads the httpOnly cookie). The API client sends all requests with `credentials: "include"` so the refresh cookie is always transmitted.

---

## REQ-07: Rate Limiting — Proxy-Aware

All public endpoints are protected by a rate limiter. Behind a reverse proxy or Docker network, `request.client.host` resolves to the proxy IP, not the real client. The limiter must read `X-Forwarded-For` when present.

```python
def get_client_ip(request: Request) -> str:
    forwarded = request.headers.get("X-Forwarded-For")
    if forwarded:
        return forwarded.split(",")[0].strip()
    return request.client.host

limiter = Limiter(key_func=get_client_ip)
app.state.limiter = limiter
```

**Scenario REQ-07-A: Rate limit enforced**
- GIVEN a client that exceeds the configured request limit (default: 100 req/min per IP)
- WHEN another request arrives
- THEN 429 Too Many Requests is returned

**Scenario REQ-07-B: Proxy header used**
- GIVEN the request carries `X-Forwarded-For: 1.2.3.4, 10.0.0.1`
- WHEN the limiter extracts the client IP
- THEN `1.2.3.4` is used (first IP in chain), not `10.0.0.1` (proxy)

