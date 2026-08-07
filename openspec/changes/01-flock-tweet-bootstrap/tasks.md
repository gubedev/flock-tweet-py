# Tasks: Flock Tweet Bootstrap — The Flock

Each task must be completed, tested, and committed before moving to the next.
Tests are committed in the same task as the feature — never deferred.

---

## FASE 0 — SDD Foundation ✅
**Commit**: `chore: initialize project with SDD structure and challenge spec`

- [x] `.gitignore` — includes `ignore/` and `.claude/temp/`
- [x] `CLAUDE.md` — workflow rules, skills index
- [x] `docs/product/prd.md`
- [x] `docs/architecture/architecture.md`
- [x] `docs/development/conventions.md`
- [x] `docs/development/development-flow.md`
- [x] `openspec/changes/01-flock-tweet-bootstrap/proposal.md`
- [x] `openspec/changes/01-flock-tweet-bootstrap/design.md`
- [x] `openspec/changes/01-flock-tweet-bootstrap/tasks.md` (this file)
- [x] `openspec/specs/product/auth/spec.md`
- [x] `openspec/specs/product/tweets/spec.md`
- [x] `openspec/specs/product/users/spec.md`
- [x] `openspec/specs/product/likes/spec.md`
- [x] `openspec/specs/product/notifications/spec.md`
- [x] `openspec/specs/product/search/spec.md`
- [x] `openspec/specs/tech/architecture/spec.md`
- [x] `openspec/specs/tech/database/spec.md`
- [x] `.claude/skills/commit/skill.md`
- [x] `.claude/skills/code-auditing/skill.md`
- [x] `.claude/skills/enrich-us/skill.md`
- [x] `.claude/skills/using-git-worktrees/skill.md`
- [x] `.claude/memory/MEMORY.md`
- [x] `.claude/rtk-reference.md`

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
- [ ] Next.js 16 init (App Router, TypeScript, Tailwind v4)
- [ ] shadcn/ui init
- [ ] `frontend/src/styles/theme.css` — CSS variables
- [ ] `frontend/src/config/env.ts` — Zod env validation
- [ ] `frontend/src/config/paths.ts` — type-safe routes
- [ ] `frontend/src/config/query-keys.ts`
- [ ] `frontend/src/lib/api-client.ts` — fetch wrapper, auto-refresh, NO localStorage
- [ ] `frontend/src/lib/utils.ts` — `cn()`
- [ ] `frontend/Dockerfile` — multi-stage + `output: 'standalone'`
- [ ] `frontend/next.config.ts` — `output: 'standalone'`
- [ ] `frontend/.env.example`
- [ ] `frontend/.dockerignore`
- [ ] `frontend/vitest.config.mts` — Vitest + jsdom setup

**Raíz**:
- [ ] `docker-compose.yml` — postgres + backend + frontend, `restart: unless-stopped`, healthcheck
- [ ] `.env.example`
- [ ] `README.md` (skeleton)

**Tests**:
- [ ] `backend/tests/integration/test_health.py` — `GET /health` → 200

---

## FASE 2 — DB Models + Domain Layer
**Commits**:
- `feat(db): SQLAlchemy models and initial Alembic migration`
- `test(db): schema constraints and table creation`
- `feat(domain): entities, repository interfaces, domain exceptions`
- `test(domain): tweet and user entity validation rules`

**DB Models**:
- [ ] `backend/app/infrastructure/database/connection.py` — async engine + session factory
- [ ] `backend/app/infrastructure/database/models/__init__.py`
- [ ] `backend/app/infrastructure/database/models/base.py`
- [ ] `backend/app/infrastructure/database/models/user_model.py`
- [ ] `backend/app/infrastructure/database/models/tweet_model.py` — includes `likes_count`, `replies_count`
- [ ] `backend/app/infrastructure/database/models/follow_model.py` — composite PK
- [ ] `backend/app/infrastructure/database/models/like_model.py` — composite PK
- [ ] `backend/app/infrastructure/database/models/notification_model.py` — ENUM type
- [ ] `backend/alembic/versions/0001_initial_schema.py`
- [ ] `backend/app/infrastructure/database/repositories/mappers.py` — `user_model_to_entity()`, `tweet_model_to_entity()`, `notification_model_to_entity()`

**Domain Layer**:
- [ ] `backend/app/domain/entities/user.py` — dataclass
- [ ] `backend/app/domain/entities/tweet.py` — `__post_init__`: EmptyTweetError, TweetTooLongError
- [ ] `backend/app/domain/entities/follow.py`
- [ ] `backend/app/domain/entities/like.py`
- [ ] `backend/app/domain/entities/notification.py`
- [ ] `backend/app/domain/repositories/user_repository.py` — ABC
- [ ] `backend/app/domain/repositories/tweet_repository.py` — ABC
- [ ] `backend/app/domain/repositories/follow_repository.py` — ABC
- [ ] `backend/app/domain/repositories/like_repository.py` — ABC
- [ ] `backend/app/domain/repositories/notification_repository.py` — ABC: `create()`, `get_by_user()`, `mark_read()`, `mark_all_read()`
- [ ] `backend/app/domain/services/password_service.py` — ABC
- [ ] `backend/app/domain/exceptions.py`

**Tests**:
- [ ] `backend/tests/conftest.py` — async SQLite in-memory engine, `async_client` fixture
- [ ] `backend/tests/integration/test_schema.py` — 5 tables exist, UNIQUE and composite PK constraints
- [ ] `backend/tests/unit/test_tweet_entity.py` — empty content, too long, valid
- [ ] `backend/tests/unit/test_user_entity.py` — valid entity

---

## FASE 3 — Auth
**Commits**:
- `feat(auth-be): JWT register, login, logout, refresh, get current user`
- `test(auth-be): unit register use case + integration auth endpoints`
- `feat(auth-fe): app shell layout, auth pages, Zod validation`
- `test(auth-fe): login form validation and submit flow`
- `test(e2e-auth): register, login, session persistence, logout`

**Backend**:
- [ ] `backend/app/infrastructure/security/jwt_service.py`
- [ ] `backend/app/infrastructure/security/password_service_impl.py` — bcrypt
- [ ] `backend/app/infrastructure/database/repositories/sql_user_repository.py` — `create()`, `get_by_email()`, `get_by_id()`
- [ ] `backend/app/application/use_cases/auth/register_user.py`
- [ ] `backend/app/application/use_cases/auth/login_user.py`
- [ ] `backend/app/application/use_cases/auth/refresh_token.py`
- [ ] `backend/app/application/use_cases/auth/logout_user.py`
- [ ] `backend/app/application/use_cases/auth/get_current_user.py`
- [ ] `backend/app/presentation/schemas/auth.py` — password `min_length=8`
- [ ] `backend/app/presentation/schemas/user.py`
- [ ] `backend/app/presentation/dependencies/auth.py` — `get_current_user` dependency
- [ ] `backend/app/presentation/dependencies/use_cases.py` — DI wiring
- [ ] `backend/app/presentation/api/v1/auth.py`
- [ ] `backend/app/presentation/api/v1/router.py`

**Frontend — App Shell**:
- [ ] `frontend/src/app/providers.tsx` — QueryClientProvider + AuthContext
- [ ] `frontend/src/stores/ui-store.ts` — Zustand: sidebarOpen, theme
- [ ] `frontend/src/hooks/use-mobile.ts` — detects `< 640px`
- [ ] `frontend/src/components/layout/sidebar.tsx` — desktop 275px / tablet 68px / mobile hidden
- [ ] `frontend/src/components/layout/mobile-nav.tsx` — bottom bar 60px, 4 icons
- [ ] `frontend/src/components/layout/main.tsx` — feed central max-width
- [ ] `frontend/src/app/(main)/layout.tsx` — auth guard: redirect to /login if no session
- [ ] `frontend/src/app/(auth)/layout.tsx` — minimal centered layout
- [ ] `frontend/src/components/shared/avatar.tsx` — fallback to initials

**Frontend — Auth**:
- [ ] `frontend/src/features/auth/schemas/auth-schemas.ts` — Zod: password min 8, username no spaces
- [ ] `frontend/src/features/auth/api/auth-api.ts`
- [ ] `frontend/src/features/auth/hooks/use-auth.ts`
- [ ] `frontend/src/features/auth/hooks/use-session.ts`
- [ ] `frontend/src/features/auth/components/login-form.tsx`
- [ ] `frontend/src/features/auth/components/register-form.tsx`
- [ ] `frontend/src/app/(auth)/login/page.tsx`
- [ ] `frontend/src/app/(auth)/register/page.tsx`
- [ ] `frontend/src/app/(main)/home/page.tsx` — minimal placeholder (renders layout shell, no tweets; replaced in FASE 5)
- [ ] `frontend/tests/setup.ts`
- [ ] `frontend/tests/mocks/handlers.ts`

**Backend Tests**:
- [ ] `backend/tests/conftest.py` — extend: add `authenticated_user` fixture (register + return token)
- [ ] `backend/tests/unit/test_register_user.py` — duplicate email, duplicate username, password hashed
- [ ] `backend/tests/integration/test_auth.py` — register, login, /me, refresh, logout (9 cases)

**Frontend Tests**:
- [ ] `frontend/tests/auth/login.test.tsx` — render, valid submit, invalid creds toast, Zod errors
- [ ] `frontend/tests/layout/sidebar.test.tsx` — desktop labels, mobile bottom nav, auth redirect

**E2E**:
- [ ] `frontend/playwright.config.ts`
- [ ] `frontend/e2e/auth.spec.ts` — register new user via API → /home, login, session persists on reload, logout → /login
  > E2E specs always create their own users via `POST /api/auth/register`. No dependency on seed data.

---

## FASE 4 — Users + Follows
**Commits**:
- `feat(users-be): profile endpoints, follow/unfollow with notification on follow`
- `test(users-be): unit follow rules + integration profile and follow endpoints`
- `feat(users-fe): profile page with follow/unfollow and stats`
- `test(users-fe): follow button optimistic update and rollback`
- `test(e2e-users): view profile, follow, unfollow, followers count`

**Backend**:
- [ ] `backend/app/infrastructure/database/repositories/sql_notification_repository.py` — implement `create()` only; `get_by_user()`, `mark_read()`, `mark_all_read()` deferred to FASE 8
- [ ] `backend/app/infrastructure/database/repositories/sql_user_repository.py` — extend: add `get_by_username()`, subquery for `followers_count`, `following_count`, and `tweets_count`
- [ ] `backend/app/infrastructure/database/repositories/sql_follow_repository.py` — cursor pagination
- [ ] `backend/app/application/use_cases/users/get_profile.py` — includes followers_count, following_count, viewer_is_following; `tweets_count` computed via COUNT subquery in `sql_user_repository` (no dependency on `sql_tweet_repository`, which doesn't exist until FASE 5)
- [ ] `backend/app/application/use_cases/users/update_profile.py`
- [ ] `backend/app/application/use_cases/follows/follow_user.py` — SelfFollowError, AlreadyFollowingError; persists `follow` notification in DB via `notification_repo.create()` (no SSE yet — dispatcher added in FASE 8)
- [ ] `backend/app/application/use_cases/follows/unfollow_user.py` — NotFollowingError
- [ ] `backend/app/presentation/api/v1/users.py`
- [ ] `backend/app/presentation/api/v1/follows.py`
- [ ] `backend/app/presentation/dependencies/use_cases.py` — extend: wire `notification_repo` into `follow_user`

**Frontend**:
- [ ] `frontend/src/features/users/api/users-api.ts`
- [ ] `frontend/src/features/users/hooks/use-user-profile.ts`
- [ ] `frontend/src/features/users/hooks/use-follow.ts` — optimistic follow/unfollow with rollback
- [ ] `frontend/src/features/users/components/user-profile.tsx` — banner, avatar, stats, follow button
- [ ] `frontend/src/features/users/components/follow-button.tsx` — syncs with server via useEffect
- [ ] `frontend/src/features/users/components/followers-list.tsx`
- [ ] `frontend/src/features/users/components/following-list.tsx`
- [ ] `frontend/src/features/users/components/user-card.tsx`
- [ ] `frontend/src/app/(main)/profile/[username]/page.tsx`

**Backend Tests**:
- [ ] `backend/tests/unit/test_follow_user.py` — self-follow, double-follow, unfollow without following
- [ ] `backend/tests/integration/test_users.py` — profile with correct counts, update, 404
- [ ] `backend/tests/integration/test_follows.py` — follow, unfollow, followers list (cursor), count correctness

**Frontend Tests**:
- [ ] `frontend/tests/users/follow-button.test.tsx` — follow, optimistic state, error rollback, disabled during mutation

**E2E**:
- [ ] `frontend/e2e/users.spec.ts` — register two users, view profile, follow → followers_count increases, unfollow → decreases

---

## FASE 5 — Tweets
**Commits**:
- `feat(tweets-be): create, delete, timeline with cursor pagination, reply threads`
- `test(tweets-be): unit ownership + integration timeline, pagination, replies`
- `feat(tweets-fe): home feed with infinite scroll, composer, thread view`
- `test(tweets-fe): composer validation, tweet card rendering, thread display`
- `test(e2e-tweets): create tweet, timeline, open thread, reply`

**Backend**:
- [ ] `backend/app/infrastructure/database/repositories/sql_tweet_repository.py` — timeline includes own tweets, author JOIN, `viewer_has_liked` LEFT JOIN, cursor pagination
- [ ] `backend/app/application/use_cases/tweets/create_tweet.py`
- [ ] `backend/app/application/use_cases/tweets/delete_tweet.py` — UnauthorizedError if not owner
- [ ] `backend/app/application/use_cases/tweets/get_timeline.py` — returns tweets + next_cursor
- [ ] `backend/app/application/use_cases/tweets/get_thread.py`
- [ ] `backend/app/application/use_cases/tweets/create_reply.py` — increments `replies_count` via atomic UPDATE on `tweet_repo`; persists `reply` notification in DB via `notification_repo.create()` (no SSE yet — dispatcher added in FASE 8)
- [ ] `backend/app/presentation/schemas/tweet.py` — includes author fields, viewer_has_liked
- [ ] `backend/app/presentation/api/v1/tweets.py`
- [ ] `backend/app/presentation/dependencies/use_cases.py` — extend: wire `tweet_repo` into all tweet use cases; additionally wire `notification_repo` into `create_reply`

**Frontend**:
- [ ] `frontend/src/features/tweets/api/tweets-api.ts`
- [ ] `frontend/src/features/tweets/hooks/use-timeline.ts` — `useInfiniteQuery`, cursor-based
- [ ] `frontend/src/features/tweets/components/tweet-card.tsx` — avatar, author info, content, delete for own tweets (like button wired in FASE 6)
- [ ] `frontend/src/features/tweets/components/tweet-composer.tsx` — textarea auto-resize, character counter, post button disabled if empty or over max
- [ ] `frontend/src/features/tweets/components/tweet-actions.tsx`
- [ ] `frontend/src/features/tweets/hooks/use-tweet.ts`
- [ ] `frontend/src/features/tweets/components/tweet-thread.tsx` — parent tweet + indented replies
- [ ] `frontend/src/features/tweets/components/reply-composer.tsx`
- [ ] `frontend/src/components/shared/infinite-scroll.tsx` — IntersectionObserver
- [ ] `frontend/src/components/shared/character-counter.tsx` — SVG circular, orange/red near limit
- [ ] `frontend/src/app/(main)/home/page.tsx` — full implementation; replaces the placeholder created in FASE 3
- [ ] `frontend/src/app/(main)/tweet/[id]/page.tsx`

**Backend Tests**:
- [ ] `backend/tests/unit/test_create_tweet.py` — empty, too long, valid
- [ ] `backend/tests/unit/test_delete_tweet.py` — unauthorized, own tweet ok
- [ ] `backend/tests/integration/test_tweets.py` — create, delete, timeline cursor, own tweets included, thread, replies excluded from timeline

**Frontend Tests**:
- [ ] `frontend/tests/tweets/tweet-composer.test.tsx` — empty → disabled, typing → enabled, over max → disabled + red
- [ ] `frontend/tests/tweets/tweet-card.test.tsx` — own tweet shows delete, other tweet hides it
- [ ] `frontend/tests/tweets/tweet-thread.test.tsx` — parent + replies render, reply composer visible

**E2E**:
- [ ] `frontend/e2e/tweets.spec.ts` — register user, create tweet → appears in timeline, click tweet → thread opens, reply → appears in thread

---

## FASE 6 — Likes
**Commits**:
- `feat(likes-be): like/unlike with atomic counters and like notification`
- `test(likes-be): idempotency and likes_count correctness`
- `feat(likes-fe): optimistic like update across all timeline pages`
- `test(likes-fe): optimistic like and rollback on error`
- `test(e2e-likes): like tweet, counter increases, unlike, counter decreases`

**Backend**:
- [ ] `backend/app/infrastructure/database/repositories/sql_like_repository.py`
- [ ] `backend/app/application/use_cases/likes/like_tweet.py` — atomic `UPDATE likes_count + 1`; persists `like` notification in DB via `notification_repo.create()` (no SSE yet — dispatcher added in FASE 8)
- [ ] `backend/app/application/use_cases/likes/unlike_tweet.py` — atomic `UPDATE likes_count - 1`
- [ ] `backend/app/presentation/api/v1/likes.py`
- [ ] `backend/app/presentation/dependencies/use_cases.py` — extend: wire `tweet_repo` and `notification_repo` into `like_tweet`

**Frontend**:
- [ ] `frontend/src/features/tweets/hooks/use-like.ts` — optimistic update across all infinite query pages, rollback on error
- [ ] Update `tweet-card.tsx` — wire like button to `use-like`, animate icon, initialize from `viewer_has_liked`

**Backend Tests**:
- [ ] `backend/tests/unit/test_like_tweet.py` — tweet not found
- [ ] `backend/tests/integration/test_likes.py` — like, unlike, idempotent double-like, `likes_count` in DB, `viewer_has_liked`

**Frontend Tests**:
- [ ] Update `frontend/tests/tweets/tweet-card.test.tsx` — like click → optimistic state, error → rollback

**E2E**:
- [ ] `frontend/e2e/likes.spec.ts` — register user, create tweet, like → counter +1 → reload → persists, unlike → counter -1

---

## FASE 7 — Search
**Commits**:
- `feat(search-be): user search with ILIKE and accurate follower counts`
- `test(search-be): partial match and followers_count correctness`
- `feat(search-fe): search page with debounced user search`
- `test(search-fe): debounce, results rendering, empty state`
- `test(e2e-search): search user by partial name`

**Backend**:
- [ ] `backend/app/infrastructure/database/repositories/sql_user_repository.py` — extend: add `search(query)` method (ILIKE on username + display_name, includes `followers_count` subquery)
- [ ] `backend/app/application/use_cases/search/search_users.py` — delegates to `user_repo.search()`
- [ ] `backend/app/presentation/api/v1/search.py`

**Frontend**:
- [ ] `frontend/src/features/search/api/search-api.ts`
- [ ] `frontend/src/features/search/hooks/use-search.ts` — debounce 400ms
- [ ] `frontend/src/features/search/components/search-input.tsx`
- [ ] `frontend/src/features/search/components/search-results.tsx`
- [ ] `frontend/src/app/(main)/search/page.tsx`

**Backend Tests**:
- [ ] `backend/tests/integration/test_search.py` — partial username match, partial display_name match, empty result, `followers_count` not 0

**Frontend Tests**:
- [ ] `frontend/tests/search/search.test.tsx` — debounce fires after 400ms, empty query → no API call, no results → empty state

**E2E**:
- [ ] `frontend/e2e/search.spec.ts` — register two users, search by partial name → correct user appears with `followers_count`

---

## FASE 8 — Notifications + SSE
**Commits**:
- `feat(notifications-be): complete notification system with SSE dispatch`
- `test(notifications-be): auto-creation on follow/like/reply + read endpoints`
- `feat(notifications-fe): notifications page with SSE real-time updates`
- `test(notifications-fe): notification item rendering and unread highlight`
- `test(e2e-notifications): real-time notification on follow, like, reply`

**Backend**:
- [ ] `backend/app/infrastructure/sse/event_dispatcher.py` — `dict[UUID, asyncio.Queue]`; module-level singleton
- [ ] `backend/app/core/container.py` — `get_dispatcher()` Depends factory
- [ ] `backend/app/infrastructure/database/repositories/sql_notification_repository.py` — extend: implement `get_by_user()`, `mark_read()`, `mark_all_read()` (batch UPDATE); `create()` was added in FASE 4
- [ ] `backend/app/application/use_cases/notifications/get_notifications.py`
- [ ] `backend/app/application/use_cases/notifications/mark_notification_read.py`
- [ ] `backend/app/application/use_cases/notifications/mark_all_notifications_read.py`
- [ ] `backend/app/presentation/api/v1/notifications.py` — `GET /api/notifications`, `GET /api/notifications/stream` (SSE), `PUT /api/notifications/{id}/read`, `PUT /api/notifications/read-all`
- [ ] `backend/app/presentation/dependencies/use_cases.py` — extend: inject `event_dispatcher` into `follow_user`, `create_reply`, `like_tweet`; wire notification use cases
- [ ] Update `follow_user.py` — add SSE dispatch via `event_dispatcher.publish()` after DB notification created
- [ ] Update `create_reply.py` — add SSE dispatch via `event_dispatcher.publish()` after DB notification created
- [ ] Update `like_tweet.py` — add SSE dispatch via `event_dispatcher.publish()` after DB notification created

**Frontend**:
- [ ] `frontend/src/features/notifications/api/notifications-api.ts`
- [ ] `frontend/src/features/notifications/hooks/use-notifications.ts`
- [ ] `frontend/src/features/notifications/hooks/use-sse.ts` — EventSource with credentials, invalidate on message, cleanup on unmount (`es.close()`)
- [ ] `frontend/src/features/notifications/components/notification-item.tsx` — follow/like/reply text, highlighted background if `read: false`
- [ ] `frontend/src/features/notifications/components/notification-bell.tsx` — badge with unread count
- [ ] `frontend/src/app/(main)/notifications/page.tsx` — `PUT /api/notifications/read-all` on mount (once, not on every render)
- [ ] Update `sidebar.tsx` — add `notification-bell` to navigation (requires `use-notifications` hook)

**Backend Tests**:
- [ ] `backend/tests/integration/test_notifications.py` — 401 without auth; follow/like/reply each persist a notification in DB; PUT /{id}/read → `read: true`; PUT /read-all → all marked read; SSE stream returns 200

**Frontend Tests**:
- [ ] `frontend/tests/notifications/notification-item.test.tsx` — follow text correct, like text correct, reply text correct; `read: false` → highlighted; `read: true` → normal

**E2E**:
- [ ] `frontend/e2e/notifications.spec.ts` — register two users; user A follows user B; user B's notification bell updates; notification appears in list in real-time via SSE

---

## FASE 9 — Image Upload + Seed
**Commits**:
- `feat(uploads-be): image upload with DI-injected storage and seed data`
- `test(uploads-be): valid upload, bad extension, oversized`
- `feat(uploads-fe): image attachment in composer and avatar upload`
- `test(e2e-uploads): tweet with image, avatar upload`

**Backend**:
- [ ] `backend/app/infrastructure/storage/local_storage.py` — validates type, size, extension from env; UUID filename
- [ ] `backend/app/core/container.py` — extend: add `get_storage()` Depends factory (LocalStorage now available)
- [ ] `backend/app/application/use_cases/users/upload_avatar.py`
- [ ] `backend/app/presentation/api/v1/uploads.py` — `Depends(get_storage)` injected
- [ ] `backend/alembic/versions/0002_add_image_url_to_tweets.py` — adds `image_url VARCHAR NULLABLE` to tweets (`image_url` intentionally omitted from 0001 to keep domain phases clean)
- [ ] `backend/seed.py` — Faker: 10 users with bio, cross-follows, tweets, likes, reply threads; credentials: `alice@flock.com / password123`
  > Seed is for manual demo and exploratory testing only. E2E specs never depend on seed data.

**Frontend**:
- [ ] Extend `tweet-composer.tsx` — file input (accept from env), local preview with cancel, upload before `POST /api/tweets`
- [ ] Extend `tweet-card.tsx` — display `image_url` when present (img tag below tweet content)
- [ ] Extend `user-profile.tsx` — avatar click → file input → `POST /api/users/me/avatar` → invalidate profile query

**Backend Tests**:
- [ ] `backend/tests/integration/test_uploads.py` — valid image → 200 + URL, bad extension → 422, oversized → 413

**E2E**:
- [ ] `frontend/e2e/uploads.spec.ts` — register user, upload avatar → visible on profile; create tweet with image → image displayed in timeline

---

## FASE 10 — Documentation + Polish
**Commits**:
- `docs: README with runbook, architecture decisions, example credentials`
- `chore: loading skeletons, empty states, error toasts, accessibility labels`

- [ ] `README.md` — prerequisites, `docker-compose up --build`, seed, test commands, env vars, tech decisions, credentials
- [ ] `docs/development/runbook.md` — setup, seed, test, troubleshoot
- [ ] Loading skeletons in timeline, profile, notifications
- [ ] Empty states: "No tweets yet", "No notifications", "No results"
- [ ] Error toasts on all failed mutations
- [ ] `aria-label` on all icon-only buttons

---

## FASE 11 — SDD Archive + Adversarial Review
**Commit**: `chore: archive openspec change and update product specs`

- [ ] Run `/code-auditing` — adversarial review
- [ ] Fix any FAIL or critical gaps
- [ ] Confirm `pytest --cov=app` ≥ 85%
- [ ] Move `openspec/changes/01-flock-tweet-bootstrap/` → `openspec/changes/archive/2026-08-XX-flock-tweet-bootstrap/`
- [ ] Create `openspec/specs/` with final product specs
