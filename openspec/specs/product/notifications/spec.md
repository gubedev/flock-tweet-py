# Spec: Notifications

**Status**: specified
**Phase**: FASE 8 — Notifications + SSE (BE + FE + E2E)
**Created**: 2026-08-04

---

## Purpose

Automatically notify users of activity relevant to them: when they are followed, when their tweet is liked, or when someone replies to their tweet. Notifications are delivered in real time via Server-Sent Events and can be read and dismissed in bulk.

---

## REQ-01: Auto-Generation

Notifications are created server-side as side effects of the triggering actions.

| Trigger | Recipient | Type | Condition |
|---|---|---|---|
| User A follows user B | B | `follow` | Always |
| User A likes tweet T (author B) | B | `like` | A ≠ B |
| User A replies to tweet T (author B) | B | `reply` | A ≠ B |

**Scenario REQ-01-A: Follow notification**
- GIVEN user A follows user B
- WHEN `follow_user` use case runs
- THEN a notification `{type: "follow", actor_id: A.id, recipient_id: B.id}` is persisted

**Scenario REQ-01-B: Like notification**
- GIVEN user A likes tweet T authored by B, and A ≠ B
- WHEN `like_tweet` use case runs
- THEN a notification `{type: "like", actor_id: A.id, recipient_id: B.id, tweet_id: T.id}` is persisted

**Scenario REQ-01-C: Reply notification**
- GIVEN user A replies to tweet T authored by B, and A ≠ B
- WHEN `create_reply` use case runs
- THEN a notification `{type: "reply", actor_id: A.id, recipient_id: B.id, tweet_id: T.id}` is persisted

**Scenario REQ-01-D: No self-notification**
- GIVEN user A likes or replies to their own tweet
- WHEN the use case runs
- THEN NO notification is created

---

## REQ-02: List Notifications

**Scenario REQ-02-A: Fetch notifications**
- GIVEN an authenticated user
- WHEN `GET /api/notifications` is called
- THEN list of notifications is returned, ordered newest first
- AND each notification includes: id, type, read, created_at, actor_username, actor_avatar_url, tweet_id (nullable)
- AND actor fields are loaded via JOIN — no N+1 queries

**Scenario REQ-02-B: Unauthenticated access**
- GIVEN no valid access token
- WHEN `GET /api/notifications` is called
- THEN 401 Unauthorized returned

---

## REQ-03: Real-Time Delivery via SSE

**Scenario REQ-03-A: SSE connection**
- GIVEN an authenticated user with a valid access token
- WHEN `GET /api/notifications/stream` is called
- THEN a persistent SSE connection is opened; the server pushes an event each time a new notification is created for that user

**Scenario REQ-03-B: Frontend SSE lifecycle**
- GIVEN the notifications page or bell is mounted
- WHEN a new notification arrives via SSE
- THEN the React Query cache for notifications is invalidated, triggering a refetch
- AND when the component unmounts, `EventSource.close()` is called to avoid connection leaks

**Scenario REQ-03-C: SSE reconnection**
- GIVEN the SSE connection drops
- WHEN the browser auto-reconnects
- THEN the server accepts the new connection and continues delivering events

---

## REQ-04: Mark Notification as Read

**Scenario REQ-04-A: Mark individual as read**
- GIVEN an unread notification with id `N`
- WHEN `PUT /api/notifications/{id}/read` is called
- THEN `notification.read = true`, 200 returned

---

## REQ-05: Mark All Notifications as Read

**Scenario REQ-05-A: Batch mark-as-read**
- GIVEN a user with multiple unread notifications
- WHEN `PUT /api/notifications/read-all` is called
- THEN all notifications for that user are set to `read = true` in a single UPDATE

**Scenario REQ-05-B: Frontend auto-trigger**
- GIVEN the user navigates to the notifications page
- WHEN the page mounts and unread notifications exist
- THEN `PUT /api/notifications/read-all` is called once (on mount, not on every render)

---

## REQ-06: Unread Count Badge

**Scenario REQ-06-A: Badge display**
- GIVEN a user with 3 unread notifications
- WHEN the sidebar or mobile nav is rendered
- THEN the notification bell shows a badge with count "3"

**Scenario REQ-06-B: Badge clears after reading**
- GIVEN the user opens the notifications page (triggers mark-all-read)
- WHEN they navigate away and back
- THEN the badge is no longer visible

