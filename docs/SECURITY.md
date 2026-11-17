# SECURITY

Baseline for every release (NFR-07). This is not a substitute for a formal penetration test; never claim one was done.

## 1. Assets
Student identity and roster data; sessions; role assignments; coupons (meal entitlements); redemption and audit records; terminal credentials; configuration and secrets.

## 2. Trust boundaries
Browser/phone (untrusted) → API · Entrance terminal (semi-trusted device; can be stolen or tampered with) → API · API → PostgreSQL · API → Google (token keys) · API → gate controller (simulated in V1).

## 3. Threat model (STRIDE-style, pilot)
| # | Threat | Example | Mitigation | Test |
|---|---|---|---|---|
| T1 | Spoofed identity | Forged Google token; client sends `email` or `role` | Verify with Google's official library (iss, aud, exp, signature, `email_verified`); never accept identity fields from the body | AC-AUTH-01 |
| T2 | Privilege escalation | Domain user claims staff | Roles only via `role.assign.*`; default deny; Main Admin bootstrap via CLI | AC-AUTH-04, AC-RBAC-01 |
| T3 | IDOR | `GET /bookings/<other id>` | Ownership filter in service queries; 404 on mismatch | AC-HIST-01 |
| T4 | QR replay | Photo of the entrance QR reused later | 30 s TTL, single-use conditional consume, terminal binding | AC-QR-03/04/06 |
| T5 | Remote QR relay | Student A at the gate sends a QR photo to friend B, who redeems **B's** coupon so A can enter | Short TTL; consumption shows the redeeming student's result on the terminal; gate opens once; staff oversight. **Residual risk accepted for pilot**; proximity checks (geolocation/BLE) are unreliable in browsers and deferred | M |
| T6 | Double redemption | Concurrent QR + biometric; retries | UNIQUE(couponId) + conditional update + idempotency | AC-RED-01/03 |
| T7 | Forged terminal events | Attacker posts a MATCH event | Per-terminal HMAC, ±60 s skew, nonce uniqueness, terminal active flag, rotation on theft | AC-BIO-02 |
| T8 | Stolen terminal | Device taken | Revoke via `terminal.manage`; credential rotation; terminal can issue challenges but cannot redeem without a student session or signed event | M |
| T9 | CSRF | Cross-site POST with the session cookie | SameSite=Lax cookie + CSRF token header on state-changing requests; strict CORS allow-list | I |
| T10 | XSS | Menu text or feedback comment with a script | React escaping; no `dangerouslySetInnerHTML`; CSP header; server-side length limits | I/E |
| T11 | Injection | SQL via filters | Prisma parameterisation; raw SQL only parameterised and reviewed | review |
| T12 | Brute force / abuse | Token guessing, scanning spam | 256-bit tokens; rate limits (API_CONTRACTS §10) | I |
| T13 | Sensitive-data exposure | Tokens, emails in logs | Log redaction list; no PII in QR, URL or analytics; generic error messages | review |
| T14 | Audit tampering | Admin deletes evidence | INSERT/SELECT-only grant on AuditEvent | AC-AUDIT-01 |
| T15 | Demo leakage into pilot | Simulated coupon or simulated biometric accepted for real | `DEMO_MODE=false` and biometric simulator disabled in pilot; `isDemo` coupons rejected by pilot checkpoints | AC-DEMO-01/04, AC-BIO-06 |
| T16 | Fail-open entrance | API outage grants entry | Never grant on error; manual process (NFR-06) | M |
| T18 | Staff-assisted abuse | Counter Staff redeem for absent friends | Permission-gated; mandatory reason code; audit with actor; ops view of assisted-entry rate per staff member; identity check per Q-10 | AC-STA-01/02 |
| T19 | Opt-out abuse or forgery | One student opts another out | Ownership check in service; 404 on mismatch; idempotent; audited | AC-OO-05 |
| T20 | Membership tampering | Unauthorised edit grants meals to non-residents | `roster.manage` only; audited; DB exclusion constraint against overlapping active memberships | AC-MEM-01 |
| T17 | Dependency vulnerabilities | Vulnerable npm package | `npm audit` in CI (review findings, no auto-fix upgrades); lockfile committed | CI |

## 4. Controls checklist
**Authentication & sessions**
- Session ID is 256-bit random, stored hashed. Cookie flags: `HttpOnly; Secure` (production); `SameSite=Lax`; `Path=/`. Idle and absolute expiry (values PROPOSED: 7 days absolute, 24 h idle).
- Rotate the session ID on sign-in; revoke it on logout and on role removal.

**Authorisation**
- `requirePermission` on every protected route plus ownership checks in services.
- Staff UI routes are hidden on the client and enforced on the server.

**Input & output**
- Zod schemas with `.strict()`; size limits on bodies and CSV uploads.
- Security headers: CSP, `X-Content-Type-Options`, `Referrer-Policy`, HSTS (production), frame-ancestors none (terminal is not embedded).

**Secrets**
- `.env` never committed; `.env.example` contains placeholders only; terminal secrets and session secrets come from environment/secret store; secrets are never printed in logs or by Claude Code.

**Privacy**
- No biometric data (D-018, D-029). Synthetic data outside an authorised pilot (D-030). Data minimisation in feedback and audit (DR-A3). Retention policy pending (Q-18).

**Operations**
- HTTPS everywhere (also required for the camera API). Backups with tested restore (NFR-15). Least-privilege DB roles (migration role vs app role).

## 5. Security review triggers
Use the security-reviewer agent before merging any change to: authentication/session, RBAC, automatic booking/opt-out/membership, entry/redemption/gate, terminal auth, roster import, audit, configuration loading, or new dependencies.
