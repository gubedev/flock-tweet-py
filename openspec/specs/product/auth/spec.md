# Spec: Authentication

**Status**: specified
**Phase**: FASE 3 — Auth (BE + FE + E2E)
**Created**: 2026-08-04

---

## Purpose

Enable users to create accounts, authenticate, and maintain sessions securely across page reloads. Supports token-based auth with access tokens in memory and refresh tokens in httpOnly cookies.

---

## REQ-01: Registration

Users can create an account with a unique email, unique username, display name, and password.

**Scenario REQ-01-A: Successful registration**
- GIVEN a unique email, unique username, and password ≥ 8 characters
- WHEN `POST /api/auth/register` is called
- THEN account is created, access token returned in response body, refresh token set as httpOnly cookie

**Scenario REQ-01-B: Duplicate email**
- GIVEN an email already registered
- WHEN `POST /api/auth/register` is called with that email
- THEN 409 Conflict returned, no account created

**Scenario REQ-01-C: Duplicate username**
- GIVEN a username already taken
- WHEN `POST /api/auth/register` is called with that username
- THEN 409 Conflict returned, no account created

**Scenario REQ-01-D: Password too short**
- GIVEN a password with fewer than 8 characters
- WHEN `POST /api/auth/register` is called
- THEN 422 Unprocessable Entity returned with validation error

---

## REQ-02: Login

Authenticated users receive a short-lived access token and a long-lived refresh cookie.

**Scenario REQ-02-A: Successful login**
- GIVEN a registered user with valid credentials
- WHEN `POST /api/auth/login` is called with email + password
- THEN 200 returned with `access_token` (15 min JWT) in body and `refresh_token` (7 day JWT) as httpOnly cookie

**Scenario REQ-02-B: Wrong password**
- GIVEN a registered email and incorrect password
- WHEN `POST /api/auth/login` is called
- THEN 401 Unauthorized returned with generic error (no info leakage)

**Scenario REQ-02-C: Unknown email**
- GIVEN an email that does not exist
- WHEN `POST /api/auth/login` is called
- THEN 401 Unauthorized returned with the same generic error as REQ-02-B

---

## REQ-03: Token Refresh

When the access token expires, the client silently re-authenticates via the refresh cookie.

**Scenario REQ-03-A: Valid refresh cookie**
- GIVEN an expired access token and a valid refresh cookie
- WHEN `POST /api/auth/refresh` is called (credentials: "include")
- THEN new access token returned in response body; refresh cookie rotated

**Scenario REQ-03-B: Missing or expired refresh cookie**
- GIVEN no valid refresh cookie
- WHEN `POST /api/auth/refresh` is called
- THEN 401 Unauthorized returned; client must redirect to login

**Scenario REQ-03-C: Automatic retry in API client**
- GIVEN any API request returns 401
- WHEN the `api-client.ts` receives that response and `isRetry` is false
- THEN it calls `POST /api/auth/refresh`, then retries the original request once with `isRetry = true`
- AND if the refresh also fails, it throws `UnauthorizedError` to trigger a login redirect

---

## REQ-04: Logout

Clears the refresh cookie and invalidates client-side session state.

**Scenario REQ-04-A: Successful logout**
- GIVEN an authenticated user
- WHEN `POST /api/auth/logout` is called
- THEN refresh cookie is cleared (max-age=0), client discards access token from memory

---

## REQ-05: Get Current User

Returns the authenticated user's profile data for session restoration.

**Scenario REQ-05-A: Valid token**
- GIVEN a valid Bearer access token
- WHEN `GET /api/auth/me` is called
- THEN 200 with user id, username, email, display_name, avatar_url returned

**Scenario REQ-05-B: Invalid or missing token**
- GIVEN no token or an expired token
- WHEN `GET /api/auth/me` is called
- THEN 401 Unauthorized returned

**Implementation note**: this endpoint MUST go through `GetCurrentUserUseCase`, not inject `SqlUserRepository` directly into the router.

---

## REQ-06: Cookie Security

Refresh token cookie is configured for XSS and CSRF resistance.

| Attribute | Value |
|---|---|
| `httpOnly` | true |
| `secure` | true in staging/production; false in development |
| `samesite` | lax |
| `max_age` | 7 days |

---

## REQ-07: Session Persistence Across Page Reloads

Users remain logged in after closing and reopening the browser tab.

**Scenario REQ-07-A: Page reload with valid refresh cookie**
- GIVEN a user who previously logged in (refresh cookie set)
- WHEN they reload the page
- THEN the frontend calls `GET /api/auth/me` (which triggers auto-refresh if needed)
- AND session is restored without requiring the user to log in again

