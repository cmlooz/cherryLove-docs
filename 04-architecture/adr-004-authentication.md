---
id: ADR-004
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-004 — Authentication

Implements: REQ-N-SEC-03, REQ-N-SEC-04, REQ-N-SEC-07, REQ-F-AUTH-02, REQ-F-AUTH-07, REQ-F-AUTH-08,
REQ-F-AUTH-09, REQ-F-ACC-03, REQ-F-ACC-06, DEC-005, DEC-017.
Builds on: C4-001 rule 4 (staff and partners share the API process, not the auth path), ADR-001 (`identity`).

## Context

REQ-N-SEC-03 already fixes the shape: short-lived JWT access, rotating refresh with blacklist, web refresh in
an `HttpOnly` + `Secure` + `SameSite` cookie, mobile refresh in OS secure storage. Still open: the library,
token lifetimes, how web and mobile differ on the wire, how revocation works, and how invites and password
recovery tokens are built.

## Decision drivers

1. No hand-rolled crypto or token parsing.
2. A stolen access token is useful for minutes, not days.
3. JavaScript can never read the web refresh token (XSS cannot steal a long-lived session).
4. Revocation for password change, logout and archived couples (REQ-F-ACC-03, REQ-F-AUTH-09).
5. Few dependencies, all behind ports (ADR-001).

## Options considered

| Option | Verdict |
|--------|---------|
| O1 — Hand-rolled JWT with PyJWT | ❌ We would own rotation, blacklist, and every edge case. Violates driver 1. |
| O2 — `djangorestframework-simplejwt` + its blacklist app ⭐ | ✅ Rotation + blacklist built in, maintained under the jazzband org, DRF-native. One dependency. |
| O3 — `dj-rest-auth` + `django-allauth` | ❌ Brings registration/social flows we do not have (no self-service, DEC-028); many transitive deps. |
| O4 — Django sessions for web, tokens for mobile | ❌ Contradicts REQ-N-SEC-03; two auth models in clients and core. |

## Decision

**O2**, wrapped behind a `TokenIssuer` port in `identity/application/ports.py`; SimpleJWT lives only in
`identity/infrastructure/adapters/`.

### 1. Tokens

| Token | Lifetime | Storage (web) | Storage (mobile) |
|-------|----------|---------------|------------------|
| Access (JWT, `HS256`, dedicated signing key ≠ `SECRET_KEY`) | **10 min** | JS memory only (never `localStorage`) | memory |
| Refresh (rotating, blacklisted after use) | **30 days** | `HttpOnly; Secure; SameSite=Strict; Path=/api/v1/auth/` cookie | `expo-secure-store` (Keychain / Keystore) |

Access claims are minimal: `sub` (user id), `exp`, `iat`, `jti`. **No `couple_id` claim**: the couple is
resolved server-side on every request (ADR-005, REQ-N-SEC-01).

Why `HS256`: one issuer and one verifier in the same process. Asymmetric keys (RS256/EdDSA) become the seam
if a module is ever split out.

### 2. Endpoints and web vs mobile

| Endpoint | Web | Mobile |
|----------|-----|--------|
| `POST /api/v1/auth/login` (`email`, `password`, `client: "web" \| "mobile"`) | access in body, refresh **set as cookie** (never in body) | access + refresh in body |
| `POST /api/v1/auth/refresh` | reads cookie, rotates cookie | reads body, returns new pair |
| `POST /api/v1/auth/logout` | blacklists refresh, clears cookie | blacklists refresh |

CSRF on the cookie endpoints: `SameSite=Strict` **plus** a required custom header (`X-CherryLove-Client: web`),
which a cross-site form cannot send. CORS allows only the web origin, with credentials.

**Constraint for the production ADR:** web and API must be served from the same site (same registrable
domain, e.g. `app.` and `api.` of one domain). Otherwise browsers treat the cookie as third-party and drop it.

### 3. Revocation

| Event | Effect | Requirement |
|-------|--------|-------------|
| Logout | Current refresh blacklisted. | REQ-F-ACC-06 |
| Password change | **All** outstanding refresh tokens of the user blacklisted except the current session's. | REQ-F-ACC-03 |
| Couple archived / user disabled | Refresh rejected; login rejected. | REQ-F-AUTH-09 |
| Reuse of an already-rotated refresh | Treated as theft: **all** the user's refresh tokens revoked; audit event. | driver 3 |

Access tokens are not checked against a blacklist: after revocation they live at most **10 min**. Accepted
trade-off (checking every request would add a DB/Redis hit to REQ-N-PERF-01 budgets).
Reuse detection is ~1 small adapter on top of SimpleJWT, not a built-in feature; plan F gives it its own step.

### 4. Passwords, invites, recovery

| Item | Decision | Dependency |
|------|----------|------------|
| Hashing | Django `Argon2PasswordHasher` first in `PASSWORD_HASHERS` | `argon2-cffi` (required by Django for Argon2) |
| Invite token | 32 random bytes (`secrets`), stored **hashed** (SHA-256), single-use, 72 h, revocable/resendable | none |
| Recovery token | Django `PasswordResetTokenGenerator` (HMAC, invalidated by password change), `PASSWORD_RESET_TIMEOUT` = **30 min**; identical response whether the email exists | none |
| Rate limits | DRF `ScopedRateThrottle` on Redis cache: login 5/min per IP+email, recovery 3/h per email, invite accept 10/h per IP | none (DRF built-in) |

### 5. Staff

Staff use **Django sessions** in Django Admin (DEC-005), never JWT. A staff account has no couple and cannot
obtain partner tokens; a partner account is never `is_staff`. Staff 2FA is **not** in the frozen
requirements → proposed as a new open question, not decided here.

### 6. Logging

Every login, logout, refresh-reuse and failed attempt writes an audit event (REQ-F-AUTH-07, REQ-F-LOG-01)
with `request_id`, IP and device; failed attempts never store the submitted password.

## Consequences

- ✅ XSS cannot steal the web session; a leaked access token expires in 10 min.
- ✅ One auth model in `cherrylove-core`; only the transport adapter differs (cookie vs secure store, ADR-002 `platform/`).
- ✅ Two new dependencies, each justified: `djangorestframework-simplejwt`, `argon2-cffi`. Mobile adds
  `expo-secure-store` (Expo SDK module). Versions pinned in ADR-016.
- ❌ Up to 10 min of access after revocation.
- ❌ Web/API same-site constraint narrows production hosting choices.
- ⚠️ Web keeps the access token in memory → a page reload calls `/auth/refresh` once at startup.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-005 (couple-scoped authorization).
