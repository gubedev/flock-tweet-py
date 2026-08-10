# Spec: Likes

**Status**: specified
**Phase**: FASE 6 — Likes (BE + FE + E2E)
**Created**: 2026-08-04

---

## Purpose

Allow authenticated users to like and unlike tweets. Like counts are maintained as stored columns updated atomically to avoid counting on every read. Each tweet response carries a `viewer_has_liked` field derived from a JOIN at query time.

---

## REQ-01: Like a Tweet

**Scenario REQ-01-A: Successful like**
- GIVEN an authenticated user and a tweet they have NOT yet liked
- WHEN `POST /api/tweets/{id}/like` is called
- THEN a like record is created, `tweet.likes_count` is incremented by 1 via atomic UPDATE, 201 returned
- AND a `like` notification is created for the tweet's author (if liker ≠ author)

**Scenario REQ-01-B: Already liked (idempotent)**
- GIVEN an authenticated user who has already liked tweet `T`
- WHEN `POST /api/tweets/{id}/like` is called again
- THEN 409 Conflict returned, `likes_count` is NOT incremented a second time

**Scenario REQ-01-C: Tweet not found**
- GIVEN a tweet ID that does not exist
- WHEN `POST /api/tweets/{id}/like` is called
- THEN 404 Not Found returned

**Scenario REQ-01-D: Self-like notification suppressed**
- GIVEN user A likes their own tweet
- WHEN like is processed
- THEN like record is saved and count increments, but NO notification is generated

---

## REQ-02: Unlike a Tweet

**Scenario REQ-02-A: Successful unlike**
- GIVEN an authenticated user who has liked tweet `T`
- WHEN `DELETE /api/tweets/{id}/like` is called
- THEN like record is removed, `tweet.likes_count` is decremented by 1 via atomic UPDATE, 204 returned

**Scenario REQ-02-B: Not liked (idempotent)**
- GIVEN an authenticated user who has NOT liked tweet `T`
- WHEN `DELETE /api/tweets/{id}/like` is called
- THEN 404 returned, count is NOT decremented

---

## REQ-03: likes_count Persistence

`likes_count` is stored as an integer column on the `tweets` table. It is maintained via atomic SQL UPDATEs, not computed by COUNT on every read.

```sql
-- On like:
UPDATE tweets SET likes_count = likes_count + 1 WHERE id = :tweet_id

-- On unlike:
UPDATE tweets SET likes_count = likes_count - 1 WHERE id = :tweet_id
```

This avoids a subquery on every timeline load while preventing race conditions from concurrent likes.

---

## REQ-04: viewer_has_liked

Every tweet response for an authenticated user includes `viewer_has_liked: boolean`.

- Computed via LEFT JOIN (or scalar subquery) against the `likes` table for the current user.
- Returned in: timeline, thread, profile tweets, search results.

**Scenario REQ-04-A: viewer_has_liked reflects current state**
- GIVEN user A likes tweet T, then unlikes it
- WHEN user A fetches the timeline
- THEN `viewer_has_liked: false` is returned for tweet T

---

## REQ-05: Optimistic Like on Frontend

Like/unlike mutations update the UI immediately without waiting for the server response.

**Scenario REQ-05-A: Optimistic update**
- GIVEN a user clicks the like button on tweet T in the timeline
- WHEN the mutation fires
- THEN `viewer_has_liked` toggles and `likes_count` adjusts immediately across all loaded timeline pages
- AND if the server returns an error, the previous state is restored (rollback)

