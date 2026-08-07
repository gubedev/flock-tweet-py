# Spec: Search

**Status**: specified
**Phase**: FASE 7 — Search (BE + FE + E2E)
**Created**: 2026-08-04

---

## Purpose

Allow authenticated users to find other users by typing a partial username or display name. Results include accurate follower counts computed from the follows table.

---

## REQ-01: User Search

**Scenario REQ-01-A: Partial username match**
- GIVEN the query string "ali"
- WHEN `GET /api/search/users?q=ali` is called
- THEN all users whose `username` or `display_name` contains "ali" (case-insensitive) are returned

**Scenario REQ-01-B: Case-insensitive match**
- GIVEN a user with username "Alice"
- WHEN the query is "alice" or "ALICE"
- THEN the user appears in results

**Scenario REQ-01-C: No results**
- GIVEN a query string that matches no user
- WHEN `GET /api/search/users?q=zzz` is called
- THEN an empty list is returned (not 404)

**Scenario REQ-01-D: Empty query**
- GIVEN an empty `q` parameter or missing `q`
- WHEN `GET /api/search/users` is called
- THEN an empty list is returned (no full-table scan)

**Scenario REQ-01-E: Unauthenticated**
- GIVEN no valid access token
- WHEN `GET /api/search/users?q=ali` is called
- THEN 401 Unauthorized returned

---

## REQ-02: Accurate Follower Counts in Results

Each user in search results includes `followers_count` computed from the `follows` table via scalar subquery — never hardcoded or from a stale cached column.

**Scenario REQ-02-A: followers_count correctness**
- GIVEN user B has 5 followers
- WHEN `GET /api/search/users?q=B.username` is called
- THEN the result for B shows `followers_count: 5`

**Scenario REQ-02-B: followers_count after follow**
- GIVEN user C previously had 0 followers, and user A just followed C
- WHEN the search is repeated
- THEN the result for C shows `followers_count: 1`

---

## REQ-03: Frontend Search UX

**Scenario REQ-03-A: Debounced input**
- GIVEN the user types in the search input
- WHEN the user pauses for 400 ms
- THEN `GET /api/search/users` is called with the current query

**Scenario REQ-03-B: No request on empty input**
- GIVEN the search input is empty
- WHEN the component renders
- THEN no API call is made

