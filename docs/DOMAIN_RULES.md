# DOMAIN_RULES — Business Rules, Configuration and State Machines

This file owns **business values** and **lifecycle rules**. Statuses → `OPEN_DECISIONS.md`. Schema → `DATABASE_DESIGN.md`.

Rule IDs (`DR-*`) are referenced by tests and acceptance criteria.

## 0. Concepts kept separate (D-036)
| Concept | Record | Answers |
|---|---|---|
| Active hostel membership | `HostelMembership` | Does this student currently belong to this hostel mess? |
| Automatic daily booking | `Booking` (source `AUTO`) | Has the system reserved this meal for this student? |
| Meal entitlement | `MealCoupon` (P-010) | May this student eat this meal service? |
| Digital coupon | App view of `MealCoupon` | What does the student see? |
| Identity verification | `BiometricEvent` / `QrChallenge` / staff-assisted `EntryAttempt` | Who is at the entrance? |
| Coupon redemption | `Redemption` | Was this entitlement consumed (once)? |
| Physical entry response | `GateCommand` | Did the gate let them in? |

Membership alone never authorises entry (D-037). Identity success alone never authorises a meal (D-021).

---

## 1. Time
| ID | Rule | Status |
|---|---|---|
| DR-T1 | Store every instant in UTC (`timestamptz`) | APPROVED (D-016) |
| DR-T2 | Evaluate windows, cutoffs, "today" and the generation horizon in `Asia/Kolkata` | APPROVED (D-016) |
| DR-T3 | A meal service's `serviceDate` and a membership's `validFrom`/`validTo` are campus-local calendar dates (SQL `date`) | APPROVED (D-016) |
| DR-T4 | The server clock is the only clock. Client and terminal timestamps are recorded for diagnostics only | PROPOSED (ARCH) |
| DR-T5 | All time-based logic receives an injected `Clock` so tests can fix "now" | PROPOSED (ARCH) |

## 2. Meal slot configuration (defaults)
All values are configuration (D-017), stored in `MealSlotConfig` and editable by Meal Admin.

| Slot | List price | Service window (IST) | Cutoff (IST) | `bookingEnabled` | `autoBookEnabled` | `optOutEnabled` | Status |
|---|---|---|---|---|---|---|---|
| Breakfast | ₹45 | 07:00–09:30 | Previous day 19:00 | true | true | **false** | APPROVED; opt-out pending Q-06 |
| Lunch | ₹70 | 13:00–15:30 | Same day 10:00 | true | true | **false** | APPROVED; opt-out pending Q-06 |
| Snacks | **not set** (Q-19) | 16:30–17:30 | **not set** (Q-19) | **false** | false | false | Window and disabled state APPROVED; rest UNRESOLVED |
| Dinner | ₹70 | 19:30–21:30 | Same day 10:00 | true | true | **false** | APPROVED; opt-out pending Q-06 |

| ID | Rule | Status |
|---|---|---|
| DR-C1 | `bookingEnabled = true` requires a list price and a cutoff. `autoBookEnabled` and `optOutEnabled` each require `bookingEnabled` | APPROVED intent (D-015); enforcement ARCH |
| DR-C2 | Cutoff is expressed as `{ dayOffset, localTime }` relative to `serviceDate` (Breakfast `{-1, 19:00}`, Lunch/Dinner `{0, 10:00}`) | PROPOSED (ARCH) |
| DR-C3 | A meal service copies the slot's window, cutoff, price and flags when it is generated. Editing a slot affects only meal services generated afterwards | PROPOSED (ARCH, P-019) |
| DR-C4 | Window boundaries are half-open: `[start, end)` | PROPOSED (ARCH) |
| DR-C5 | Under automatic booking, the cutoff is the **opt-out cutoff** and the demo paid-booking cutoff | APPROVED (D-014) |

## 3. Students, sign-in and membership
| ID | Rule | Status |
|---|---|---|
| DR-E1 | Sign-in requires a verified Google token with `email_verified = true` and an email domain in `ALLOWED_EMAIL_DOMAINS` (Q-01) | APPROVED (D-005) |
| DR-E2 | Sign-in as a student requires a `Student` record matching the email (normalised) with `status = ACTIVE`. The `User` is linked to that `Student` on first sign-in | APPROVED (D-006); linking PROPOSED (P-023) |
| DR-E3 | Hosteller eligibility for a meal service = `Student.status = ACTIVE` **and** a `HostelMembership` with `status = ACTIVE`, the service's mess, and `validFrom ≤ serviceDate ≤ validTo` (open-ended if `validTo` is null; inclusive per Q-26). Creating a new entitlement additionally requires DR-AB3 | APPROVED intent (D-034, D-042); model PROPOSED (P-029) |
| DR-E4 | A student never needs to sign in to receive entitlements. Signing in is needed only for QR entry, opt-out, history and feedback | APPROVED (D-034); P-023 |
| DR-E5 | Domain, roster or membership never grants a non-student role (D-007) | APPROVED |
| DR-E6 | Out-of-domain users see the message in D-031 | APPROVED |
| DR-E7 | Membership changes come from the roster import or Main Admin edits; memberships default to the configured academic-year dates (D-045); each change is audited | PROPOSED (P-011, P-030) |
| DR-E8 | Allocation change (different hostel/mess) = end the current membership and start a new one; never edit the mess of an existing membership | PROPOSED (P-029) |

## 4. Automatic daily booking
### 4.1 Rules
| ID | Rule | Status |
|---|---|---|
| DR-AB1 | For every campus date in the horizon (P-025, Q-07) and every slot with `autoBookEnabled`, a meal service exists for the mess (generated if missing) | APPROVED intent (D-034); generation PROPOSED (P-024) |
| DR-AB2 | For every eligible hosteller (DR-E3) and every published, not-yet-ended meal service in the horizon that satisfies the **generation limit** (DR-AB3), exactly one booking (`source = AUTO`, `CONFIRMED`, `HOSTEL_PREPAID`, ₹0) and exactly one `ACTIVE` coupon exist | APPROVED (D-034, D-008) |
| DR-AB3 | Generation limit: an entitlement is created only if the student's membership was **recorded** (`HostelMembership.recordedAt`) before the service's cutoff (Q-24 default). This excludes late joiners, but lets any later run, or the entry-time safety net, repair a service that a failed job missed | UNRESOLVED (safe default) |
| DR-AB4 | Uniqueness: one booking per (student, meal service), ever (P-005); one coupon per booking. Enforced by database constraints | APPROVED intent (D-035); mechanism PROPOSED |
| DR-AB5 | Generation is a set-based insert `… ON CONFLICT DO NOTHING`, so re-running it any number of times, in parallel or after a crash, produces the same final state | APPROVED (D-035) |
| DR-AB6 | Runs on a schedule, on server start and by manual trigger, each covering the whole horizon (P-026). A missed or failed run is repaired by the next one | APPROVED intent (D-035); triggers PROPOSED |
| DR-AB7 | A generator re-run **never** re-creates or revives an opted-out booking | APPROVED (D-038) |
| DR-AB8 | Membership ended → unredeemed coupons for services after `validTo` are voided (`MEMBERSHIP_ENDED`). If eligibility returns before a service's generation limit, the generator revives that booking and coupon | PROPOSED (P-027) |
| DR-AB9 | Student suspended or inactive → excluded from new generation; existing coupons stay `ACTIVE` but are rejected at entry with `STUDENT_NOT_ACTIVE` while the status lasts | PROPOSED (P-031, Q-25) |
| DR-AB10 | Entry-time safety net: when an eligible hosteller has **no booking row** for the current service and DR-AB3 is satisfied, the redemption transaction creates it with the same eligibility function. No row is ever created over an existing `OPTED_OUT` or `VOIDED` row | PROPOSED (P-028) |
| DR-AB11 | Each run records a `JobRun` with per-service counts (created, revived, voided, skipped) and one audit event; each booking stores its `generationRunId` | APPROVED (D-041); detail PROPOSED (P-030) |
| DR-AB12 | Automatic booking, opt-out and revival never change `plannedQuantity` | APPROVED (D-024) |
| DR-AB13 | One function `isEligibleForMeal(student, mealService)` is used by the generator, the reconciliation pass and the entry-time check | PROPOSED (ARCH) |

### 4.2 Opt-out
| ID | Rule | Status |
|---|---|---|
| DR-OO1 | Opt-out is available only if the meal service's `optOutEnabled = true` (default false, Q-06) | APPROVED mechanism (D-038); enablement UNRESOLVED |
| DR-OO2 | Allowed only while `now < cutoffAt` | APPROVED (D-038, D-014) |
| DR-OO3 | Effect: booking → `OPTED_OUT`, coupon → `VOID` (`OPTED_OUT`); other meals untouched | APPROVED (D-038) |
| DR-OO4 | Undo of an opt-out is not available (Q-06) | UNRESOLVED (safe default) |
| DR-OO5 | Idempotent: repeating the request returns the same result; opting out of a redeemed coupon is impossible (`ALREADY_REDEEMED`) | PROPOSED (P-015) |
| DR-OO6 | Each opt-out writes an audit event | APPROVED (D-041) |
| DR-OO7 | Meal Admin may reopen the opt-out window of one service until a new time, with a reason; audited (D-046) | APPROVED |

### 4.3 Booking state machine
```text
 AUTO (hosteller, ₹0):
   (none) ──generate / entry safety net──► CONFIRMED
   CONFIRMED ──opt-out before cutoff (if enabled)──► OPTED_OUT        (terminal; never revived)
   CONFIRMED ──meal service voided by admin─────────► VOIDED[SERVICE_VOIDED]   (terminal)
   CONFIRMED ──membership ended─────────────────────► VOIDED[MEMBERSHIP_ENDED]
   VOIDED[MEMBERSHIP_ENDED] ──eligible again before generation limit──► CONFIRMED (revive, audited)

 DEMO (demo students only, DEMO_MODE=true, P-013):
   PENDING_PAYMENT ──attempt SUCCEEDED──► CONFIRMED
   PENDING_PAYMENT ──attempt FAILED / CANCELLED──► PENDING_PAYMENT (a new attempt may be made)
   PENDING_PAYMENT ──cutoff passes──► EXPIRED (terminal)
```

## 5. Simulated payment (demo only)
| ID | Rule | Status |
|---|---|---|
| DR-P1 | Available only when `DEMO_MODE=true`, only to demo (non-hosteller) students | PROPOSED (P-013) |
| DR-P2 | Hosteller bookings never involve a payment attempt | APPROVED (D-008) |
| DR-P3 | Never collects card, UPI, bank or credential data. Every screen is labelled **DEMO / SIMULATED — NO MONEY CHARGED** | APPROVED (V2) |
| DR-P4 | Attempt outcomes: `SUCCEEDED`, `FAILED`, `CANCELLED`, `PENDING` (later resolved or `EXPIRED`) | APPROVED (V3) |
| DR-P5 | Only the server records outcomes and moves the booking; client callbacks are ignored | APPROVED (V2) |
| DR-P6 | A demo success creates a coupon with `fundingSource = SIMULATED_PAYMENT`, `isDemo = true`; pilot checkpoints reject it (`DEMO_COUPON_NOT_ACCEPTED`) | APPROVED intent (D-009); mechanism PROPOSED |
| DR-P7 | Payment sits behind a `PaymentProvider` interface; `SimulatedPaymentProvider` is the only V1 implementation | APPROVED (V3) |

Payment attempt states: `CREATED → PENDING → SUCCEEDED | FAILED | CANCELLED | EXPIRED`.

## 6. Meal coupon (entitlement)
| ID | Rule | Status |
|---|---|---|
| DR-K1 | Exactly one coupon per booking that has been confirmed at least once | PROPOSED (P-010) |
| DR-K2 | A coupon is bound to one student and one meal service | APPROVED (V3) |
| DR-K3 | Coupons are not transferable | APPROVED working default (V2) |
| DR-K4 | Coupon data is never placed in any QR code | APPROVED (D-020, D-033) |
| DR-K5 | After the window (plus approved grace, Q-08) an `ACTIVE` coupon becomes `EXPIRED`, evaluated lazily at read/redeem time and by a sweep | PROPOSED (P-018) |

Coupon states: `ACTIVE → REDEEMED` (only via the redemption service) · `ACTIVE → EXPIRED` · `ACTIVE → VOID[reason]`, where reason is `OPTED_OUT`, `SERVICE_VOIDED` or `MEMBERSHIP_ENDED` · `VOID[MEMBERSHIP_ENDED] → ACTIVE` (generator revival only). All other states are terminal.

## 7. Entrance, entitlement and redemption
| ID | Rule | Status |
|---|---|---|
| DR-R1 | Identity verification (biometric, QR plus session, or staff-assisted) **never** authorises a meal by itself | APPROVED (D-021) |
| DR-R2 | All verification methods call one `RedemptionService` | APPROVED (D-021, D-040) |
| DR-R3 | Entitlement check, in order: student `ACTIVE` (`STUDENT_NOT_ACTIVE`); eligible for this service (DR-E3; `NOT_ELIGIBLE`); a current meal service exists at this checkpoint's mess (`OUTSIDE_SERVICE_WINDOW`); a coupon exists (after the DR-AB10 safety net; otherwise `NO_ENTITLEMENT`, or `OPTED_OUT` if opted out); coupon `ACTIVE`; booking `CONFIRMED`; not demo unless the checkpoint accepts demo coupons | APPROVED (V3, D-037) + PROPOSED details |
| DR-R4 | Exactly one `Redemption` per coupon, enforced by a **unique constraint** on `couponId` plus a conditional status update in one transaction | APPROVED (D-022) |
| DR-R5 | Every attempt, successful or not, writes an `EntryAttempt` row with method, terminal, outcome and reason code | APPROVED (V3 audit) |
| DR-R6 | Redemption outside the service window is rejected (`OUTSIDE_SERVICE_WINDOW`). Grace: Q-08 | PROPOSED (P-006) |
| DR-R7 | An already-redeemed coupon is reported as `ALREADY_REDEEMED`, with a reference to the existing redemption, and never redeemed again | APPROVED (D-022) |
| DR-R8 | Live pilot verification methods: entrance QR and staff-assisted recovery. Biometric is disabled in the pilot environment | APPROVED (D-039, D-040) |

### Entrance QR challenge rules
| ID | Rule | Status |
|---|---|---|
| DR-Q1 | Displayed **by the entrance terminal**; scanned **by the student** in the authenticated BiteCampus app | APPROVED (D-019) |
| DR-Q2 | Token ≥ 256 bits from a CSPRNG; only `SHA-256(token)` is stored | APPROVED (D-020); size and hash ARCH |
| DR-Q3 | Bound to one terminal and checkpoint; TTL and rotation per P-007 | APPROVED intent; values PROPOSED |
| DR-Q4 | Single-use: consumed atomically by the successful redemption it enables (P-008) | APPROVED (D-020); consumption rule PROPOSED |
| DR-Q5 | Expired, consumed, superseded or unknown challenges are rejected; a photographed QR is useless after expiry or consumption | APPROVED (D-020) |
| DR-Q6 | QR payload format: `BC1.<base64url token>`; no PII, no coupon or user ID | PROPOSED (ARCH) |
| DR-Q7 | The challenge lifecycle (`ISSUED → CONSUMED / EXPIRED / SUPERSEDED`) is independent of the coupon lifecycle | APPROVED (D-020) |

### Biometric simulator rules
| ID | Rule | Status |
|---|---|---|
| DR-V1 | Outcomes: `MATCH`, `NO_MATCH`, `DEVICE_UNAVAILABLE`, `TIMEOUT` | APPROVED (D-018) |
| DR-V2 | The simulator emits a signed identity event `{eventId, terminalId, subjectRef, outcome, occurredAt}`; `subjectRef` is an opaque synthetic reference mapped to a `Student`, never biometric data | APPROVED (D-018, V2 G2) |
| DR-V3 | Only `MATCH` proceeds to the entitlement check; the other outcomes prompt QR fallback | APPROVED (V3) |
| DR-V4 | `eventId` is the idempotency key for biometric redemption | PROPOSED (ARCH) |
| DR-V5 | Enabled only in local, test and demo environments; every simulator screen says **SIMULATOR**; disabled in the live pilot | APPROVED (D-039) |

### Staff-assisted recovery rules
| ID | Rule | Status |
|---|---|---|
| DR-S1 | Available in the live pilot to holders of `entry.assist` | APPROVED (D-040) |
| DR-S2 | Staff search the student and see today's entitlement status only (no unrelated personal data) | PROPOSED (P-012) |
| DR-S3 | A reason code is mandatory; the identity check required of staff is Q-10 | APPROVED intent; detail UNRESOLVED |
| DR-S4 | Uses the same `RedemptionService` with method `STAFF_ASSISTED`; audited with the staff member's identity | APPROVED (D-021, D-040) |
| DR-S5 | Staff-assisted recovery restores **identity**, not entitlement. It cannot redeem for a student who has no valid coupon | APPROVED (D-037) |

## 8. Gate controller and reconciliation
| ID | Rule | Status |
|---|---|---|
| DR-G1 | A gate command is sent only **after** the redemption transaction commits | APPROVED (D-023) |
| DR-G2 | Gate outcomes: `OPENED`, `REJECTED`, `FAULT`, `TIMEOUT` (outcome unknown) | APPROVED (V3) |
| DR-G3 | Gate command states: `SENT → ACKNOWLEDGED_OPEN | FAILED | UNKNOWN`; `UNKNOWN → ACKNOWLEDGED_OPEN | FAILED` via reconciliation | PROPOSED (ARCH) |
| DR-G4 | On `UNKNOWN`, the server queries the controller using the command ID. If the outcome is still unknown, the command goes to the reconciliation queue | PROPOSED (P-016) |
| DR-G5 | A retry creates a **new gate command for the same redemption**. It never calls the redemption service again | APPROVED (D-023) |
| DR-G6 | Retry window, maximum retries and override authority: Q-09 | UNRESOLVED (safe default in OPEN_DECISIONS) |
| DR-G7 | The student-facing result distinguishes "Meal redeemed — entry confirmed", "Meal redeemed — entry pending/retry" and "Not redeemed — reason" | APPROVED (V3) |

## 9. Kitchen planned quantity
| ID | Rule | Status |
|---|---|---|
| DR-KQ1 | `plannedQuantity`, booked count (confirmed bookings), opted-out count and redeemed (attended) count are separate values | APPROVED (D-024) |
| DR-KQ2 | No automatic change to `plannedQuantity` from automatic booking, opt-out, voiding or revival | APPROVED (D-024) |
| DR-KQ3 | Manual adjustment requires `kitchen.plan.adjust`, a reason and the previous/new values; it writes an audit event | APPROVED (D-024) |
| DR-KQ4 | Initial value and baseline: Q-12 | UNRESOLVED |
| DR-KQ5 | Booked counts may be shown as forecasting input only; the UI labels them "Booked", never "To prepare" | APPROVED (D-024) |

## 10. Feedback
| ID | Rule | Status |
|---|---|---|
| DR-F1 | A signed-in student may leave feedback only for a meal service they held a coupon for | PROPOSED (ARCH) |
| DR-F2 | One feedback entry per student per meal service; not editable (edit window unresolved) | PROPOSED (ARCH) |
| DR-F3 | Identifiability notice is shown before submission (Q-13) | APPROVED (V2) |

## 11. Audit
| ID | Rule | Status |
|---|---|---|
| DR-A1 | Audit events are append-only; no update or delete path in the application | APPROVED (V3) |
| DR-A2 | Audited: role changes, roster imports and membership changes, generation runs, opt-outs, voids and revivals, config/price/menu changes, planned-quantity adjustments, redemptions, entry attempts, gate commands, reconciliation actions, staff-assisted recoveries, simulator mode changes | APPROVED (V2, V3, P2, D-041) |
| DR-A3 | Audit never records tokens, secrets, raw QR values or session IDs | APPROVED (V2) |
