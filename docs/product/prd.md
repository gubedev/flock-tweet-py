# PRD — The Flock (Twitter Clone Challenge)

## Overview

The Flock is a full-stack Twitter/X clone built as a technical challenge. It demonstrates end-to-end product development: from REST API design with Clean Architecture to a modern React frontend with real-time features.

## Goals

- Build a functional microblogging platform replicating core X/Twitter features.
- Demonstrate production-quality code: tested, documented, containerized.
- Follow a rigorous development process (SDD cycle) evaluable by an auditor.

## Non-Goals

- DMs, Retweets/Quotes, Hashtags, Blocking/Muting
- Tweet editing, Email verification, Multi-image, Video
- Verification badges, Algorithmic timeline

## Users

| Role | Description |
|---|---|
| Visitor | Can view the login/register pages only |
| Authenticated user | Can tweet, follow, like, search, receive notifications |

## Features

### Authentication
- Register with email, username (unique), password (min 8 chars), display name
- Login with email + password
- JWT: access token (15 min) in response body; refresh token (7 days) in httpOnly cookie
- Auto-refresh: when access token expires, client retries with refresh cookie
- Logout: clears refresh cookie, invalidates client state
- `GET /auth/me`: returns current user data

### Tweets
- Create tweet (max 280 chars, configurable via env)
- Attach one image per tweet (upload via `POST /api/uploads/image`)
- Delete own tweet (403 if trying to delete another user's tweet)
- Timeline: paginated cursor-based feed of tweets from followed users + own tweets
- Tweet detail: single tweet with its reply thread

### Replies
- Reply to any tweet (`parent_id` set on the reply tweet)
- Replies appear in the tweet's thread view; excluded from main timeline
- `replies_count` on parent tweet increments atomically
- Author of parent tweet receives a `reply` notification

### Likes
- Like / unlike any tweet
- `likes_count` persists in DB (atomic UPDATE, not computed on the fly)
- `viewer_has_liked` returned in every tweet response for the authenticated user
- Like generates a `like` notification for the tweet's author

### Social Graph
- Follow / unfollow any user (cannot follow yourself)
- `followers_count` and `following_count` computed from the follows table (not cached columns)
- `viewer_is_following` returned in user profile response
- Follow generates a `follow` notification for the followed user
- Paginated lists: `/users/{username}/followers` and `/users/{username}/following`

### Search
- Search users by username or display name (partial match, case-insensitive)
- Results include correct `followers_count`

### Notifications
- Auto-generated on: follow, like, reply
- Displayed in chronological order (newest first)
- `read` flag per notification; batch mark-all-as-read endpoint
- Real-time delivery via Server-Sent Events (SSE stream)
- Unread count badge in navigation

### User Profile
- View any user's profile: avatar, banner, bio, stats (tweets / following / followers)
- Edit own profile: display name, bio
- Upload avatar image
- View user's tweets (paginated)

### Responsive UI
| Breakpoint | Layout |
|---|---|
| Mobile < 640px | Single column, bottom navigation bar |
| Tablet 640–1024px | Sidebar collapsed (icons only, 68px) |
| Desktop > 1024px | Full sidebar (275px) + central feed + right panel |

### Dark Mode
- Toggle stored in Zustand `ui-store`
- Applied via CSS variables in `theme.css`

## Design Reference

Replicates the visual design of X (Twitter) using shadcn/ui + Tailwind v4.

Key CSS variables:
- `--color-primary: #1D9BF0`
- `--color-like: #F91880`
- `--sidebar-width: 275px`
- `--feed-max-width: 600px`

## Technical Constraints

- Zero hardcoded values — all configuration via environment variables
- `docker-compose up --build` must start the full app without any manual steps
- Backend coverage ≥ 85% (pytest-cov)
- Commits are progressive — tests committed alongside each feature phase
