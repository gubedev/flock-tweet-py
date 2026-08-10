# Spec: Users & Social Graph

**Status**: specified
**Phase**: FASE 4 — Users + Follows (BE + FE + E2E)
**Created**: 2026-08-04

---

## Purpose

Allow users to view and edit their own profiles, follow and unfollow other users, and access accurate social graph counts. Follower and following counts are computed at read time from the follows table — never stored as cached columns.

---

## REQ-01: View User Profile

**Scenario REQ-01-A: View another user's profile**
- GIVEN a username that exists
- WHEN `GET /api/users/{username}` is called by an authenticated user
- THEN 200 returned with: id, username, display_name, bio, avatar_url, tweets_count, followers_count, following_count, viewer_is_following

**Scenario REQ-01-B: User not found**
- GIVEN a username that does not exist
- WHEN `GET /api/users/{username}` is called
- THEN 404 Not Found returned

**Scenario REQ-01-C: viewer_is_following accuracy**
- GIVEN user A follows user B
- WHEN user A calls `GET /api/users/B.username`
- THEN `viewer_is_following: true` is returned
- AND after A unfollows B, the same call returns `viewer_is_following: false`

---

## REQ-02: Edit Own Profile

**Scenario REQ-02-A: Update display name and bio**
- GIVEN an authenticated user
- WHEN `PATCH /api/users/me` is called with updated `display_name` and/or `bio`
- THEN user record is updated, updated profile returned

**Scenario REQ-02-B: Unauthenticated update attempt**
- GIVEN no valid access token
- WHEN `PATCH /api/users/me` is called
- THEN 401 Unauthorized returned

---

## REQ-03: Upload Avatar

**Scenario REQ-03-A: Successful avatar upload**
- GIVEN an authenticated user and a valid image file (JPEG or PNG, ≤ 5 MB)
- WHEN `POST /api/users/me/avatar` is called with the file
- THEN file is saved, `avatar_url` on user record is updated, new URL returned

---

## REQ-04: Follow a User

**Scenario REQ-04-A: Successful follow**
- GIVEN authenticated user A, and user B (whom A does not follow)
- WHEN `POST /api/users/{username}/follow` is called
- THEN follow record is created, 201 returned
- AND a `follow` notification is created for user B
- AND `B.followers_count` increases by 1 on the next profile read

**Scenario REQ-04-B: Self-follow attempt**
- GIVEN authenticated user A
- WHEN `POST /api/users/A.username/follow` is called
- THEN 400 Bad Request returned, no follow record created

**Scenario REQ-04-C: Already following**
- GIVEN A already follows B
- WHEN `POST /api/users/B.username/follow` is called again
- THEN 409 Conflict returned, no duplicate record created

---

## REQ-05: Unfollow a User

**Scenario REQ-05-A: Successful unfollow**
- GIVEN authenticated user A following user B
- WHEN `DELETE /api/users/{username}/follow` is called
- THEN follow record is removed, 204 returned
- AND `B.followers_count` decreases by 1 on the next profile read

**Scenario REQ-05-B: Not following**
- GIVEN A does not follow B
- WHEN `DELETE /api/users/B.username/follow` is called
- THEN 404 Not Found returned

---

## REQ-06: Follower and Following Counts

Counts are computed via scalar subquery on every profile read. No stored counter columns.

```sql
followers_count = SELECT COUNT(*) FROM follows WHERE following_id = user.id
following_count = SELECT COUNT(*) FROM follows WHERE follower_id = user.id
```

This prevents count drift from concurrent follow/unfollow operations.

---

## REQ-07: Paginated Follower / Following Lists

**Scenario REQ-07-A: List followers**
- GIVEN user B with followers F1, F2, F3
- WHEN `GET /api/users/{username}/followers` is called
- THEN paginated list of follower user objects returned (cursor-based)

**Scenario REQ-07-B: List following**
- GIVEN user A following U1, U2, U3
- WHEN `GET /api/users/{username}/following` is called
- THEN paginated list of followed user objects returned (cursor-based)

---

## REQ-08: User Tweets

**Scenario REQ-08-A: View user's tweets**
- GIVEN a username
- WHEN `GET /api/users/{username}/tweets` is called
- THEN cursor-paginated list of that user's tweets returned (newest first)

