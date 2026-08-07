# Spec: Database

**Status**: specified
**Phase**: FASE 2 — DB Models + Domain Layer
**Created**: 2026-08-04

---

## Purpose

Define the relational schema, migration strategy, and query patterns that govern data persistence. All decisions here directly affect correctness of counts, pagination stability, and concurrent write safety.

---

## REQ-01: Schema

Five tables form the core data model.

```sql
users
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid()
  username      VARCHAR(50) UNIQUE NOT NULL
  email         VARCHAR(255) UNIQUE NOT NULL
  password_hash VARCHAR NOT NULL
  display_name  VARCHAR(100) NOT NULL
  bio           TEXT
  avatar_url    VARCHAR
  created_at    TIMESTAMP DEFAULT now()
  updated_at    TIMESTAMP DEFAULT now()

tweets
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid()
  content       VARCHAR(280) NOT NULL
  author_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  parent_id     UUID REFERENCES tweets(id) ON DELETE CASCADE   -- NULL = top-level tweet
  image_url     VARCHAR                                         -- added in migration 0002
  likes_count   INTEGER NOT NULL DEFAULT 0
  replies_count INTEGER NOT NULL DEFAULT 0
  created_at    TIMESTAMP DEFAULT now()

follows
  follower_id   UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  following_id  UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  created_at    TIMESTAMP DEFAULT now()
  PRIMARY KEY (follower_id, following_id)

likes
  user_id       UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  tweet_id      UUID NOT NULL REFERENCES tweets(id) ON DELETE CASCADE
  created_at    TIMESTAMP DEFAULT now()
  PRIMARY KEY (user_id, tweet_id)

notifications
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid()
  recipient_id  UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  actor_id      UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE
  type          notification_type NOT NULL  -- SQLAlchemy Enum('follow', 'like', 'reply')
  tweet_id      UUID REFERENCES tweets(id) ON DELETE SET NULL
  read          BOOLEAN NOT NULL DEFAULT FALSE
  created_at    TIMESTAMP DEFAULT now()
```

---

## REQ-02: Migrations

| Version | File | Description |
|---|---|---|
| 0001 | `0001_initial_schema.py` | Creates all five tables above (including likes_count, replies_count on tweets) |
| 0002 | `0002_add_image_url_to_tweets.py` | Adds nullable `image_url` column to tweets (if not included in 0001) |

Migration rules:
- Alembic is the only tool used for schema changes — no raw `CREATE TABLE` in application code.
- `entrypoint.sh` runs `alembic upgrade head` before starting uvicorn, making migrations automatic on `docker-compose up`.
- Downgrade scripts are written for every migration.

---

## REQ-03: Cursor-based Pagination

All paginated list endpoints use a cursor derived from `created_at` (ISO timestamp), not integer offsets.

```python
# Timeline query pattern
.where(TweetModel.created_at < cursor)
.order_by(TweetModel.created_at.desc())
.limit(page_size)
```

Response shape:
```json
{
  "items": [...],
  "next_cursor": "2026-08-04T12:00:00Z"  // or null if no more pages
}
```

**Why**: offset pagination shifts under concurrent insertions. A new tweet at position 1 would shift every subsequent page, causing items to be skipped or duplicated during scrolling.

---

## REQ-04: likes_count and replies_count — Atomic UPDATE

Stored as integer columns on `tweets`. Maintained via atomic SQL UPDATEs:

```python
# Like
await session.execute(
    update(TweetModel)
    .where(TweetModel.id == tweet_id)
    .values(likes_count=TweetModel.likes_count + 1)
)

# Reply creation
await session.execute(
    update(TweetModel)
    .where(TweetModel.id == parent_id)
    .values(replies_count=TweetModel.replies_count + 1)
)
```

**Scenario REQ-04-A: Concurrent likes**
- GIVEN 10 concurrent `POST /api/tweets/{id}/like` requests
- WHEN all complete
- THEN `tweet.likes_count` is exactly 10 (no lost updates due to atomic UPDATE)

---

## REQ-05: followers_count and following_count — Subquery at Read Time

Not stored as columns. Computed inline on every profile read:

```python
followers_subq = (
    select(func.count())
    .where(FollowModel.following_id == UserModel.id)
    .correlate(UserModel)
    .scalar_subquery()
)

following_subq = (
    select(func.count())
    .where(FollowModel.follower_id == UserModel.id)
    .correlate(UserModel)
    .scalar_subquery()
)
```

Used in: `get_by_username()`, `get_by_id()`, `search_users()`.

**Scenario REQ-05-A: No count drift**
- GIVEN 100 concurrent follow/unfollow operations on user B
- WHEN they complete
- THEN `GET /api/users/B` returns a `followers_count` that matches the actual row count in `follows`

---

## REQ-06: viewer_has_liked — LEFT JOIN at Read Time

Computed per tweet for the authenticated user via scalar subquery:

```python
liked_sq = (
    select(LikeModel.tweet_id)
    .where(LikeModel.user_id == current_user_id)
    .scalar_subquery()
)
stmt = select(
    TweetModel,
    liked_sq.label("viewer_has_liked"),
)
```

---

## REQ-07: async Session

All database access uses async SQLAlchemy sessions:

```python
# connection.py
engine = create_async_engine(settings.database_url, echo=False)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_db_session() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        yield session
```

---

## REQ-08: Test Database

Integration tests use an async SQLite in-memory database configured in `backend/tests/conftest.py`. The same SQLAlchemy models are used — no schema divergence between test and production.

```python
TEST_DATABASE_URL = "sqlite+aiosqlite:///:memory:"
```

