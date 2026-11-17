# SYSTEM_DESIGN

How the architecture behaves at runtime: load, concurrency, idempotency, failure handling. Entry-specific sequences → `BIOMETRIC_QR_REDEMPTION_DESIGN.md`.

## 1. Load model (estimates, not measurements)
- About 500 users. The worst case is all hostellers arriving at one meal start. If 300 arrive in 30 minutes, that is roughly 10 redemptions per minute on average, with short bursts of a few per second at one terminal.
- Each redemption is about 6 SQL statements in one transaction, which is far below a single small PostgreSQL instance's capacity.
- Automatic booking: about 500 students × 3 meals × 3 horizon days ≈ 4,500 candidate rows per run, almost all no-ops after the first run.
- Conclusion: one API instance and one database are sufficient. The design risk is **correctness under concurrency and retries**, not throughput. Verify with NFR-02 load tests before claiming anything.

## 2. Concurrency strategy
| Operation | Race | Guard |
|---|---|---|
| Automatic booking | Overlapping runs; retries after a crash; safety net at entry | Full UNIQUE(studentId, mealServiceId) + UNIQUE(bookingId) on coupons + `INSERT … ON CONFLICT DO NOTHING`; advisory lock only to avoid wasted work (DR-AB4/AB5) |
| Opt-out vs entry | Student opts out while a scan is in flight (only possible before cutoff, so before the window) | Both conditional on coupon `ACTIVE`; first commit wins |
| Demo booking | Double tap | UNIQUE(studentId, mealServiceId) + idempotency record |
| Redeem coupon | QR + biometric at once; two phones; retries | `UNIQUE(Redemption.couponId)` + conditional `UPDATE MealCoupon SET status='REDEEMED' WHERE id=$1 AND status='ACTIVE'` (row count must be 1) in the same transaction |
| Consume QR challenge | Two students scan one QR | Conditional `UPDATE QrChallenge SET status='CONSUMED' ... WHERE tokenHash=$1 AND status='ISSUED' AND expiresAt > now()`; row count 0 → `QR_ALREADY_USED`/`QR_EXPIRED` |
| Gate retry | Student and staff retry together | `UNIQUE(GateCommand.redemptionId, attemptNo)`; retry allowed only when the latest command is `FAILED` or `UNKNOWN` |
| Planned-quantity edit | Two editors | Optimistic concurrency: `version` column; mismatch → 409 |

Isolation level: PostgreSQL default **READ COMMITTED** is enough, because every guard is a single-row conditional update or a unique constraint that serialises the race at the database. Unique violations (Prisma `P2002`) are translated to domain errors, never surfaced as 500s.

## 3. Idempotency
- Header `Idempotency-Key` (UUID v4, client-generated per user action) on: `POST /bookings/:id/opt-out`, `POST /entry/qr-redemptions`, `POST /entry/redemptions/:id/gate-retry`, `POST /admin/entry/assisted-redemptions`, `POST /demo/bookings`, `POST /demo/payments` (P-015).
- The generator needs no key: its natural key is (studentId, mealServiceId), enforced by the database.
- `IdempotencyRecord(principalId, route, key)` is unique and stores `requestHash`, `status`, `responseCode`, `responseBody` and `expiresAt` (24 h).
- Flow: insert the record as `IN_PROGRESS` → run the handler → store the response. Same key with a different body → 422 `IDEMPOTENCY_KEY_REUSED`. Same key still in progress → 409 `REQUEST_IN_PROGRESS` (the client retries later).
- Biometric events use `BiometricEvent.eventId` (unique per terminal) as their idempotency key.
- Idempotency makes **retries** safe; the unique constraints make **different requests** safe. Both are needed.

## 4. Transaction boundaries
| Use case | Inside one transaction | After commit |
|---|---|---|
| Automatic booking (per meal service) | ensure service + insert bookings + insert coupons + reconciliation voids/revivals | run audit event + `JobRun` counts |
| Opt-out | booking → OPTED_OUT + coupon → VOID + audit + idempotency response | — |
| QR / staff-assisted redemption | challenge consume (QR) + entitlement check (+ safety-net booking/coupon if missing, P-028) + coupon update + redemption + entry attempt + audit + idempotency response | gate command send |
| Biometric redemption | biometric event + entitlement + coupon + redemption + attempt + audit | gate command send |
| Failed attempt | entry attempt (+ audit) only | — |
| Gate outcome | gate command update + audit | — |

Gate I/O never happens inside a database transaction. Holding row locks while waiting for hardware would stall other students (V2 G10).

## 5. Terminal ↔ server
- The terminal is a kiosk browser running `/terminal`, authenticated by device credentials (P-009).
- Challenge rotation: the terminal calls `POST /terminal/qr-challenges` every 15 s and polls `GET /terminal/events?since=` every 1–2 s to learn about consumption and outcomes. Polling was chosen over WebSockets/SSE for simplicity; revisit if measured load requires it.
- Rotation does not invalidate the previous challenge. It stays valid until its own `expiresAt`, so a student who scanned just before rotation still succeeds (P-007). `SUPERSEDED` is used only when a terminal is deactivated or its challenges are explicitly reset.

## 6. Failure modes
| Failure | Behaviour |
|---|---|
| Phone loses network after sending | Retry with the same key → original outcome |
| API down | Terminal shows "Service unavailable — use the manual process"; nothing is granted automatically (NFR-06) |
| DB unavailable mid-transaction | Rollback; nothing redeemed; client sees 503 and may retry with the same key |
| Gate timeout after commit | `UNKNOWN` → status query → reconciliation (DR-G4) |
| Terminal clock skew | Ignored; server time decides (DR-T4); HMAC accepts ±60 s skew |
| Google certificates unreachable | Sign-in fails; existing sessions continue |
| Job runner missed a run | Auto-booking: the next run covers the whole horizon, and the entry-time safety net covers the current service (DR-AB6, DR-AB10). Coupon expiry is evaluated lazily at read and redeem time (DR-K5) |
| Host has no scheduler | Generator still runs on server start and by manual trigger; ops view flags missing coupons (FR-OPS-04, Q-17) |

## 7. Configuration
Environment variables (secrets, URLs, `DEMO_MODE`, `ALLOWED_EMAIL_DOMAINS`, timeouts) are validated with Zod at startup; the process refuses to start if they are invalid. Business values live in the database (`MealSlotConfig`) and are edited through admin APIs (D-017).

## 8. Environments
| Env | Data | `DEMO_MODE` | Biometric sim | Purpose |
|---|---|---|---|---|
| local | synthetic | true | on | development |
| test/CI | synthetic, reset per run | true | on | automated tests |
| demo | synthetic | true | on | recruiters, demonstrations |
| pilot | authorised roster only (Q-18) | **false** | **off** (D-039) | real controlled pilot: QR + staff-assisted recovery (D-040) |
