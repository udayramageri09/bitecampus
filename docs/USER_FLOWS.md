# USER_FLOWS

> Rule IDs (DR-*) → `DOMAIN_RULES.md`. Endpoints → `API_CONTRACTS.md`. Entry sequence detail → `BIOMETRIC_QR_REDEMPTION_DESIGN.md`.

## UF-00 Automatic daily booking (system flow — no student action)
1. A generator run starts (hourly schedule, server start, or manual admin trigger; P-026). It records a `JobRun`.
2. It computes the horizon in IST: today .. today + 2 (P-025, Q-07).
3. It ensures a `MealService` exists for each auto-bookable slot (Breakfast, Lunch, Dinner) on each horizon date.
4. For each meal service that has not ended, it inserts one ₹0 `HOSTEL_PREPAID` booking and one `ACTIVE` coupon per eligible hosteller (DR-E3) whose membership was recorded before that service's cutoff (DR-AB3, Q-24). Existing rows are left untouched, which includes opt-outs.
5. Reconciliation (P-027): for memberships that have ended, it voids unredeemed coupons after the end date. It revives `MEMBERSHIP_ENDED` voids where the student is eligible again before the limit.
6. It stores counts in `JobRun` and writes one audit event per run.
7. **Re-run, overlap or crash** → same final state (DR-AB5). A missed run → the next run catches up over the whole horizon.

## UF-01 Student sign-in
1. The student taps "Continue with Google" and Google returns an ID token.
2. The app sends `POST /api/v1/auth/google {credential}`.
3. The server verifies the token (DR-E1), checks the domain, then finds an `ACTIVE` `Student` by email (DR-E2).
4. **Success:** the server creates or loads the `User`, links it to the `Student` on first sign-in, grants the Student role and sets the session cookie. The dashboard already shows the student's automatic coupons.
5. **Failures:** wrong domain → D-031 message; no active student record → "We couldn't find you on the active hostel roster. Contact the hostel office." (the roster is supplied by College IT and imported by Main Admin.)"; invalid token → generic sign-in failure.

Signing in is **not** required to receive meals (DR-E4). It is required for QR entry, opt-out, history and feedback.

## UF-02 Student views meals and coupons
1. The dashboard lists today and the next horizon days: Breakfast, Lunch and Dinner, each marked "Booked automatically — covered by hostel mess fee (₹0)" with the list price for information. Snacks show "Not available yet".
2. Coupon states shown: Active · Redeemed (with time and method) · Skipped (opted out) · Expired · Cancelled by mess · On hold (student not active).
3. No coupon screen contains a QR code (FR-COUP-01).

## UF-03 Opt out of one meal (mechanism built; disabled until Q-06)
1. If `optOutEnabled` is true for that service and `now < cutoffAt`, the meal card shows **Skip this meal**.
2. A confirmation dialog explains: "You will not be able to eat this meal. This cannot be undone." (no undo, Q-06).
3. The app sends `POST /api/v1/bookings/{id}/opt-out` with an `Idempotency-Key`.
4. The server re-checks ownership, the enablement flag and the cutoff. In one transaction: booking → `OPTED_OUT`, coupon → `VOID[OPTED_OUT]`, plus an audit event. Planned quantity is unchanged.
5. **Failures:** `OPT_OUT_DISABLED`, `CUTOFF_PASSED`, `ALREADY_REDEEMED`. A retry with the same key returns the same success.

## UF-04 Entry by biometric simulator (local/test/demo/training only, D-039)
1. The terminal shows a **SIMULATOR** label; the operator or test selects an outcome and a synthetic subject.
2. **MATCH:** the terminal sends a signed identity event. The server maps `subjectRef → Student` and runs `RedemptionService` → redemption committed → gate command → the terminal shows "Welcome — Lunch redeemed".
3. **NO_MATCH / DEVICE_UNAVAILABLE / TIMEOUT:** the terminal shows "Biometric didn't work — scan the QR with BiteCampus" and switches to QR display (UF-05).
4. **MATCH but no valid entitlement** (opted out, not eligible, suspended, already redeemed): the terminal shows the reason and the gate stays closed (D-037).

## UF-05 Entry by entrance QR (the live-pilot path)
1. The terminal **displays** a rotating QR (`BC1.<token>`). It refreshes every 15 s, after expiry and right after each successful use (P-007).
2. The student opens BiteCampus on **their own phone**, signs in if needed, taps "Scan entrance QR" and the camera opens.
3. The app validates the QR format locally (FR-QR-07), generates an `Idempotency-Key` and sends `POST /api/v1/entry/qr-redemptions {challengeToken}` with the session cookie.
4. The server validates, in order: challenge hash exists → not expired → `ISSUED` → terminal active → student linked and `ACTIVE` → eligible for the current meal service at that checkpoint's mess → coupon (created by the safety net if the generator missed it, P-028) → coupon `ACTIVE`, booking `CONFIRMED`, not demo.
5. In **one transaction** the server consumes the challenge, records the redemption, marks the coupon `REDEEMED` and writes the `EntryAttempt` and audit records.
6. After commit, the server sends a gate command for that terminal's gate.
7. The phone shows one of:
   - "Lunch redeemed — entry confirmed" (`OPENED`)
   - "Lunch redeemed — gate didn't respond. Tap Retry entry, or ask counter staff" (`UNKNOWN`/`FAULT`) → UF-06
   - "Not redeemed — <reason>" (expired QR, already-used QR, skipped this meal, not active, already redeemed, outside window)
8. The terminal shows the same outcome and a new QR.
9. **Network failure on the phone:** the app retries with the **same** key and gets the original outcome. It never produces a second redemption.

## UF-06 Gate timeout reconciliation
1. The redemption is committed; the gate command times out (3 s, P-016) → `UNKNOWN`.
2. The server queries the gate simulator by command ID: opened → `ACKNOWLEDGED_OPEN`; not opened → `FAILED` (retry allowed); still unknown → reconciliation queue.
3. The student taps **Retry entry** → `POST /api/v1/entry/redemptions/{id}/gate-retry`. This creates a new gate command for the **same** redemption (DR-G5).
4. Counter Staff see the queue and can retry. Meal Admin or Main Admin resolves remaining items with a note (Q-09).
5. Scanning a new QR after a successful redemption returns `ALREADY_REDEEMED` with a "Retry entry" option. It never redeems again.

## UF-07 Staff-assisted recovery (live pilot, D-040)
Used when the student cannot complete QR entry (no phone, flat battery, camera problem, app issue).
1. Counter Staff open "Assisted entry" and search the student by roster ID or email.
2. The screen shows only today's entitlement status for the current meal.
3. Staff perform the identity check required by policy (Q-10), choose a reason code and confirm.
4. `RedemptionService` runs with method `STAFF_ASSISTED` → normal gate flow. Audited with the staff member's identity.
5. If the student has no valid entitlement, staff see the reason; recovery cannot create one (DR-S5).

## UF-08 Demo booking and simulated checkout (`DEMO_MODE=true` only, P-013)
1. A **demo (non-hosteller) student** opens a meal and taps "Book (demo)" → `POST /api/v1/demo/bookings` → `PENDING_PAYMENT`, `SIMULATED_PAYMENT`.
2. The checkout screen is labelled **DEMO / SIMULATED — NO MONEY CHARGED**. The user picks Success / Fail / Cancel / Leave pending → `POST /api/v1/demo/payments`.
3. Success → `CONFIRMED` and a coupon with `isDemo = true`. A pilot checkpoint rejects it (DR-P6). Fail/Cancel → the booking stays `PENDING_PAYMENT` and a new attempt is possible. At the cutoff it becomes `EXPIRED`.

## UF-09 Meal Admin: configuration and closures
1. Edit slot config. DR-C1 rejects enabling Snacks without a price and cutoff; `optOutEnabled` stays off until Q-06.
2. Add or edit menus on generated meal services.
3. Void a service for a closure (Q-15, Q-27) → its coupons become `VOID[SERVICE_VOIDED]`; students see "Cancelled by mess".
4. View generator run history; trigger a manual run (idempotent).

## UF-10 Inventory Admin: planned quantity
1. Inventory Admin (or Meal Admin) opens the meal-service ops view: **Planned** (manual), **Auto-booked**, **Opted out**, **Redeemed**.
2. They set or adjust Planned with a mandatory reason → audit event (DR-KQ3).
3. Automatic booking and opt-outs never change Planned (DR-KQ2).

## UF-11 Main Admin: academic year, roster and membership (D-045, P-011, P-029)
0. Set the academic-year dates and hostel meal rules (D-045).
1. Upload the College IT roster (CSV/XLSX) → preview with validation and duplicate checks (students added/updated, memberships started/ended, errors) → confirm → apply in one transaction → audit event with counts only.
2. Membership actions: end (with `validTo`), suspend, reinstate, allocation change (end + start). Each is audited.
3. The next generator run (or a manual trigger) applies the effects: new entitlements for joiners for every meal whose cutoff is after the import (Q-24), voids for leavers (P-027).

## UF-12 Role management
Main Admin searches a user → grants or revokes a role (USER_ROLES_AND_PERMISSIONS §3) → audit event.

## UF-13 Feedback
After a meal service the student held a coupon for, the signed-in student opens "Rate this meal" → sees the identifiability notice → submits rating and comment. Meal Admin, Inventory Admin and Main Admin review it.

## UF-14 Meal Admin reopens a window (D-046)
Meal Admin selects one meal service → reopens its opt-out (or demo/paid booking) window until a new time → mandatory reason → audit event. Other services are unaffected.

## Deferred flows (V1.1/V1.2) — outline only
- **Paid diner (D-044):** a diner (including outsiders) with a verified Google account buys a meal coupon with simulated payment; refunds by Meal Admin after Hostel Director approval for hostel-management-caused issues only, audited.
- **Staff attendance (F-015):** Staff Manager records kitchen-staff attendance and audited corrections.
- **Event coupons (BL):** organiser requests quantity and guest details → authorised approval recorded (approver Q-04) → Meal Admin generates and emails assigned coupons → guest redeems through `RedemptionService` with guest rules (QR model Q-28).
- **Inventory:** stock receipt → issue to kitchen → consumption vs plan → waste record → reports.
- **Notifications:** opt-out cutoff reminders, meal-change notices.
