# Spec: Tweets

**Status**: specified
**Phase**: FASE 5 — Tweets (BE + FE + E2E); image upload in FASE 9
**Created**: 2026-08-04

---

## Purpose

Allow authenticated users to publish short text posts (optionally with one image), view a chronological feed of content from accounts they follow, reply to tweets to form threads, and delete their own content.

---

## REQ-01: Create Tweet

**Scenario REQ-01-A: Valid tweet**
- GIVEN an authenticated user and content between 1 and 280 characters
- WHEN `POST /api/tweets` is called
- THEN tweet is persisted with `author_id`, `created_at`, `likes_count=0`, `replies_count=0`, `parent_id=null`
- AND tweet data is returned with author fields (username, display_name, avatar_url)

**Scenario REQ-01-B: Empty content**
- GIVEN content is an empty string or whitespace only
- WHEN `POST /api/tweets` is called
- THEN 422 Unprocessable Entity returned

**Scenario REQ-01-C: Content exceeds limit**
- GIVEN content exceeds `MAX_TWEET_LENGTH` (default 280, configured via env)
- WHEN `POST /api/tweets` is called
- THEN 422 Unprocessable Entity returned

**Scenario REQ-01-D: Unauthenticated**
- GIVEN no valid access token
- WHEN `POST /api/tweets` is called
- THEN 401 Unauthorized returned

---

## REQ-02: Delete Tweet

**Scenario REQ-02-A: Owner deletes own tweet**
- GIVEN an authenticated user who is the author of tweet `T`
- WHEN `DELETE /api/tweets/{id}` is called
- THEN tweet is removed from DB, 204 No Content returned

**Scenario REQ-02-B: Non-owner attempts delete**
- GIVEN an authenticated user who is NOT the author of tweet `T`
- WHEN `DELETE /api/tweets/{id}` is called
- THEN 403 Forbidden returned, tweet is NOT deleted

**Scenario REQ-02-C: Tweet not found**
- GIVEN a tweet ID that does not exist
- WHEN `DELETE /api/tweets/{id}` is called
- THEN 404 Not Found returned

---

## REQ-03: Timeline

Cursor-based chronological feed containing the authenticated user's own tweets and tweets from accounts they follow. Replies are excluded from the timeline.

Response shape:
```json
{ "tweets": [...], "next_cursor": "2026-08-04T12:00:00Z" }
```
`next_cursor` is `null` when there are no more pages. Default page size: 20 tweets.

**Scenario REQ-03-A: Default timeline**
- GIVEN an authenticated user following users A and B
- WHEN `GET /api/tweets/timeline` is called without a cursor
- THEN the 20 most recent tweets from user, A, and B are returned (no replies)
- AND response includes `next_cursor` (ISO timestamp of oldest tweet in page) or `null` if no more pages

**Scenario REQ-03-B: Cursor pagination**
- GIVEN a `cursor` query param (ISO timestamp)
- WHEN `GET /api/tweets/timeline?cursor=<timestamp>` is called
- THEN tweets with `created_at < cursor` are returned, enabling stable infinite scroll

**Scenario REQ-03-C: Own tweets included**
- GIVEN a user with no followers and their own tweets
- WHEN `GET /api/tweets/timeline` is called
- THEN own tweets appear in the feed (not an empty timeline)

**Scenario REQ-03-D: Replies excluded**
- GIVEN tweets that are replies (`parent_id IS NOT NULL`)
- WHEN `GET /api/tweets/timeline` is called
- THEN those tweets do NOT appear in the feed

---

## REQ-04: Tweet Detail (with Thread)

Returns a single tweet and its direct replies in chronological order. This is the canonical endpoint for viewing a tweet and its thread — there is no separate `/thread` path.

**Scenario REQ-04-A: Thread view**
- GIVEN tweet `T` with replies R1, R2, R3
- WHEN `GET /api/tweets/{id}` is called
- THEN response contains `tweet: T` and `replies: [R1, R2, R3]` sorted oldest-first

**Scenario REQ-04-B: Tweet not found**
- GIVEN an ID that does not exist
- WHEN `GET /api/tweets/{id}` is called
- THEN 404 Not Found returned

---

## REQ-05: Reply to Tweet

Replies are created via a dedicated endpoint. Replies appear in threads but not in the main timeline.

**Scenario REQ-05-A: Valid reply**
- GIVEN an authenticated user and an existing tweet `T`
- WHEN `POST /api/tweets/{id}/replies` is called with valid content
- THEN reply is persisted with `parent_id = T.id`, `T.replies_count` is incremented atomically
- AND if the replier is NOT the author of `T`, a `reply` notification is created for `T.author_id`

**Scenario REQ-05-B: Self-reply (no notification)**
- GIVEN user U replying to their own tweet
- WHEN `POST /api/tweets/{id}/replies` is called
- THEN reply is saved and `replies_count` increments, but NO notification is generated

**Scenario REQ-05-C: Parent tweet not found**
- GIVEN a tweet ID that does not exist
- WHEN `POST /api/tweets/{id}/replies` is called
- THEN 404 Not Found returned

---

## REQ-06: Image Attachment

Tweets can optionally include one image. The image is uploaded first, then the returned URL is included in the tweet creation request.

Upload constraints (enforced on `POST /api/uploads/image`):
- Max size: **10 MB**
- Allowed extensions: `jpg`, `jpeg`, `png`, `gif`, `webp`
- Filename stored as UUID to prevent collisions

**Scenario REQ-06-A: Tweet with image**
- GIVEN an authenticated user who has uploaded an image via `POST /api/uploads/image`
- WHEN `POST /api/tweets` includes `image_url` pointing to the uploaded file
- THEN tweet is saved with `image_url` and the URL is returned in all tweet responses

**Scenario REQ-06-B: Tweet without image**
- GIVEN no `image_url` in the request body
- WHEN `POST /api/tweets` is called
- THEN `image_url` is `null` in the saved tweet

**Scenario REQ-06-C: Invalid upload rejected**
- GIVEN a file with unsupported extension (e.g., `.pdf`) or size exceeding 10 MB
- WHEN `POST /api/uploads/image` is called
- THEN 400 Bad Request returned, no file saved

