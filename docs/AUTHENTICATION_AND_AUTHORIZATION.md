# AUTHENTICATION_AND_AUTHORIZATION

Decisions: D-005, D-006, D-007, P-002, P-004, P-009, P-017, P-023. Roles: `USER_ROLES_AND_PERMISSIONS.md`. Threats: `SECURITY.md`.

## 1. Three kinds of principal
| Principal | Authenticates with | Used by |
|---|---|---|
| **User** (student or staff) | Google ID token → server session cookie | Student app, staff screens |
| **Entrance terminal** (device) | Per-terminal secret → HMAC-signed requests | `/terminal/*` |
| **System** | Runs in-process (jobs) | Automatic booking, expiry, reconciliation |

## 2. Google sign-in (server-side verification)
```text
Browser ──(Google Identity Services button)──► Google ──► ID token (JWT) ──► Browser
Browser ── POST /api/v1/auth/google { credential } ──► API
API: verify with Google's official library (google-auth-library):
     signature (Google certs) · iss = accounts.google.com · aud = our GOOGLE_CLIENT_ID · exp · email_verified = true
API: domain ∈ ALLOWED_EMAIL_DOMAINS?  → else 403 AUTH_DOMAIN_NOT_ALLOWED
API: Student with this email and status ACTIVE? → else 403 AUTH_NOT_ON_ROSTER
API: upsert User (googleSub, email), link Student.userId on first sign-in, ensure STUDENT role
API: create Session → Set-Cookie: bc_session=<random>; HttpOnly; Secure; SameSite=Lax; Path=/
```
- The client never sends email, role or student ID as trusted input.
- Staff accounts follow the same sign-in. Staff without a `Student` record are allowed when they hold at least one non-student role (assigned explicitly, D-007).

## 3. Sessions (P-004)
- Random 256-bit ID in the cookie; only its SHA-256 hash is stored in `Session`.
- Expiry (working default): 7 days absolute, 24 h idle. Rotated on sign-in; revoked on logout and on role removal.
- CSRF: `GET /auth/csrf` returns a token bound to the session; every POST/PUT/PATCH/DELETE must send `X-CSRF-Token`.
- CORS: allow-list of the web origin only, `credentials: true`.

**Why sessions instead of JWTs?** Revocation is instant (delete the row), nothing sensitive lives in the browser, and at ~500 users a database lookup per request is cheap. JWTs only shine when you need stateless verification across many services, which we don't have.

## 4. Authorisation
```text
request → authenticate (session | terminal HMAC) → requirePermission('x.y') → service checks ownership → data
```
- **Permission check** (route level): does this principal hold the permission? → 403.
- **Ownership check** (service level): is this row the caller's? → 404 (no existence leak).
- Permissions are loaded per request from `UserRole → RolePermission`; nothing is cached across requests.

## 5. Terminal authentication (P-009)
Headers: `X-Terminal-Id`, `X-Timestamp`, `X-Nonce`, `X-Signature = HMAC-SHA256(secret, method \n path \n timestamp \n nonce \n sha256(body))`.
Rejected when: unknown or inactive terminal, skew > ±60 s, nonce reused within 5 min, bad signature. The secret is shown once at registration or rotation. HMAC verification needs the raw key, so it is stored **encrypted** (AES-256-GCM with `TERMINAL_SECRET_KEY` from the environment), never hashed-only and never logged.

## 6. Bootstrap
`npm run cli -- create-main-admin --email <email>` (P-017) creates the first Main Admin. It is never exposed over HTTP.

## 7. Development sign-in
Local development and tests may use a **dev-only login stub** behind `AUTH_DEV_STUB=true`. Startup fails if this flag is set while `NODE_ENV=production`. The stub never ships enabled.

## 8. Tests
AC-AUTH-01…05, AC-RBAC-01, AC-HIST-01, AC-BIO-02 (HMAC). The Google library is mocked at its boundary in unit tests; integration tests use the dev stub.
