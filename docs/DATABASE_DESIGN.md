# DATABASE_DESIGN

> **Proposed logical schema (ARCH).** It must be reconciled with the existing Prisma schema and migrations found by the repository audit. Never reset or rewrite existing migrations. Changes are additive migrations, approved first.

Conventions: `id` = UUID; all instants `timestamptz` (UTC); campus dates `date`; money stored as integer **paise**; enums as Postgres enums via Prisma.

## 1. Entity overview
```text
User (Google login) ──0..1── Student (roster person)            ← P-023
User ─< UserRole >─ Role ─< RolePermission >─ Permission
User ─< Session
Student ─< HostelMembership >─ Mess ─< Checkpoint ─< EntranceTerminal
Mess ─< MealService >─ MealSlotConfig ;  MealService ─< MenuItem
MealService ─< Booking >─ Student      (UNIQUE studentId+mealServiceId)
Booking ─< PaymentAttempt              (demo only)
Booking ── MealCoupon (1:1) ── Redemption (0..1) ─< GateCommand
EntranceTerminal ─< QrChallenge ; EntranceTerminal ─< BiometricEvent
EntryAttempt (every attempt) · BiometricSubject (synthetic subjectRef → Student)
MealService ─< PlannedQuantityAdjustment ; MealService ─< Feedback >─ Student
RosterImportBatch · JobRun · AuditEvent · IdempotencyRecord
```

The key design choice: entitlements belong to **`Student`**, not to the Google `User`. A hosteller who never signs in still has bookings and coupons; signing in links the `User` (DR-E2, DR-E4).

## 2. Tables (key columns and constraints)

### Identity and membership
| Table | Columns | Constraints |
|---|---|---|
| User | id, email (normalised), googleSub, displayName, status(ACTIVE/DISABLED), createdAt | UNIQUE(email), UNIQUE(googleSub) |
| Student | id, rosterRef (opaque college ID), email (normalised), displayName, userId?, status(ACTIVE/SUSPENDED/INACTIVE), isDemo, updatedAt | UNIQUE(rosterRef), UNIQUE(email), UNIQUE(userId) |
| Mess | id, name, active | — |
| HostelMembership | id, studentId, messId, status(ACTIVE/SUSPENDED/ENDED), validFrom(date), validTo(date?), source(IMPORT/MANUAL), importBatchId?, recordedAt (immutable; drives DR-AB3), updatedAt | CHECK(validTo IS NULL OR validFrom ≤ validTo); **exclusion constraint**: no two overlapping ACTIVE memberships per student (`EXCLUDE USING gist (studentId WITH =, daterange(validFrom, validTo, '[]') WITH &&) WHERE (status='ACTIVE')`) |
| RosterImportBatch | id, uploadedBy, fileHash, dryRun, rowsTotal, created, updated, ended, errors (jsonb, no PII), appliedAt? | — |
| Role / Permission | id, key | UNIQUE(key) |
| UserRole | userId, roleId, grantedBy, grantedAt | PK(userId, roleId) |
| Session | id (random, stored hashed), userId, expiresAt, revokedAt, createdAt, lastSeenAt | INDEX(userId) |

### Catalog
| Table | Columns | Constraints |
|---|---|---|
| MealSlotConfig | id, mealType(BREAKFAST/LUNCH/SNACKS/DINNER), windowStartLocal, windowEndLocal, cutoffDayOffset?, cutoffLocalTime?, listPricePaise?, bookingEnabled, autoBookEnabled, optOutEnabled, updatedBy, updatedAt | UNIQUE(mealType); CHECKs for DR-C1 (§3) |
| MealService | id, messId, mealType, serviceDate(date), windowStartAt, windowEndAt, cutoffAt, listPricePaise?, bookingEnabled, autoBookEnabled, optOutEnabled, status(PUBLISHED/VOIDED), plannedQuantity?, voidReason?, version, generatedByRunId? | **UNIQUE(messId, mealType, serviceDate)**, which makes meal-service generation idempotent; CHECK(windowStartAt < windowEndAt) |
| MenuItem | id, mealServiceId, name, sortOrder | — |

Meal services are generated as `PUBLISHED` (P-024). Without a menu, the app shows "Menu not yet published". There is no DRAFT state, because generation is automatic.

### Booking, payment, coupon
| Table | Columns | Constraints |
|---|---|---|
| Booking | id, studentId, mealServiceId, source(AUTO/DEMO), status(CONFIRMED/OPTED_OUT/VOIDED/PENDING_PAYMENT/EXPIRED), voidReason?(SERVICE_VOIDED/MEMBERSHIP_ENDED), fundingSource(HOSTEL_PREPAID/SIMULATED_PAYMENT), listPricePaise, amountPayablePaise, isDemo, generationRunId?, createdAt, confirmedAt?, optedOutAt?, optedOutByUserId? | **UNIQUE(studentId, mealServiceId)** (full, P-005); CHECK(fundingSource <> 'HOSTEL_PREPAID' OR amountPayablePaise = 0); CHECK(source <> 'AUTO' OR fundingSource = 'HOSTEL_PREPAID'); CHECK((status = 'VOIDED') = (voidReason IS NOT NULL)) |
| PaymentAttempt | id, bookingId, provider('SIMULATED'), status(CREATED/PENDING/SUCCEEDED/FAILED/CANCELLED/EXPIRED), demoReference, createdAt, resolvedAt? | UNIQUE(demoReference); at most one SUCCEEDED per booking (partial unique index) |
| MealCoupon | id, bookingId, studentId, mealServiceId, status(ACTIVE/REDEEMED/EXPIRED/VOID), voidReason?(OPTED_OUT/SERVICE_VOIDED/MEMBERSHIP_ENDED), fundingSource, isDemo, issuedAt, redeemedAt? | **UNIQUE(bookingId)**; UNIQUE(studentId, mealServiceId); INDEX(studentId, status) |

### Entry, redemption, gate
| Table | Columns | Constraints |
|---|---|---|
| Checkpoint | id, messId, name, acceptsDemoCoupons, active | — |
| EntranceTerminal | id, checkpointId, name, credentialEncrypted (AES-GCM, see AUTHENTICATION_AND_AUTHORIZATION §5), biometricSimulatorEnabled, active, lastSeenAt | — |
| QrChallenge | id, terminalId, checkpointId, tokenHash (bytea, SHA-256), status(ISSUED/CONSUMED/EXPIRED/SUPERSEDED), issuedAt, expiresAt, consumedAt?, consumedByStudentId?, redemptionId? | UNIQUE(tokenHash); INDEX(terminalId, status) |
| BiometricSubject | studentId, subjectRef (synthetic) | PK(subjectRef); UNIQUE(studentId) |
| BiometricEvent | id, terminalId, eventId, subjectRef?, outcome(MATCH/NO_MATCH/DEVICE_UNAVAILABLE/TIMEOUT), occurredAt (terminal), receivedAt, nonce | UNIQUE(terminalId, eventId); UNIQUE(terminalId, nonce) |
| EntryAttempt | id, method(BIOMETRIC/QR/STAFF_ASSISTED), terminalId?, checkpointId, studentId?, couponId?, qrChallengeId?, biometricEventId?, actorUserId?, reasonCode, outcome(REDEEMED/REJECTED/ERROR), createdAt | INDEX(checkpointId, createdAt) |
| Redemption | id, couponId, studentId, mealServiceId, checkpointId, method, entryAttemptId, redeemedAt | **UNIQUE(couponId)** — the exactly-once guarantee |
| GateCommand | id, redemptionId, terminalId, attemptNo, status(SENT/ACKNOWLEDGED_OPEN/FAILED/UNKNOWN), outcomeDetail, requestedBy, sentAt, resolvedAt?, resolutionNote? | UNIQUE(redemptionId, attemptNo) |

No biometric image, template or sample column exists anywhere (AC-BIO-05).

### Operations, feedback, platform
| Table | Columns | Constraints |
|---|---|---|
| PlannedQuantityAdjustment | id, mealServiceId, previousQuantity?, newQuantity, reason, actorUserId, createdAt | CHECK(newQuantity >= 0); CHECK(length(reason) > 0) |
| Feedback | id, studentId, mealServiceId, rating(1–5), comment?, createdAt | UNIQUE(studentId, mealServiceId); CHECK(rating BETWEEN 1 AND 5) |
| JobRun | id, job(AUTO_BOOKING/COUPON_EXPIRY/GATE_RECONCILE/CHALLENGE_EXPIRY), trigger(SCHEDULE/STARTUP/MANUAL), horizonFrom?, horizonTo?, status(RUNNING/SUCCEEDED/FAILED), counts (jsonb), error?, startedAt, finishedAt? | INDEX(job, startedAt) |
| AuditEvent | id, occurredAt, actorType(USER/TERMINAL/SYSTEM), actorId, action, entityType, entityId, data (jsonb, no secrets), requestId | INDEX(entityType, entityId); INDEX(occurredAt); append-only by privilege (§5) |
| IdempotencyRecord | principalId, route, key, requestHash, status, responseCode?, responseBody?, createdAt, expiresAt | PK(principalId, route, key) |

Counts (booked, opted out, redeemed) are **derived by query**, never stored on `MealService`. That keeps planned, booked and attended separate (D-024).

## 3. Constraints Prisma cannot express
Add these in hand-edited SQL inside the generated migration (documented in the migration file):
```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE "HostelMembership" ADD CONSTRAINT one_active_membership_at_a_time
  EXCLUDE USING gist ("studentId" WITH =, daterange("validFrom","validTo",'[]') WITH &&)
  WHERE (status = 'ACTIVE');

ALTER TABLE "MealSlotConfig"
  ADD CONSTRAINT enabled_requires_price_cutoff
    CHECK (NOT "bookingEnabled" OR ("listPricePaise" IS NOT NULL AND "cutoffLocalTime" IS NOT NULL)),
  ADD CONSTRAINT auto_requires_enabled  CHECK (NOT "autoBookEnabled" OR "bookingEnabled"),
  ADD CONSTRAINT optout_requires_enabled CHECK (NOT "optOutEnabled"  OR "bookingEnabled");

ALTER TABLE "Booking"
  ADD CONSTRAINT prepaid_is_zero  CHECK ("fundingSource" <> 'HOSTEL_PREPAID' OR "amountPayablePaise" = 0),
  ADD CONSTRAINT auto_is_prepaid  CHECK ("source" <> 'AUTO' OR "fundingSource" = 'HOSTEL_PREPAID');

CREATE UNIQUE INDEX one_success_per_booking ON "PaymentAttempt" ("bookingId") WHERE status = 'SUCCEEDED';
```

## 4. Automatic booking — how the database makes it idempotent
One generator run, for horizon dates `[today, today + H]` in IST (P-025), executes per meal service (or per date) in short transactions:

```sql
-- 1. Ensure meal services exist (idempotent via UNIQUE(messId, mealType, serviceDate))
INSERT INTO "MealService" (...) SELECT ... FROM generate_series(:from, :to, '1 day') d
  CROSS JOIN "MealSlotConfig" s WHERE s."autoBookEnabled"
ON CONFLICT ("messId","mealType","serviceDate") DO NOTHING;

-- 2. Create bookings for eligible hostellers (idempotent via UNIQUE(studentId, mealServiceId))
INSERT INTO "Booking" (id, "studentId","mealServiceId", source, status, "fundingSource",
                       "listPricePaise","amountPayablePaise","generationRunId", ...)
SELECT gen_random_uuid(), st.id, ms.id, 'AUTO','CONFIRMED','HOSTEL_PREPAID', ms."listPricePaise", 0, :runId, ...
FROM "MealService" ms
JOIN "HostelMembership" m ON m."messId" = ms."messId" AND m.status = 'ACTIVE'
     AND ms."serviceDate" BETWEEN m."validFrom" AND COALESCE(m."validTo", 'infinity')
JOIN "Student" st ON st.id = m."studentId" AND st.status = 'ACTIVE' AND NOT st."isDemo"
WHERE ms.id = :mealServiceId AND ms.status = 'PUBLISHED' AND ms."autoBookEnabled"
  AND :now < ms."windowEndAt"                       -- service not over; :now from injected Clock
  AND m."recordedAt" < ms."cutoffAt"                -- generation limit (DR-AB3, Q-24 default)
ON CONFLICT ("studentId","mealServiceId") DO NOTHING;   -- re-runs and opted-out rows untouched

-- 3. Create missing coupons for CONFIRMED auto bookings (idempotent via UNIQUE(bookingId))
INSERT INTO "MealCoupon" (...) SELECT ... FROM "Booking" b
WHERE b."mealServiceId" = :mealServiceId AND b.status = 'CONFIRMED'
ON CONFLICT ("bookingId") DO NOTHING;

-- 4. Reconcile: void for ended memberships; revive MEMBERSHIP_ENDED voids that are eligible again (P-027)
```
- Steps 2 and 3 run in the **same transaction** per meal service, so a crash never leaves a confirmed booking without a coupon. If one does exist (for example, from a manual repair), step 3 repairs it on the next run.
- `pg_try_advisory_xact_lock(hashtext('auto_booking'))` prevents wasted parallel runs (P-026). It is an optimisation; the unique constraints are the guarantee.
- `RETURNING` counts feed `JobRun.counts` and the run's audit event (P-030).
- The entry-time safety net (P-028) executes steps 2–3 for one student and one service inside the redemption transaction.

**Scale check (estimate):** 500 students × 3 meals × 3 days = 4,500 candidate rows per run, almost all `DO NOTHING` after the first run. This is trivial for PostgreSQL. Measure before claiming numbers (P-020).

## 5. Audit immutability
The application database role gets `INSERT, SELECT` on `AuditEvent` only (no `UPDATE`/`DELETE`). The migration role owns the table. This is enforced by grants, not just by code (DR-A1).

## 6. Indexes for known queries
- Generator: `HostelMembership(messId, status, validFrom, validTo, recordedAt)`; `MealService(messId, serviceDate)`.
- Student app: `MealCoupon(studentId, status)`; `Booking(studentId, mealServiceId)` (the unique index).
- Entrance: `MealService(messId, windowStartAt, windowEndAt)`; `MealCoupon(studentId, mealServiceId)`.
- Ops: `Booking(mealServiceId, status)`, `Redemption(mealServiceId)`.
- Add further indexes only after `EXPLAIN ANALYZE` on real query patterns.

## 7. Seed data
Synthetic only (D-030): slot config defaults (DOMAIN_RULES §2; Snacks disabled, price null; `optOutEnabled` false); one mess; one checkpoint (`acceptsDemoCoupons` true locally, false in pilot); one terminal (biometric simulator on locally, off in pilot); synthetic students with ACTIVE memberships, plus one suspended, one ended and one future-joining student to exercise DR-AB8/AB9/Q-24; demo (non-hosteller) students for P-013; one user per role; no real names or emails.

## 8. Migration policy
Use `prisma migrate dev` locally and `prisma migrate deploy` in deployed environments. Never run `migrate reset` on any shared database. Every migration is reviewed by the database-reviewer agent and approved by the owner.
