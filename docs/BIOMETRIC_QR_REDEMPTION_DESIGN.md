# BIOMETRIC_QR_REDEMPTION_DESIGN

The design for verification at the mess entrance, entitlement authorisation, exactly-once redemption and gate response.
Decisions: D-018 – D-023, D-033. Rules: DOMAIN_RULES §7–§8. Schema: DATABASE_DESIGN §2. Endpoints: API_CONTRACTS §4–§6.

## 1. Four separate questions
| Step | Question | Answered by | Durable record |
|---|---|---|---|
| 1. Identity verification | *Who* is at this entrance now? | QR challenge + authenticated session **or** staff-assisted recovery (live pilot, D-040) **or** biometric simulator signed event (non-pilot environments, D-039) | `BiometricEvent` / `QrChallenge` / `EntryAttempt` |
| 2. Entitlement authorisation | May this person eat **this** meal now? | `RedemptionService` checks DR-R3 | `EntryAttempt` (reason) |
| 3. Coupon redemption | Record the single consumption | `RedemptionService` (transaction) | `Redemption` (UNIQUE couponId) |
| 4. Physical entry response | Did the gate let them in? | `GateController` (simulated) after commit | `GateCommand` |

Steps 1–3 happen in BiteCampus's database transaction. Step 4 is outside it and may fail **independently**. A failure at step 4 never undoes step 3, and recovering step 4 never repeats step 3.

## 2. Entrance QR challenge vs meal coupon
| | Entrance QR challenge | Meal coupon |
|---|---|---|
| Purpose | Prove "an authenticated student is at terminal T now" | Represent "student S may eat meal M on date D" |
| Created by | Terminal request, every 15 s | Automatic daily booking (or the entry-time safety net, P-028); demo checkout for demo students |
| Shown where | **On the entrance terminal screen** | In the student's app (no QR) |
| Lifetime | ~30 s | Until redeemed / window end |
| Contains | Random 256-bit token (`BC1.<token>`) | Nothing is ever encoded; server record only |
| Stored as | SHA-256 hash | Row in `MealCoupon` |
| States | ISSUED → CONSUMED / EXPIRED / SUPERSEDED | ACTIVE → REDEEMED / EXPIRED / VOID |

They are linked only after the fact (`QrChallenge.redemptionId`).

## 3. RedemptionService contract
```ts
type VerifiedIdentity =
  | { method: 'QR';             studentId: string; qrChallengeId: string; terminalId: string }
  | { method: 'BIOMETRIC';      studentId: string; biometricEventId: string; terminalId: string }
  | { method: 'STAFF_ASSISTED'; studentId: string; actorUserId: string; reasonCode: string; checkpointId: string };

redeem(tx, identity: VerifiedIdentity, checkpointId: string, now: Instant):
  Promise<{ outcome: 'REDEEMED', redemption } | { outcome: 'REJECTED', reasonCode }>
```
Identity resolves to a **`Student`** (P-023): QR via the session user's linked student, biometric via `BiometricSubject`, staff-assisted via staff search.

Inside the transaction the service: (1) checks student status and eligibility (DR-R3, D-037); (2) finds the coupon for the current meal service, or creates it through `ensureAutoBooking` if the generator missed it (P-028, never over an opt-out); (3) redeems.

The service does **not** know how identity was verified beyond the tag. That is the convergence point (D-021). Verification adapters must finish, and in the QR case lock the challenge, before calling it.

## 4. QR fallback sequence
```text
Terminal          API                         PostgreSQL                 Gate simulator      Student phone
   │ POST /terminal/qr-challenges (HMAC)        │                              │                   │
   │──────────────►│ token=CSPRNG(32B)          │                              │                   │
   │               │ INSERT QrChallenge(hash)──►│                              │                   │
   │◄──────────────│ payload BC1.<token>        │                              │                   │
   │ display QR                                                                                    │
   │                                                                     scan in BiteCampus app ◄──│
   │               │◄──────── POST /entry/qr-redemptions {challengeToken} + session + Idem-Key ───│
   │               │ BEGIN                                                                        │
   │               │ idempotency check/insert ─────────────►│                                     │
   │               │ SELECT challenge by hash FOR UPDATE ───►│ (ISSUED? unexpired? terminal active?)│
   │               │ student ACTIVE + membership eligible; current MealService for checkpoint     │
   │               │ coupon FOR UPDATE (or ensureAutoBooking → booking+coupon if missing, P-028)   │
   │               │ UPDATE coupon SET REDEEMED WHERE status='ACTIVE'  (1 row)                    │
   │               │ INSERT Redemption (UNIQUE couponId)                                          │
   │               │ UPDATE challenge SET CONSUMED, consumedBy, redemptionId                      │
   │               │ INSERT EntryAttempt, AuditEvent; store idempotent response                    │
   │               │ COMMIT ────────────────────────────────►│                                     │
   │               │ INSERT GateCommand(SENT) ; open(cmd) ───────────────────────►│                │
   │               │◄──────────────────── OPENED / REJECTED / FAULT / (timeout) ──│                │
   │               │ UPDATE GateCommand status                                                     │
   │               │──────── 200 { REDEEMED, entry: CONFIRMED|PENDING|FAILED } ──────────────────►│
   │◄─ GET /terminal/events: CHALLENGE_CONSUMED, ENTRY_RESULT → show result, display new QR        │
```
**Failure ordering:** if any entitlement check fails, the transaction writes only the `EntryAttempt` (in a separate short transaction) and the challenge stays `ISSUED` (P-008). Challenge validation errors (`QR_EXPIRED`, `QR_ALREADY_USED`, `QR_INVALID`) are returned before any entitlement lookup, so a bad QR never reveals entitlement information.

## 5. Biometric sequence
0. Environment check: the route exists only where the simulator is enabled (local, test, demo, training). In the live pilot it returns 404 (D-039).
1. The terminal simulator, labelled **SIMULATOR**, produces an outcome (controls select outcome and synthetic subject).
2. Non-MATCH → the terminal shows QR fallback locally and also reports the event (for metrics).
3. MATCH → `POST /terminal/biometric-events` (HMAC, `eventId`).
4. API: `INSERT BiometricEvent` (UNIQUE terminalId+eventId → replays return the stored outcome) → map `subjectRef → studentId` → `RedemptionService.redeem` → commit → gate command → response to the terminal.

## 5a. Staff-assisted recovery (live pilot, D-040)
Counter Staff search the student → see only the current entitlement → perform the identity check (Q-10) → submit a reason code with an `Idempotency-Key` → `RedemptionService.redeem({ method: 'STAFF_ASSISTED', … })` → normal gate flow. It restores **identity**, never entitlement (DR-S5).

## 6. Why each guard exists (interview view)
| Guard | Prevents | Without it |
|---|---|---|
| UNIQUE(Redemption.couponId) | Two committed redemptions of one coupon | Race between QR and biometric creates two rows |
| Conditional coupon update | Redeeming a non-ACTIVE coupon | An EXPIRED or VOID coupon could be redeemed if the check and the write were separate |
| Conditional challenge consume / `FOR UPDATE` | Two students using one QR | Both pass the "ISSUED?" check before either writes |
| Idempotency record | Retries returning errors or creating new effects | A phone retry after a lost response shows "Already redeemed" or, worse, a second effect |
| Hash-only token storage | DB leak → usable live tokens | Anyone reading the table could redeem during the TTL |
| Gate after commit | Gate open without a durable redemption; locks held during I/O | The gate opens, the transaction rolls back → free meal, no record |
| Full UNIQUE(studentId, mealServiceId) on bookings | Duplicate auto bookings; generator resurrecting an opt-out | Two coupons for one meal → two redemptions possible; a skipped meal silently reappears |
| Reconciliation instead of re-redeem | Double counting when the gate outcome is unknown | A retry after a timeout consumes the coupon twice or blocks a legitimate student |

## 7. Gate outcome handling
| Simulator result | GateCommand | Student sees | Next |
|---|---|---|---|
| OPENED within 3 s | ACKNOWLEDGED_OPEN | Entry confirmed | — |
| REJECTED / FAULT | FAILED | Redeemed — gate failed; Retry entry | Retry → new GateCommand (attemptNo+1) |
| No reply in 3 s | UNKNOWN | Redeemed — entry pending | Status query; then queue; retry allowed |
| Status query says OPENED | ACKNOWLEDGED_OPEN | Entry confirmed (on refresh) | — |

Retry limits, the window and override authority are UNRESOLVED (Q-09); the safe defaults are in OPEN_DECISIONS §4.

## 8. Simulator modes (demo/test)
- **Biometric:** `MATCH(subject)`, `NO_MATCH`, `DEVICE_UNAVAILABLE`, `TIMEOUT(after ms)`.
- **Gate:** `OPEN`, `REJECT`, `FAULT`, `TIMEOUT` (never answers), `DELAYED(ms)` (answers after the timeout so status queries can find `OPENED`), `RANDOM(seed)` for soak tests.
Mode changes are audited and only possible with `DEMO_MODE=true`.

## 9. Tests that prove the design (all against real PostgreSQL)
AC-QR-02…09, AC-BIO-01…05, AC-RED-01…04, AC-GATE-01…04. Concurrency tests fire N parallel HTTP requests with `Promise.all` through Supertest against a test database. They assert row counts directly in the database, not just HTTP responses.

## 10. V2 replacement path
- `BiometricVerifier` → adapter that receives signed events from the college/vendor system (opaque college person ID, never biometric data; V2 G1–G2).
- `GateController` → adapter to the college access-control owner's interface, only after a safety review (V2 G5, G10).
- The QR challenge, `RedemptionService` and schema remain unchanged.
