# Proposal: Flock Tweet Bootstrap — The Flock

**Change ID**: 01-flock-tweet-bootstrap
**Status**: in-progress
**Started**: 2026-08-04

---

## Why

The Flock is a technical challenge that evaluates full-stack development capability across five dimensions: functional correctness, testing rigor, code quality, development process, and documentation. The challenge requires building a Twitter/X clone from scratch with a backend API and a frontend SPA, demonstrating production-grade decisions at every layer.

---

## What Changes

### New System: Backend (FastAPI + Clean Architecture)

- **Domain layer**: pure Python entities and repository ABCs (zero framework dependencies).
- **Application layer**: one use case per action; orchestrates domain entities.
- **Infrastructure layer**: SQLAlchemy 2.0 async repositories; centralized `mappers.py`.
- **Presentation layer**: FastAPI routers + Pydantic schemas; DI via `Depends()`.
- **Security**: JWT access (15 min, in-memory) + refresh (7 days, httpOnly cookie). Rate limiting with `X-Forwarded-For`.

### New System: Frontend (Next.js 16 + Feature-based)

- Feature modules: auth, tweets, users, search, notifications.
- TanStack Query v5 for server state; Zustand for UI state only.
- Optimistic likes and follows with rollback.
- SSE-based real-time notification delivery.
- Responsive layout: mobile / tablet / desktop.

### New System: Infrastructure

- Docker Compose: PostgreSQL + backend + frontend.
- Auto-migration: `entrypoint.sh` runs `alembic upgrade head` before starting uvicorn.
- Multi-stage Dockerfiles for both services.
- Realistic seed with 10 users, cross-follows, tweets, likes, and reply threads.

---

## Capabilities After This Change

- User registration, login, logout, session persistence across reloads.
- Publish tweets with optional image attachment.
- Follow/unfollow users; view follower and following lists.
- Like/unlike tweets with persistent counters.
- Cursor-paginated timeline including own tweets.
- Reply threads with notification to original tweet author.
- User search with accurate follower counts.
- Real-time notifications via SSE: follow, like, reply.
- Responsive UI across mobile / tablet / desktop.
- Full test suite: ≥ 85% backend coverage, frontend unit + E2E tests.

---

## Impact

- **New**: entire project (empty repo).
- **Roles affected**: challenge auditor (evaluates process), technical reviewer (evaluates code).
- **No breaking changes**: greenfield.

---

## Non-Goals

- DMs, Retweets, Hashtags, Blocking, Tweet editing.
- Email verification, push notifications, video upload.
- Multi-worker SSE (no Redis pub/sub — single-worker deployment).
- Algorithmic timeline ranking.
