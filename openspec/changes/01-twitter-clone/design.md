# Design: Twitter Clone Challenge — The Flock

---

## Backend Architecture

### Layer Dependency Graph

```
presentation  →  application  →  domain  ←  infrastructure
     ↓                ↓              ↑              ↑
  schemas         use_cases      entities       sql_repos
  routers         (one per         ABCs          models
  depends()        action)      exceptions       mappers
```

Domain has zero outward dependencies. Infrastructure implements domain ABCs.

### Use Case List

```
auth/           register_user, login_user, refresh_token, logout_user, get_current_user
tweets/         create_tweet, delete_tweet, get_timeline, get_thread, create_reply
users/          get_profile, update_profile, upload_avatar
follows/        follow_user, unfollow_user
likes/          like_tweet, unlike_tweet
search/         search_users
notifications/  get_notifications, mark_notification_read, mark_all_notifications_read
```

### Key Design Decisions

#### `GetCurrentUserUseCase`
`GET /auth/me` must go through a use case, not directly inject `SqlUserRepository`. This enforces the layer contract and makes the endpoint testable in isolation.

#### `mappers.py`
All model→entity conversions live in `infrastructure/database/repositories/mappers.py`:
```python
def user_model_to_entity(m: UserModel, followers_count=0, following_count=0) -> User: ...
def tweet_model_to_entity(m: TweetModel, viewer_has_liked=False, ...) -> Tweet: ...
```
No `_to_entity()` methods inside individual repositories.

#### `followers_count` — Subquery at Read Time
```python
followers_count = (
    select(func.count())
    .where(FollowModel.following_id == UserModel.id)
    .scalar_subquery()
)
# Used in: get_by_username(), get_by_id(), search_users()
```
Chosen over stored columns to avoid count drift on concurrent follow/unfollow operations.

#### `likes_count` / `replies_count` — Atomic UPDATE
```python
await session.execute(
    update(TweetModel)
    .where(TweetModel.id == tweet_id)
    .values(likes_count=TweetModel.likes_count + 1)
)
```
Stored as columns for read performance; maintained via atomic UPDATEs to prevent race conditions.

#### `viewer_has_liked` — LEFT JOIN at Read Time
```python
liked_subquery = (
    select(LikeModel.tweet_id)
    .where(LikeModel.user_id == current_user_id)
    .scalar_subquery()
)
stmt = select(TweetModel, liked_subquery.label("viewer_has_liked"))
```

#### Timeline Query — Includes Own Tweets
```python
.where(
    or_(
        TweetModel.author_id.in_(following_subquery),
        TweetModel.author_id == current_user_id,
    )
)
.where(TweetModel.parent_id.is_(None))  # exclude replies
.where(TweetModel.created_at < cursor)
.order_by(TweetModel.created_at.desc())
.limit(page_size)
```

#### Reply Notifications
`CreateReplyUseCase` (or `CreateTweetUseCase` when `parent_id` is set) receives `notification_repo` as a dependency and dispatches a `reply` notification to the parent tweet's author:
```python
if tweet.parent_id:
    parent = await tweet_repo.get_by_id(tweet.parent_id)
    if parent and parent.author_id != author_id:
        await notification_repo.create(Notification(
            recipient_id=parent.author_id,
            actor_id=author_id,
            type=NotificationType.REPLY,
            tweet_id=tweet.id,
        ))
        await dispatcher.publish(parent.author_id, notification)
```

#### Storage DI
```python
# core/container.py
def get_storage(settings: Settings = Depends(get_settings)) -> LocalStorage:
    return LocalStorage(settings)

# presentation/api/v1/uploads.py
@router.post("/image")
async def upload_image(
    file: UploadFile,
    storage: LocalStorage = Depends(get_storage),  # injected
    ...
```

#### Cookie Security Conditional
```python
response.set_cookie(
    key="refresh_token",
    value=refresh_token,
    httponly=True,
    secure=settings.app_env != "development",
    samesite="lax",
)
```

#### Rate Limiting with Proxy Awareness
```python
def get_client_ip(request: Request) -> str:
    forwarded = request.headers.get("X-Forwarded-For")
    if forwarded:
        return forwarded.split(",")[0].strip()
    return request.client.host

limiter = Limiter(key_func=get_client_ip)
```

---

## Frontend Architecture

### API Client (`lib/api-client.ts`)
```typescript
let accessToken: string | null = null

async function request<T>(path: string, init?: RequestInit, isRetry = false): Promise<T> {
  const res = await fetch(`${env.NEXT_PUBLIC_API_URL}${path}`, {
    ...init,
    credentials: "include",  // sends refresh cookie
    headers: { Authorization: accessToken ? `Bearer ${accessToken}` : "", ...init?.headers },
  })
  if (res.status === 401 && !isRetry) {
    const refreshed = await refresh()
    if (refreshed) return request(path, init, true)
    throw new UnauthorizedError()
  }
  return res.json()
}
```
No `localStorage`. Token lives in memory only; re-acquired via refresh cookie on page reload.

### Optimistic Like Update — All Infinite Query Pages
```typescript
// use-like.ts
onMutate: async ({ tweetId, liked }) => {
  await queryClient.cancelQueries({ queryKey: queryKeys.tweets.timeline() })
  const prev = queryClient.getQueryData(queryKeys.tweets.timeline())
  queryClient.setQueryData(queryKeys.tweets.timeline(), (old: InfiniteData<TimelinePage>) => ({
    ...old,
    pages: old.pages.map(page => ({
      ...page,
      tweets: page.tweets.map(t =>
        t.id === tweetId
          ? { ...t, likes_count: t.likes_count + (liked ? 1 : -1), viewer_has_liked: liked }
          : t
      ),
    })),
  }))
  return { prev }
},
onError: (_, __, ctx) => {
  queryClient.setQueryData(queryKeys.tweets.timeline(), ctx?.prev)
},
```

### SSE Cleanup
```typescript
// use-sse.ts
useEffect(() => {
  if (!enabled) return
  const es = new EventSource(`${env.NEXT_PUBLIC_API_URL}/api/notifications/stream`, {
    withCredentials: true,
  })
  es.onmessage = () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.notifications.all() })
  }
  return () => es.close()  // cleanup prevents accumulated open connections
}, [enabled])
```

### Follow Button — Sync with Server
```typescript
// follow-button.tsx
const [isFollowing, setIsFollowing] = useState(initialFollowing)
useEffect(() => {
  setIsFollowing(initialFollowing)
}, [initialFollowing])  // re-sync when navigating between profiles
```

### Notifications — Single Batch Mark-as-Read
```typescript
// notifications/page.tsx
useEffect(() => {
  if (notifications?.some(n => !n.read)) {
    markAllRead.mutate()
  }
}, [])  // only on mount, not on every render
```

---

## Docker Setup

### `entrypoint.sh`
```bash
#!/bin/sh
set -e
alembic upgrade head
exec uvicorn app.main:app --host 0.0.0.0 --port "${APP_PORT:-8000}"
# No --reload in production
```

### `docker-compose.yml` key settings
```yaml
services:
  backend:
    restart: unless-stopped
    command: ./entrypoint.sh
    depends_on:
      postgres:
        condition: service_healthy
  frontend:
    restart: unless-stopped
  postgres:
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
```

---

## Migrations

| Version | Description |
|---|---|
| `0001_initial_schema` | users, tweets (with likes_count, replies_count), follows, likes, notifications |
| `0002_add_image_url_to_tweets` | adds `image_url VARCHAR NULLABLE` to tweets if not included in 0001 |

---

## Test Strategy

### Backend
- **Unit tests** (`tests/unit/`): test domain entities and use case logic with mocked repositories. No DB, no FastAPI.
- **Integration tests** (`tests/integration/`): test HTTP endpoints via `httpx.AsyncClient`. Use async SQLite in-memory DB configured in `conftest.py`.
- **Coverage**: `pytest --cov=app --cov-report=term-missing` → ≥ 85%.

### Frontend
- **Unit tests** (`tests/`): Vitest + `@testing-library/react` + MSW for API mocking.
- **E2E** (`e2e/`): Playwright against the full Docker stack.

### Seed credentials for E2E
```
alice@flock.com / password123
```
