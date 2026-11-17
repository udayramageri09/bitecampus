# ARCHITECTURE

> **Subject to the read-only repository audit.** The existing application may already have a structure. Adapt this document to sound existing code rather than forcing a rewrite (D-004).

## 1. Style
**Modular monolith** (D-003): one Express API process, one React SPA, one PostgreSQL database. Code is organised by business module. Modules talk to each other through exported service interfaces, never through each other's tables.

Why not microservices: about 500 users, one team of one, and strong cross-entity transactions (booking + coupon, challenge + coupon + redemption) that are trivial in one database and hard across services. No Redis or queues: PostgreSQL row locks, unique constraints and a small job table cover idempotency, rotation and reconciliation at this scale (NFR-20).

## 2. Context diagram
```text
 Student phone (BiteCampus SPA)        Staff browser (SPA, staff routes)
          │  HTTPS + session cookie            │
          ▼                                    ▼
 ┌─────────────────────────── BiteCampus API (Express) ───────────────────────────┐
 │ identity · catalog · booking · payments(sim) · coupons · entry · redemption ·   │
 │ gate · operations · feedback · audit · platform                                 │
 └───────────┬──────────────────────────┬──────────────────────────┬───────────────┘
             │ Prisma                   │ HMAC-signed HTTPS        │ in-process adapter
             ▼                          ▲                          ▼
        PostgreSQL           Entrance terminal (kiosk SPA)   Gate-controller simulator
                             · biometric simulator            (V2: real controller adapter)
                             · rotating QR display
        Google Identity (token verification keys)
```

## 3. Modules
| Module | Owns (tables) | Exposes | Pilot |
|---|---|---|---|
| `identity` | User, Role, Permission, UserRole, Session | `verifyGoogleSignIn`, `getPrincipal`, `requirePermission` | ✔ |
| `residency` | Student, Mess, HostelMembership, RosterImportBatch | `isEligibleForMeal(student, mealService)` (DR-AB13), roster import, membership changes | ✔ |
| `catalog` | MealSlotConfig, MealService, MenuItem | `ensureMealServices(horizon)`, `getCurrentService(checkpoint, now)` | ✔ |
| `booking` | Booking | `AutoBookingGenerator.run(horizon)`, `ensureAutoBooking(student, service)` (safety net), `optOut`, `createDemoBooking` | ✔ |
| `payments` | PaymentAttempt | `PaymentProvider` (Simulated) | demo only |
| `coupons` | MealCoupon | `issueCoupon`, `findRedeemable` | ✔ |
| `entry` | Checkpoint, EntranceTerminal, QrChallenge, BiometricEvent, EntryAttempt | `issueChallenge`, `verifyQr`, `verifyBiometricEvent`, `BiometricVerifier` (Simulated) | ✔ |
| `redemption` | Redemption | `RedemptionService.redeem` — the only writer of `REDEEMED` | ✔ |
| `gate` | GateCommand | `GateController` (Simulated), `reconcile` | ✔ |
| `operations` | PlannedQuantityAdjustment | dashboards, planned quantity | ✔ |
| `feedback` | Feedback | create/list | ✔ |
| `audit` | AuditEvent | `audit.record(tx, event)` | ✔ |
| `platform` | IdempotencyRecord, JobRun | config, errors, logging, clock, idempotency middleware | ✔ |
| `kitchen-inventory` | — | — | V1.1 (F-001/F-002) |
| `events-guests` | — | — | V1.1 (F-003–F-005) |
| `notifications`, `reporting` | — | — | V1.1 (F-007/F-008) |

Dependency direction: `booking → residency, catalog, coupons`; `entry → redemption → booking (safety net), coupons, catalog, residency`; `redemption → audit`; `gate` is called by the entry application service after commit; nothing depends on `gate` inside a transaction.

## 4. Backend request flow
`route → auth (session or terminal HMAC) → requirePermission → Zod validation → idempotency middleware (where applicable) → application service → Prisma transaction → response mapper → error handler`

- Routes are thin; services hold rules; Prisma calls stay inside module repositories or services.
- One error class hierarchy (`DomainError` with a code) mapped to the envelope in `API_CONTRACTS.md`.
- Services take injected `Clock`, `Random`, `PaymentProvider`, `BiometricVerifier` and `GateController` (NFR-13, NFR-18).

## 5. Integration boundaries (replaceable)
```ts
interface PaymentProvider   { createAttempt(...); reportOutcome(...); }            // Simulated in V1
interface BiometricVerifier { verify(terminalId, sample?): Promise<BiometricResult> } // Simulated; never receives biometric data in V1
interface GateController    { open(cmd: GateCommandRef): Promise<GateOutcome>;
                              status(commandId): Promise<GateOutcome> }               // Simulated in V1
```
Real V2 adapters implement the same interfaces behind configuration (F-009–F-011).

## 6. Frontend
- One React SPA with three route areas: `/app` (student), `/staff` (permission-gated) and `/terminal` (kiosk, device-authenticated). They could be split later if needed.
- Feature folders: `auth`, `meals`, `bookings`, `coupons`, `entry` (scanner), `history`, `feedback`, `staff/*`, `terminal`.
- A typed API client in one place; server state via a query library (TBD by audit); no business rules in the UI (DR-R1).
- QR scanning uses a maintained browser library over `getUserMedia` (chosen in implementation; requires HTTPS).

## 7. Repository layout (proposed; adapt to the existing repo)
```text
apps/web/src/{app,features,components,lib}
apps/api/src/{app.ts,server.ts,config,platform,modules/<module>/{routes,service,schemas,repo}.ts}
prisma/{schema.prisma,migrations,seed.ts}
tests/{integration,e2e}
docs/  .claude/
```

## 8. Background work
No queue. A minimal in-process scheduler (or a cron-triggered endpoint, Q-17) runs idempotent jobs, each recorded in `JobRun`:
- **Automatic booking** — hourly, on server start, and by manual trigger, over the whole horizon (P-026). It is safe to run any number of times (D-035).
- Expire coupons (P-018); expire challenges; query `UNKNOWN` gate commands.

Jobs are plain service functions, so integration tests call them directly with a fixed `Clock`.

## 9. Deferred-module seams
- `events-guests` will add `GuestCoupon` (or a `holderType` on `MealCoupon`) and call `RedemptionService` with guest identity (FR-EVT-04).
- `kitchen-inventory` reads `operations` counts; it never writes `plannedQuantity` automatically (D-024).
- Paid user categories (F-013) would add a booking source that uses `PaymentProvider`; automatic hosteller booking is unaffected.
