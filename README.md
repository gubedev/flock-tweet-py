# The Flock

A production-grade Twitter/X clone — FastAPI backend with Clean Architecture, Next.js frontend with feature-based structure, real-time SSE notifications, and cursor-based pagination.

## Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI · SQLAlchemy 2.0 async · Alembic · PostgreSQL |
| Frontend | Next.js 16 · TanStack Query v5 · Tailwind CSS v4 · shadcn/ui · Zustand |
| Infrastructure | Docker Compose · multi-stage Dockerfiles |
| Testing | pytest · Vitest · Playwright |

## Architecture

**Backend**: Clean Architecture — domain → application → infrastructure → presentation. Zero framework dependencies in the domain layer.

**Frontend**: Feature-based modules (auth, tweets, users, search, notifications). Access token in memory only; refresh token in httpOnly cookie.

## Getting Started

```bash
cp .env.example .env
docker compose up
```

- Frontend: http://localhost:3000
- Backend API docs: http://localhost:8000/docs

## Development

See [Development Flow](docs/development/development-flow.md) for the phase-by-phase workflow and branching strategy.
