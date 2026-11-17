# ACCEPTANCE_CRITERIA

Given/When/Then criteria keyed to `FUNCTIONAL_REQUIREMENTS.md`. Each criterion names the test layer that proves it: **U** unit, **I** API/service integration against real PostgreSQL, **E** Playwright end-to-end, **M** manual check.

## 0. Pilot exit criteria
- All criteria below pass, with commands and results recorded.
- AC-AB-02/03 (generator idempotency), AC-RED-03 (concurrency) and AC-QR-06 (replay) pass against real PostgreSQL, not mocks.
- The live pilot configuration has QR entry and staff-assisted recovery working, and the biometric simulator disabled (AC-BIO-06).
- No UNRESOLVED decision is silently implemented; each uses its safe default from `OPEN_DECISIONS.md` §4.
- Database backup and restore tested (NFR-15). Privacy notice and disclaimers visible.

## Authentication
- **AC-AUTH-01 (I):** Given a token with an invalid signature, wrong audience, expired, or `email_verified=false`, when it is posted to `/auth/google`, then 401 `AUTH_INVALID_TOKEN` and no user is created.
- **AC-AUTH-02 (I):** A verified token for `x@other.com` → 403 `AUTH_DOMAIN_NOT_ALLOWED` with the D-031 message.
- **AC-AUTH-03 (I):** An allowed-domain email with no `ACTIVE` `Student` → 403 `AUTH_NOT_ON_ROSTER`.
- **AC-AUTH-04 (I):** First sign-in of an eligible student → `User` linked to the `Student`; only the `STUDENT` role; coupons created before sign-in are visible immediately.
- **AC-AUTH-05 (I):** After logout, the old session cookie returns 401.

## Automatic booking
- **AC-AB-01 (I):** Given 3 eligible hostellers and a horizon of 3 days with B/L/D auto-bookable, when the generator runs, then 27 bookings (`AUTO`, `CONFIRMED`, `HOSTEL_PREPAID`, `amountPayable=0`, list price snapshot) and 27 `ACTIVE` coupons exist, and no `PaymentAttempt` exists.
- **AC-AB-02 (I):** Running the generator a second time → 0 created; row counts unchanged.
- **AC-AB-03 (I):** Two generator runs started concurrently (`Promise.all`) → final state identical to one run; no unique-violation surfaces as a failure.
- **AC-AB-04 (I):** A run killed after creating bookings for one meal service and before the next → the next run completes the rest, **even if it runs after those services' cutoffs**; every confirmed booking has exactly one coupon.
- **AC-AB-05 (I):** Snacks (`bookingEnabled=false`) → no Snacks bookings.
- **AC-AB-06 (I):** A student with `status=SUSPENDED` or `INACTIVE` → no new bookings; an existing `ACTIVE` coupon is rejected at entry with `STUDENT_NOT_ACTIVE`; after reinstatement the coupon is redeemable without repair.
- **AC-AB-07 (I):** Membership `validTo = D` → coupons for D are kept (inclusive, Q-26 default); unredeemed coupons for D+1 and later become `VOID[MEMBERSHIP_ENDED]` on the next run.
- **AC-AB-08 (I):** Membership starting on D, recorded at 11:00 IST on D → under the Q-24 default (membership recorded before cutoff), Lunch(D) and Dinner(D) are **not** generated because their 10:00 cutoff had already passed when it was recorded. Breakfast(D+1), whose cutoff is D 19:00, **is** generated. The test asserts the configured limit, so it changes in one place if Q-24 changes.
- **AC-AB-09 (I):** Allocation change from mess A to mess B effective D+1 → A coupons after D voided; B coupons for D+1 onward generated.
- **AC-AB-10 (I):** Generation, opt-out, voiding and revival leave `MealService.plannedQuantity` unchanged.
- **AC-AB-11 (I):** Each run writes a `JobRun` with counts and exactly one audit event; each booking carries `generationRunId`.
- **AC-AB-12 (I):** Entry-time safety net: the generator has not run for Lunch(D) (simulated job failure). A hosteller whose membership was recorded on D-3 scans the QR at 13:10 → booking, coupon and redemption are created in one transaction. A hosteller whose membership was recorded at 11:00 on D → `NO_ENTITLEMENT`.
- **AC-AB-13 (U):** Horizon and cutoff computations across midnight IST (e.g. 18:29:59 UTC vs 18:30:00 UTC) produce the correct campus dates.

## Opt-out
- **AC-OO-01 (I):** With `optOutEnabled=false` (default) → 422 `OPT_OUT_DISABLED`.
- **AC-OO-02 (I):** With opt-out enabled, before the cutoff → 200; booking `OPTED_OUT`; coupon `VOID[OPTED_OUT]`; other meals unchanged; audit event written.
- **AC-OO-03 (U+I):** Clock fixed at Lunch-day 10:00:00 IST → 422 `CUTOFF_PASSED`; at 09:59:59 → 200. Breakfast(D) opt-out allowed until D-1 19:00 IST.
- **AC-OO-04 (I):** After an opt-out, generator re-runs never re-create or revive the booking.
- **AC-OO-05 (I):** Opting out of another student's booking → 404. Opting out of a redeemed coupon → 409 `ALREADY_REDEEMED`.
- **AC-OO-06 (I):** Opted-out student scans the QR → 403 `OPTED_OUT`; the safety net does not create a new booking.

## Demo payment
- **AC-DEMO-01 (I):** With `DEMO_MODE=false`, every `/demo/*` route returns 404.
- **AC-DEMO-02 (I):** A hosteller calling `/demo/bookings` → 403 `NOT_ELIGIBLE` (demo bookings are for demo students only).
- **AC-DEMO-03 (I):** In demo mode, SUCCEEDED → `CONFIRMED` + demo coupon; FAILED/CANCELLED → stays `PENDING_PAYMENT`; cutoff → `EXPIRED`; repeated outcome reports are idempotent.
- **AC-DEMO-04 (I):** A demo coupon at a checkpoint with `acceptsDemoCoupons=false` → `DEMO_COUPON_NOT_ACCEPTED`; no redemption row.

## Coupons and history
- **AC-COUP-01 (E):** The coupon screen shows meal, date, window, status and funding source, and contains no QR code element.
- **AC-HIST-01 (I):** Student A requesting student B's booking or coupon by ID → 404 (no existence leak).

## Entrance QR
- **AC-QR-01 (I):** A terminal-issued challenge stores only a SHA-256 hash; the raw token appears in no table or log.
- **AC-QR-02 (I):** A valid challenge plus an entitled student session → 200 `REDEEMED`; challenge `CONSUMED`; coupon `REDEEMED`; one `Redemption` row.
- **AC-QR-03 (I):** A challenge older than TTL → 410 `QR_EXPIRED`; nothing redeemed.
- **AC-QR-04 (I):** A consumed challenge reused by a different student → 409 `QR_ALREADY_USED`.
- **AC-QR-05 (I):** No session → 401; QR scanning alone never redeems (D-033).
- **AC-QR-06 (I):** The same challenge submitted by two entitled students concurrently → exactly one redemption; the other gets `QR_ALREADY_USED`.
- **AC-QR-07 (I):** Phone retry with the same `Idempotency-Key` after a success → the original 200 body; still one redemption.
- **AC-QR-08 (U):** The scanner rejects payloads not matching `BC1.<base64url>` without calling the API.
- **AC-QR-09 (I):** Valid challenge, but the student is not eligible or has no valid coupon → 403 with the specific code; the challenge stays `ISSUED` (P-008); an `EntryAttempt` is recorded.
- **AC-QR-10 (E/M):** On a phone, scanning the terminal QR shows the confirmation within the NFR-03 target in the test environment (measured, recorded).

## Biometric simulator
- **AC-BIO-01 (I):** A MATCH event with a valid HMAC signature → redemption via the same service; method `BIOMETRIC`.
- **AC-BIO-02 (I):** A bad signature, stale timestamp or reused nonce → 401; nothing redeemed.
- **AC-BIO-03 (I):** The same `eventId` posted twice → the same outcome; one redemption.
- **AC-BIO-04 (E):** NO_MATCH, DEVICE_UNAVAILABLE and TIMEOUT each switch the terminal to QR display.
- **AC-BIO-05 (I):** The database schema contains no biometric image, template or sample field (schema review + test).
- **AC-BIO-06 (I):** With pilot configuration, `POST /terminal/biometric-events` → 404 `NOT_FOUND`, and the terminal UI shows no biometric option.
- **AC-BIO-07 (E):** Every simulator screen shows a visible **SIMULATOR** label.

## Staff-assisted recovery
- **AC-STA-01 (I):** `entry.assist` holder, valid entitlement, reason code → redemption with method `STAFF_ASSISTED` and `actorUserId` recorded.
- **AC-STA-02 (I):** Missing reason code → 400; user without `entry.assist` → 403.
- **AC-STA-03 (I):** Student without a valid coupon → rejected with the reason; no booking or coupon is created except by the DR-AB10 safety net for an eligible hosteller with no booking row.
- **AC-STA-04 (I):** Staff-assisted and QR redemption racing for the same coupon → one redemption.

## Redemption (cross-method)
- **AC-RED-01 (I):** Biometric MATCH and QR for the same student and coupon at the same time → exactly one redemption; the other `ALREADY_REDEEMED`.
- **AC-RED-02 (I):** Redemption at 15:30:00 IST for Lunch → `OUTSIDE_SERVICE_WINDOW` (half-open window, no grace until Q-08).
- **AC-RED-03 (I):** 50 parallel redemption attempts for one coupon across methods → one `Redemption` row, 49 rejections, no 5xx.
- **AC-RED-04 (I):** Every attempt writes an `EntryAttempt` with a reason code.
- **AC-RED-05 (I):** A student with an active membership but no coupon for the meal (e.g. a voided service) → rejected; membership alone never redeems (D-037).

## Gate
- **AC-GATE-01 (I):** The gate simulator in TIMEOUT mode → response `REDEEMED` + `entry=PENDING`; gate command `UNKNOWN`; redemption count 1.
- **AC-GATE-02 (I):** Gate retry for the same redemption → a new gate command; redemption count still 1.
- **AC-GATE-03 (I):** Gate failure never rolls back the redemption.
- **AC-GATE-04 (I):** Gate retry by another student → 404; by Counter Staff → allowed; audited.

## Admin, membership, ops, audit, feedback
- **AC-ADMIN-01 (I):** Enabling Snacks booking without price and cutoff → 422 `CONFIG_INCOMPLETE`; enabling `optOutEnabled` or `autoBookEnabled` while `bookingEnabled=false` → 422.
- **AC-ADMIN-02 (I):** A slot price change does not alter already generated meal services or bookings (P-019).
- **AC-ADMIN-03 (I):** Voiding a meal service → all its `ACTIVE` coupons `VOID[SERVICE_VOIDED]`; redeemed coupons unchanged.
- **AC-MEM-01 (I):** Roster import dry-run changes nothing; apply creates/updates students and memberships in one transaction; overlapping ACTIVE memberships for one student are rejected by the database.
- **AC-OPS-01 (I):** Planned-quantity adjustment without a reason → 422; with a reason → updated value plus an audit event with old/new values.
- **AC-OPS-02 (I):** The ops view returns planned, auto-booked, opted-out and redeemed as separate fields.
- **AC-OPS-03 (I):** The ops view flags a meal service in the horizon that has confirmed bookings without coupons, or no bookings although eligible hostellers exist.
- **AC-RBAC-01 (I):** Each protected route has 401 and 403 tests per the matrix.
- **AC-AUDIT-01 (I):** No API exists to update or delete audit events.
- **AC-FB-01 (I):** Feedback for a meal service without a coupon → 403; a duplicate → 409.
