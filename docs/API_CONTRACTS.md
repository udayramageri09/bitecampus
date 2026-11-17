# API_CONTRACTS

> REST JSON under `/api/v1` (P-014). Proposed contracts, to be reconciled with existing routes found in the repository audit. Request bodies are validated with Zod; unknown fields are rejected.

## 1. Conventions
- **Auth**: user routes use the session cookie (P-004) plus a CSRF token header `X-CSRF-Token` on state-changing methods. Terminal routes use HMAC headers (§6).
- **Idempotency**: `Idempotency-Key: <uuid>` is required where marked **[idem]** (P-015).
- **Times**: ISO-8601 UTC instants (`2026-12-01T07:30:00Z`); campus dates `YYYY-MM-DD`. Display conversion to IST happens in the client.
- **Money**: integer paise (`7000` = ₹70).
- **IDs**: UUID. Another user's resource returns **404**, not 403 (no existence leak).
- **Pagination**: `?cursor=&limit=` (max 50) → `{ items, nextCursor }`.

### Error envelope
```json
{ "error": { "code": "CUTOFF_PASSED", "message": "Booking closed at 10:00 AM.", "requestId": "req_01H...", "details": {} } }
```

### Error codes (shared)
| HTTP | Codes |
|---|---|
| 400 | `VALIDATION_FAILED` |
| 401 | `UNAUTHENTICATED`, `AUTH_INVALID_TOKEN`, `TERMINAL_AUTH_FAILED` |
| 403 | `FORBIDDEN`, `AUTH_DOMAIN_NOT_ALLOWED`, `AUTH_NOT_ON_ROSTER`, `NOT_ELIGIBLE`, `STUDENT_NOT_ACTIVE`, `NO_ENTITLEMENT`, `OPTED_OUT`, `DEMO_COUPON_NOT_ACCEPTED` |
| 404 | `NOT_FOUND`, `MEAL_SERVICE_NOT_FOUND` |
| 409 | `BOOKING_EXISTS` (demo only), `ALREADY_REDEEMED`, `JOB_ALREADY_RUNNING`, `QR_ALREADY_USED`, `REQUEST_IN_PROGRESS`, `VERSION_CONFLICT`, `GATE_RETRY_NOT_ALLOWED` |
| 410 | `QR_EXPIRED` |
| 422 | `CUTOFF_PASSED`, `BOOKING_DISABLED`, `OPT_OUT_DISABLED`, `OUTSIDE_SERVICE_WINDOW`, `CONFIG_INCOMPLETE`, `IDEMPOTENCY_KEY_REUSED`, `QR_INVALID` |
| 429 | `RATE_LIMITED` |
| 503 | `SERVICE_UNAVAILABLE` |

## 2. Auth & profile
| Method & path | Body → Response | Permission |
|---|---|---|
| `POST /auth/google` | `{ credential }` → `200 { user, roles, permissions }` + Set-Cookie | public, rate-limited |
| `POST /auth/logout` | → `204` | session |
| `GET /me` | → `{ user, roles, permissions, student: { id, status, membership: { messId, validFrom, validTo, status } \| null } \| null }` | session |
| `GET /auth/csrf` | → `{ csrfToken }` | session |

## 3. Meals, bookings, coupons, history (student)
Hostellers have **no booking-creation endpoint**: bookings are created automatically (D-034). The student identity is always the `Student` linked to the session user (P-023).

| Method & path | Body → Response | Permission |
|---|---|---|
| `GET /meal-services?from=&to=` | → `[{ id, mealType, serviceDate, windowStartAt, windowEndAt, cutoffAt, listPricePaise\|null, bookingEnabled, optOutEnabled, menu:[...], myBooking:{ id, status, fundingSource, amountPayablePaise, coupon:{ id, status, voidReason } }\|null, canOptOut }]` (`canOptOut` is computed by the server) | `meal.read` |
| `GET /meal-services/:id` | → same, single | `meal.read` |
| `POST /bookings/:id/opt-out` **[idem]** | → `200 { booking:{ status:"OPTED_OUT" }, coupon:{ status:"VOID", voidReason:"OPTED_OUT" } }` · `422 OPT_OUT_DISABLED` · `422 CUTOFF_PASSED` · `409 ALREADY_REDEEMED` · `404` (not own). Opting out of an already opted-out booking returns `200` with the current state | `booking.optout.own` |
| `GET /bookings/me?from=&to=` | → paged meal history (each auto booking with its outcome) | `booking.read.own` |
| `GET /coupons/me?status=` | → `[{ id, mealServiceId, mealType, serviceDate, window, status, voidReason, fundingSource, isDemo, redemption:{ redeemedAt, method, entry:"CONFIRMED"\|"PENDING"\|"FAILED" }\|null }]` | `coupon.read.own` |
| `GET /redemptions/me` | → paged redemption history with gate outcome | `redemption.read.own` |
| `POST /feedback` | `{ mealServiceId, rating, comment? }` → `201` | `feedback.create.own` |

## 4. Entrance — student side
### `POST /entry/qr-redemptions` **[idem]** — permission `entry.qr.redeem.self`
Request:
```json
{ "challengeToken": "BC1.q8Z...base64url" }
```
The student identity comes **only** from the session. The body carries no user, coupon or meal ID.

Responses:
```json
200 { "outcome": "REDEEMED", "redemptionId": "…", "mealType": "LUNCH", "serviceDate": "2026-12-01",
      "entry": { "status": "CONFIRMED" | "PENDING" | "FAILED", "gateCommandId": "…" } }
```
`410 QR_EXPIRED` · `409 QR_ALREADY_USED` · `422 QR_INVALID` · `403 STUDENT_NOT_ACTIVE` · `403 NOT_ELIGIBLE` · `403 OPTED_OUT` · `403 NO_ENTITLEMENT` · `403 DEMO_COUPON_NOT_ACCEPTED` · `422 OUTSIDE_SERVICE_WINDOW` · `409 ALREADY_REDEEMED` (with `details.redemptionId` and `details.entry`) · `429 RATE_LIMITED`.

### `GET /entry/redemptions/:id` — own redemption status (for polling `entry.status`).
### `POST /entry/redemptions/:id/gate-retry` **[idem]** — own redemption, or `gate.retry` for staff
→ `202 { gateCommandId, attemptNo }` · `409 GATE_RETRY_NOT_ALLOWED` (latest command already `ACKNOWLEDGED_OPEN`, or outside the retry window, Q-09).

## 5. Entrance — terminal side (device principal, HMAC)
| Method & path | Body → Response |
|---|---|
| `POST /terminal/qr-challenges` | → `201 { challengeId, payload: "BC1.<token>", expiresAt, rotateAfter }` (the raw token is returned once and never stored) |
| `GET /terminal/events?since=` | → `[{ type: "CHALLENGE_CONSUMED"\|"ENTRY_RESULT", challengeId?, outcome, displayName?: "first name only", mealType }]` |
| `POST /terminal/biometric-events` | `{ eventId, subjectRef?, outcome, occurredAt }` → `200 { outcome: "REDEEMED"\|"REJECTED"\|"FALLBACK_TO_QR", reasonCode?, entry? }` — idempotent on `eventId`. **Returns 404 when the biometric simulator is disabled** (live pilot, D-039) |
| `POST /terminal/heartbeat` | → `204` |

Showing a first name on the terminal is PROPOSED and pending a privacy decision (Q-18). The default is to show no name.

## 6. Terminal authentication headers (P-009)
`X-Terminal-Id`, `X-Timestamp` (unix seconds), `X-Nonce` (random 128-bit), `X-Signature` = `base64(HMAC-SHA256(secret, method\npath\ntimestamp\nnonce\nsha256(body)))`. The server rejects requests with skew over ±60 s or a reused nonce (stored 5 min).

## 7. Staff / admin
| Method & path | Purpose | Permission |
|---|---|---|
| `GET/PUT /admin/meal-slots/:mealType` | slot config; PUT validates DR-C1 | `mealslot.config.manage` |
| `PATCH /admin/meal-services/:id` | menu items; `If-Match: <version>` (services themselves are generated, P-024) | `mealservice.manage` |
| `POST /admin/auto-booking/runs` | `{ }` → `202 { jobRunId }` · `409 JOB_ALREADY_RUNNING`; always idempotent in effect | `autobooking.run` |
| `GET /admin/auto-booking/runs` | run history with counts and errors | `ops.dashboard.read` |
| `POST /admin/meal-services/:id/void` | `{ reason }` | `mealservice.manage` |
| `PUT /admin/meal-services/:id/planned-quantity` | `{ quantity, reason }` + `If-Match` | `kitchen.plan.adjust` |
| `GET /admin/ops/meal-services/:id` | `{ planned, autoBooked, optedOut, voided, redeemed, unredeemed, byMethod, failuresByReason, integrity: { bookingsWithoutCoupon, eligibleWithoutBooking } }` | `ops.dashboard.read` |
| `GET /admin/ops/entrance?checkpointId=` | recent attempts, queue size | `ops.dashboard.read` |
| `GET /admin/reconciliation` / `POST /admin/reconciliation/:gateCommandId/resolve` | `{ resolution: "OPENED"\|"NOT_OPENED", note }` | view: `gate.retry`; resolve: `reconciliation.resolve` |
| `GET /admin/entry/students?query=` | search by roster ID/email → `{ studentId, displayName, currentEntitlement }` only | `entry.assist` |
| `POST /admin/entry/assisted-redemptions` **[idem]** | `{ studentId, checkpointId, reasonCode }` (D-040; identity check Q-10) | `entry.assist` |
| `POST /admin/roster/imports?dryRun=true\|false` | CSV upload → report (students added/updated, memberships started/ended, errors) | `roster.import` |
| `POST /admin/memberships/:id/end` · `/suspend` · `/reinstate` | `{ validTo?, reason }` | `roster.manage` |
| `POST /admin/students/:id/memberships` | allocation change: `{ messId, validFrom }` (ends the current one) | `roster.manage` |
| `GET/POST/DELETE /admin/users/:id/roles` | grant/revoke | `role.assign.*` |
| `POST /admin/terminals`, `POST /admin/terminals/:id/rotate-credential` | returns the secret once | `terminal.manage` |
| `GET /admin/audit?entityType=&entityId=&actorId=&from=&to=` | paged | `audit.read` |
| `GET /admin/feedback?mealServiceId=` | paged | `feedback.read` |
| `PUT /admin/simulators/gate` · `PUT /admin/simulators/biometric` | `{ mode }` — 404 unless `DEMO_MODE=true` | `simulator.control` |

## 8. Demo payments (`DEMO_MODE=true` only, otherwise 404)
| Method & path | Body → Response |
|---|---|
| `POST /demo/bookings` **[idem]** | `{ mealServiceId }` → `201 { booking:{status:"PENDING_PAYMENT", fundingSource:"SIMULATED_PAYMENT"} }`; demo (non-hosteller) students only, otherwise `403 NOT_ELIGIBLE` |
| `POST /demo/payments` **[idem]** | `{ bookingId, outcome: "SUCCEEDED"\|"FAILED"\|"CANCELLED"\|"PENDING" }` → `200 { attempt, booking, coupon?: { isDemo:true } }` |

## 9. Health
`GET /health` (liveness, no DB) · `GET /ready` (DB reachable, migrations applied) — both unauthenticated, no internal details.

## 10. Rate limits (initial, configurable, PROPOSED)
`/auth/google`: 10/min per IP · `/entry/qr-redemptions`: 10/min per user · `/bookings/:id/opt-out`: 20/min per user · `/terminal/*`: 120/min per terminal · admin writes: 60/min per user.
