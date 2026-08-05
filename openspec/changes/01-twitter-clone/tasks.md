# Tasks: Twitter Clone Challenge — The Flock

Each task must be completed, tested, and committed before moving to the next.
Tests are committed in the same task as the feature — never deferred.

---

## FASE 0 — SDD Foundation ✅
**Commit**: `chore: initialize project with SDD structure and challenge spec`

- [x] `.gitignore` — includes `ignore/`
- [x] `CLAUDE.md` — workflow rules
- [x] `docs/product/prd.md`
- [x] `docs/architecture/architecture.md`
- [x] `docs/development/conventions.md`
- [x] `openspec/changes/01-twitter-clone/proposal.md`
- [x] `openspec/changes/01-twitter-clone/design.md`
- [x] `openspec/changes/01-twitter-clone/tasks.md` (this file)
- [x] `.claude/skills/commit/skill.md`
- [x] `.claude/skills/code-auditing/skill.md`

---

## FASE 1 — Scaffolding
**Commits**:
- `chore: scaffold backend and frontend with Docker Compose`
- `test(backend): health endpoint smoke test`

**Backend**:
- [ ] `backend/requirements.txt`
- [ ] `backend/app/__init__.py`
- [ ] `backend/app/core/config.py` — `Settings` with pydantic-settings
- [ ] `backend/app/main.py` — FastAPI app, CORS, rate limiter, `GET /health`
- [ ] `backend/alembic.ini`
- [ ] `backend/alembic/env.py` — async config
- [ ] `backend/Dockerfile` — multi-stage, no `--reload`
- [ ] `backend/entrypoint.sh` — `alembic upgrade head` + uvicorn
- [ ] `backend/.env.example`
- [ ] `backend/.dockerignore`

**Frontend**:
- [ ] Next.js 15 init (App Router, TypeScript, Tailwind v4)
- [ ] shadcn/ui init
- [ ] `frontend/src/styles/theme.css` — CSS variables
- [ ] `frontend/src/config/env.ts` — Zod env validation
- [ ] `frontend/src/config/paths.ts` — type-safe routes
- [ ] `frontend/src/config/query-keys.ts`
- [ ] `frontend/src/lib/api-client.ts` — fetch wrapper, auto-refresh
- [ ] `frontend/src/lib/utils.ts` — `cn()`
- [ ] `frontend/Dockerfile` — multi-stage + `output: 'standalone'`
- [ ] `frontend/.env.example`
- [ ] `frontend/.dockerignore`
- [ ] `frontend/next.config.ts` — `output: 'standalone'`

**Raíz**:
- [ ] `docker-compose.yml` — postgres + backend + frontend, `restart: unless-stopped`, healthcheck
- [ ] `.env.example`
- [ ] `README.md` (skeleton)

**Test**:
- [ ] `backend/tests/integration/test_health.py` — `GET /health` → 200

---

## FASE 2 — DB Models + Migration
**Commits**:
- `feat(db): SQLAlchemy models and initial Alembic migration`
- `test(db): verify schema constraints and table creation`

- [ ] `backend/app/infrastructure/database/connection.py` — async engine + session factory
- [ ] `backend/app/infrastructure/database/models/__init__.py`
- [ ] `backend/app/infrastructure/database/models/base.py`
- [ ] `backend/app/infrastructure/database/models/user_model.py`
- [ ] `backend/app/infrastructure/database/models/tweet_model.py` — includes `likes_count`, `replies_count`
- [ ] `backend/app/infrastructure/database/models/follow_model.py` — composite PK
- [ ] `backend/app/infrastructure/database/models/like_model.py` — composite PK
- [ ] `backend/app/infrastructure/database/models/notification_model.py` — ENUM type
- [ ] `backend/alembic/versions/0001_initial_schema.py`
- [ ] `backend/app/infrastructure/database/repositories/mappers.py` — `user_model_to_entity()`, `tweet_model_to_entity()`

**Tests**:
- [ ] `backend/tests/conftest.py` — async SQLite in-memory, `async_client` fixture
- [ ] `backend/tests/integration/test_schema.py` — 5 tables exist, UNIQUE and composite PK constraints

---

## FASE 3 — Domain Layer
**Commits**:
- `feat(domain): entities, repository interfaces, domain exceptions`
- `test(domain): tweet and user entity validation rules`

- [ ] `backend/app/domain/entities/__init__.py`
- [ ] `backend/app/domain/entities/user.py` — dataclass
- [ ] `backend/app/domain/entities/tweet.py` — `__post_init__`: EmptyTweetError, TweetTooLongError
- [ ] `backend/app/domain/entities/follow.py`
- [ ] `backend/app/domain/entities/like.py`
- [ ] `backend/app/domain/entities/notification.py`
- [ ] `backend/app/domain/repositories/user_repository.py` — ABC
- [ ] `backend/app/domain/repositories/tweet_repository.py` — ABC
- [ ] `backend/app/domain/repositories/follow_repository.py` — ABC
- [ ] `backend/app/domain/repositories/like_repository.py` — ABC
- [ ] `backend/app/domain/repositories/notification_repository.py` — ABC
- [ ] `backend/app/domain/services/password_service.py` — ABC
- [ ] `backend/app/domain/exceptions.py`

**Tests**:
- [ ] `backend/tests/unit/test_tweet_entity.py` — empty content, too long, valid
- [ ] `backend/tests/unit/test_user_entity.py` — valid entity

---

## FASE 4 — Auth Backend
**Commits**:
- `feat(auth): JWT register, login, logout, refresh token`
- `test(auth): unit use cases + integration endpoints`

- [ ] `backend/app/infrastructure/security/jwt_service.py`
- [ ] `backend/app/infrastructure/database/repositories/sql_user_repository.py`
- [ ] `backend/app/infrastructure/security/password_service_impl.py` — bcrypt
- [ ] `backend/app/application/use_cases/auth/register_user.py`
- [ ] `backend/app/application/use_cases/auth/login_user.py`
- [ ] `backend/app/application/use_cases/auth/refresh_token.py`
- [ ] `backend/app/application/use_cases/auth/logout_user.py`
- [ ] `backend/app/application/use_cases/auth/get_current_user.py`
- [ ] `backend/app/presentation/schemas/auth.py` — password `min_length=8`
- [ ] `backend/app/presentation/schemas/user.py`
- [ ] `backend/app/presentation/dependencies/auth.py` — `get_current_user` dependency
- [ ] `backend/app/presentation/dependencies/use_cases.py`
- [ ] `backend/app/presentation/api/v1/auth.py`
- [ ] `backend/app/presentation/api/v1/router.py`

**Tests**:
- [ ] `backend/tests/unit/test_register_user.py` — duplicate email, duplicate username, password hashed
- [ ] `backend/tests/integration/test_auth.py` — register, login, /me, refresh, logout (9 cases)

---

## FASE 5 — Users + Follows Backend
**Commits**:
- `feat(users): profile endpoints, follow/unfollow with accurate counts`
- `test(users): unit follow rules + integration profile and follow endpoints`

- [ ] `backend/app/infrastructure/database/repositories/sql_user_repository.py` — subquery for followers/following counts
- [ ] `backend/app/infrastructure/database/repositories/sql_follow_repository.py` — cursor pagination
- [ ] `backend/app/application/use_cases/users/get_profile.py`
- [ ] `backend/app/application/use_cases/users/update_profile.py`
- [ ] `backend/app/application/use_cases/follows/follow_user.py` — dispatches follow notification
- [ ] `backend/app/application/use_cases/follows/unfollow_user.py`
- [ ] `backend/app/presentation/api/v1/users.py`
- [ ] `backend/app/presentation/api/v1/follows.py`

**Tests**:
- [ ] `backend/tests/unit/test_follow_user.py` — self-follow, double-follow, unfollow without following
- [ ] `backend/tests/integration/test_users.py` — profile with correct counts, update, 404
- [ ] `backend/tests/integration/test_follows.py` — follow, unfollow, followers list (cursor), count correctness

---

## FASE 6 — Tweets Backend
**Commits**:
- `feat(tweets): create, delete, timeline with cursor pagination, reply threads`
- `test(tweets): unit ownership + integration timeline pagination and replies`

- [ ] `backend/app/infrastructure/database/repositories/sql_tweet_repository.py` — timeline (own tweets included), author JOIN, viewer_has_liked LEFT JOIN, cursor pagination
- [ ] `backend/app/application/use_cases/tweets/create_tweet.py`
- [ ] `backend/app/application/use_cases/tweets/delete_tweet.py` — UnauthorizedError if not owner
- [ ] `backend/app/application/use_cases/tweets/get_timeline.py`
- [ ] `backend/app/application/use_cases/tweets/get_thread.py`
- [ ] `backend/app/application/use_cases/tweets/create_reply.py` — dispatches reply notification
- [ ] `backend/app/presentation/schemas/tweet.py` — includes author fields, viewer_has_liked
- [ ] `backend/app/presentation/api/v1/tweets.py`

**Tests**:
- [ ] `backend/tests/unit/test_create_tweet.py` — empty, too long, valid
- [ ] `backend/tests/unit/test_delete_tweet.py` — unauthorized, own tweet ok
- [ ] `backend/tests/integration/test_tweets.py` — create, delete, timeline cursor, own tweets in timeline, thread, replies excluded from timeline

---

## FASE 7 — Likes + Search Backend
**Commits**:
- `feat(likes-search): like/unlike with counters, user search with follower counts`
- `test(likes-search): like idempotency + search by partial username`

- [ ] `backend/app/infrastructure/database/repositories/sql_like_repository.py`
- [ ] `backend/app/application/use_cases/likes/like_tweet.py` — atomic UPDATE likes_count, dispatches like notification
- [ ] `backend/app/application/use_cases/likes/unlike_tweet.py` — atomic UPDATE likes_count
- [ ] `backend/app/application/use_cases/search/search_users.py` — ILIKE + followers_count subquery
- [ ] `backend/app/presentation/api/v1/likes.py`
- [ ] `backend/app/presentation/api/v1/search.py`

**Tests**:
- [ ] `backend/tests/unit/test_like_tweet.py` — tweet not found
- [ ] `backend/tests/integration/test_likes.py` — like, unlike, idempotent, likes_count in DB, viewer_has_liked
- [ ] `backend/tests/integration/test_search.py` — partial match, empty result, followers_count correct

---

## FASE 8 — Notifications + SSE Backend
**Commits**:
- `feat(notifications-sse): notifications model + SSE stream + batch mark-as-read`
- `test(notifications): auto-creation on follow/like/reply + read endpoints`

- [ ] `backend/app/infrastructure/sse/event_dispatcher.py` — `dict[UUID, asyncio.Queue]`
- [ ] `backend/app/infrastructure/database/repositories/sql_notification_repository.py` — JOIN for actor_username (no N+1)
- [ ] `backend/app/application/use_cases/notifications/get_notifications.py`
- [ ] `backend/app/application/use_cases/notifications/mark_notification_read.py`
- [ ] `backend/app/application/use_cases/notifications/mark_all_notifications_read.py`
- [ ] `backend/app/core/container.py` — `get_dispatcher()`, `get_storage()`
- [ ] `backend/app/presentation/api/v1/notifications.py` — GET list, SSE stream, PUT /{id}/read, PUT /read-all

**Tests**:
- [ ] `backend/tests/integration/test_notifications.py` — follow/like/reply generate notifications, mark read, mark-all-read, 401 without auth

---

## FASE 9 — Image Upload + Seed
**Commits**:
- `feat(uploads): image upload with DI-injected storage service`
- `feat(seed): realistic seed with 10 users, tweets, follows, likes, replies`

- [ ] `backend/app/infrastructure/storage/local_storage.py` — validate type, size, extension; UUID filename
- [ ] `backend/app/application/use_cases/users/upload_avatar.py`
- [ ] `backend/app/presentation/api/v1/uploads.py` — `Depends(get_storage)`
- [ ] `backend/alembic/versions/0002_add_image_url_to_tweets.py` (if not in 0001)
- [ ] `backend/seed.py` — Faker, 10 users, credentials: `alice@flock.com / password123`

**Tests**:
- [ ] `backend/tests/integration/test_uploads.py` — valid upload, bad extension, oversized

---

## FASE 10 — Frontend: Auth + Base
**Commits**:
- `feat(fe-auth): Next.js 15, shadcn, API client, auth pages with Zod validation`
- `test(fe-auth): login form validation and submit flow`

- [ ] `frontend/src/app/providers.tsx`
- [ ] `frontend/src/features/auth/schemas/auth-schemas.ts`
- [ ] `frontend/src/features/auth/api/auth-api.ts`
- [ ] `frontend/src/features/auth/hooks/use-auth.ts`
- [ ] `frontend/src/features/auth/hooks/use-session.ts`
- [ ] `frontend/src/features/auth/components/login-form.tsx`
- [ ] `frontend/src/features/auth/components/register-form.tsx`
- [ ] `frontend/src/app/(auth)/layout.tsx`
- [ ] `frontend/src/app/(auth)/login/page.tsx`
- [ ] `frontend/src/app/(auth)/register/page.tsx`
- [ ] `frontend/src/components/shared/avatar.tsx` — fallback to initials
- [ ] `frontend/tests/setup.ts`
- [ ] `frontend/tests/mocks/handlers.ts`

**Tests**:
- [ ] `frontend/tests/auth/login.test.tsx` — render, valid submit, invalid creds toast, Zod errors

---

## FASE 11 — Frontend: Layout
**Commits**:
- `feat(fe-layout): sidebar desktop/tablet, mobile nav, auth guard`
- `test(fe-layout): responsive breakpoints and auth guard redirect`

- [ ] `frontend/src/stores/ui-store.ts`
- [ ] `frontend/src/hooks/use-mobile.ts`
- [ ] `frontend/src/components/layout/sidebar.tsx`
- [ ] `frontend/src/components/layout/mobile-nav.tsx`
- [ ] `frontend/src/components/layout/main.tsx`
- [ ] `frontend/src/app/(main)/layout.tsx` — auth guard
- [ ] `frontend/src/app/(auth)/layout.tsx`

**Tests**:
- [ ] `frontend/tests/layout/sidebar.test.tsx` — desktop labels, mobile bottom nav, auth redirect

---

## FASE 12 — Frontend: Timeline + Composer
**Commits**:
- `feat(fe-timeline): home feed with infinite scroll and tweet composer`
- `test(fe-timeline): tweet composer validation and like optimistic update`

- [ ] `frontend/src/features/tweets/api/tweets-api.ts`
- [ ] `frontend/src/features/tweets/hooks/use-timeline.ts`
- [ ] `frontend/src/features/tweets/hooks/use-like.ts` — optimistic update across all pages
- [ ] `frontend/src/features/tweets/components/tweet-card.tsx`
- [ ] `frontend/src/features/tweets/components/tweet-composer.tsx`
- [ ] `frontend/src/features/tweets/components/tweet-actions.tsx`
- [ ] `frontend/src/components/shared/infinite-scroll.tsx`
- [ ] `frontend/src/components/shared/character-counter.tsx`
- [ ] `frontend/src/app/(main)/home/page.tsx`

**Tests**:
- [ ] `frontend/tests/tweets/tweet-composer.test.tsx`
- [ ] `frontend/tests/tweets/tweet-card.test.tsx`

---

## FASE 13 — Frontend: Profile
**Commits**:
- `feat(fe-profile): profile page with follow/unfollow, stats, and tweet list`
- `test(fe-profile): follow button optimistic update and rollback`

- [ ] `frontend/src/features/users/api/users-api.ts`
- [ ] `frontend/src/features/users/hooks/use-user-profile.ts`
- [ ] `frontend/src/features/users/hooks/use-follow.ts`
- [ ] `frontend/src/features/users/components/user-profile.tsx`
- [ ] `frontend/src/features/users/components/follow-button.tsx`
- [ ] `frontend/src/features/users/components/followers-list.tsx`
- [ ] `frontend/src/features/users/components/user-card.tsx`
- [ ] `frontend/src/app/(main)/profile/[username]/page.tsx`

**Tests**:
- [ ] `frontend/tests/users/follow-button.test.tsx`

---

## FASE 14 — Frontend: Search + Thread + Notifications
**Commits**:
- `feat(fe-search): search page with debounced user search`
- `feat(fe-thread): tweet reply threads`
- `feat(fe-notifications): notifications page + SSE real-time updates`
- `test(fe-search-thread-notifications): search, thread, and notification rendering`

- [ ] `frontend/src/features/search/api/search-api.ts`
- [ ] `frontend/src/features/search/hooks/use-search.ts` — debounce 400ms
- [ ] `frontend/src/features/search/components/search-input.tsx`
- [ ] `frontend/src/features/search/components/search-results.tsx`
- [ ] `frontend/src/app/(main)/search/page.tsx`
- [ ] `frontend/src/features/tweets/hooks/use-tweet.ts`
- [ ] `frontend/src/features/tweets/components/tweet-thread.tsx`
- [ ] `frontend/src/features/tweets/components/reply-composer.tsx`
- [ ] `frontend/src/app/(main)/tweet/[id]/page.tsx`
- [ ] `frontend/src/features/notifications/api/notifications-api.ts`
- [ ] `frontend/src/features/notifications/hooks/use-notifications.ts`
- [ ] `frontend/src/features/notifications/hooks/use-sse.ts` — cleanup on unmount
- [ ] `frontend/src/features/notifications/components/notification-item.tsx`
- [ ] `frontend/src/features/notifications/components/notification-bell.tsx`
- [ ] `frontend/src/app/(main)/notifications/page.tsx` — batch mark-all-read on mount

**Tests**:
- [ ] `frontend/tests/search/search.test.tsx`
- [ ] `frontend/tests/tweets/tweet-thread.test.tsx`
- [ ] `frontend/tests/notifications/notification-item.test.tsx`

---

## FASE 15 — Frontend: Image Upload UI
**Commit**: `feat(fe-upload): image attachment in tweet composer and avatar upload`

- [ ] Extend `tweet-composer.tsx` — file input, local preview, upload before POST
- [ ] Extend `user-profile.tsx` — avatar click → upload → invalidate profile query

---

## FASE 16 — E2E Tests
**Commits**:
- `test(e2e): Playwright auth flow — login, timeline, logout`
- `test(e2e): tweet creation and like flow`

- [ ] `frontend/playwright.config.ts`
- [ ] `frontend/e2e/auth.spec.ts` — login with alice, verify timeline, logout
- [ ] `frontend/e2e/tweets.spec.ts` — create tweet, like, open thread

---

## FASE 17 — Documentation + Polish
**Commits**:
- `docs: README with runbook, architecture decisions, example credentials`
- `chore: loading skeletons, empty states, error toasts, accessibility labels`

- [ ] `README.md` — full runbook, env vars, credentials, tech decisions
- [ ] `docs/development/runbook.md` — setup, seed, troubleshooting
- [ ] Loading skeletons in timeline, profile, notifications
- [ ] Empty states: "No tweets yet", "No notifications", "No results"
- [ ] Error toasts on all failed mutations
- [ ] `aria-label` on icon-only buttons

---

## FASE 18 — SDD Archive + Adversarial Review
**Commit**: `chore: archive openspec change and update product specs`

- [ ] Run `/code-auditing` — adversarial review
- [ ] Fix any FAIL or critical gaps
- [ ] Confirm `pytest --cov=app` ≥ 85%
- [ ] Move `openspec/changes/01-twitter-clone/` → `openspec/changes/archive/2026-08-XX-twitter-clone/`
- [ ] Create `openspec/specs/` with final product specs
