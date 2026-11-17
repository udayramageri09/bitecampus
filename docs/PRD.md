# PRD — BiteCampus Product Requirements

> Requirement IDs (`FR-*`) are defined in `FUNCTIONAL_REQUIREMENTS.md`. Business values live in `DOMAIN_RULES.md`. Statuses and sources are defined in `OPEN_DECISIONS.md`.

## 1. Context and goal
BiteCampus starts as a real, controlled pilot in one BMSCE hostel mess with approximately 500 users (D-002). It must be practical enough for daily operation and engineered well enough to demonstrate and defend in technical interviews.

The pilot does not imply college endorsement. Operational use with real student data requires written college authorisation (D-030, Q-18).

## 2. Personas
| Persona | Need | Source |
|---|---|---|
| Hostel resident (Student) | Be fed without booking chores; know what is served; skip a meal if allowed; get in without friction; see history | V2, V3, P2R |
| Counter Staff (entrance) | See clear entry outcomes; resolve stuck cases | V2, ES |
| Inventory Admin | Know planned quantity vs bookings vs attendance; adjust preparation plans deliberately; (V1.1) stock, usage, waste | BL, P2 |
| Meal Admin | Manage menus, prices, windows, cutoffs | V2, ES |
| Main Admin | Configure academic year and hostel meal rules; import the College IT roster; provision roles; cross-department oversight | BL |
| Hostel Director (permission) | Approve refunds (V1.1) | BL |
| Staff Manager | Kitchen-staff attendance (V1.1) | BL |
| Guest / Recruiter | Event coupons (V1.1) / isolated demo (demo environment) | BL |

## 3. Key concepts (glossary)
| Term | Meaning |
|---|---|
| **Meal slot** | A configured meal category (Breakfast, Lunch, Snacks, Dinner) with its default window, cutoff rule, list price and flags (`bookingEnabled`, `autoBookEnabled`, `optOutEnabled`) |
| **Student** | The roster person who receives entitlements, whether or not they ever sign in. A Google login (`User`) links to it (P-023) |
| **Hostel membership** | A dated record that a student belongs to a hostel mess. Required for automatic booking, but never sufficient for entry (D-037) |
| **Meal service** | One concrete instance of a slot on one campus-local date (e.g. Lunch on 2026-12-01), with its menu, price, window and planned quantity |
| **Automatic daily booking** | The system-created reservation of a meal service for an eligible hosteller (`source = AUTO`, ₹0, `HOSTEL_PREPAID`). Hostellers never book manually (D-034). Demo bookings (`source = DEMO`) exist only for demo students |
| **Opt-out** | A hosteller skipping one meal before its cutoff, if enabled (D-038, Q-06) |
| **Meal coupon** | The durable server-side **entitlement** created with a confirmed booking (P-010). The "digital coupon" is how the app shows it. It is never put in a QR code |
| **Entrance terminal** | An authenticated kiosk at the mess entrance. It runs the biometric simulator and displays the rotating entrance QR |
| **Entrance QR challenge** | A short-lived, single-use, random token shown by the terminal and scanned by the student's phone. It proves that the student is physically at that terminal now. It is **not** a coupon |
| **Identity verification** | Establishing *who* is at the entrance: entrance QR plus authenticated session, or staff-assisted recovery (live pilot, D-040); biometric simulator (non-pilot environments, D-039) |
| **Entitlement authorisation** | Server check that this person holds a valid, unredeemed, non-demo coupon for the current meal service |
| **Redemption** | The single durable record that a coupon was used. At most one per coupon |
| **Gate command** | An instruction to the gate-controller simulator to admit someone for an existing redemption. It can be retried; the redemption itself is never repeated |
| **Reconciliation** | Resolving a gate command whose outcome is unknown, without creating a second redemption |
| **Planned quantity** | The kitchen's intended preparation quantity for a meal service. It is never changed automatically by bookings or opt-outs (D-024) |

## 4. Scope

### 4.1 Pilot (V1.0) — APPROVED (D-025)
| # | Capability | FR group |
|---|---|---|
| 1 | Student authentication and authorisation | FR-AUTH |
| 2 | Student dashboard | FR-DASH |
| 3 | Meal menu and pricing | FR-MEAL |
| 4 | Automatic daily meal booking (with opt-out mechanism, enablement pending Q-06) | FR-AB, FR-OO |
| 5 | ₹0 prepaid hostel entitlement | FR-AB |
| 6 | Digital coupons | FR-COUP |
| 7 | Biometric verification simulator (development, testing, training, demonstrations; not live pilot, D-039) | FR-BIO |
| 8 | Dynamic (rotating) entrance QR fallback | FR-QR |
| 9 | Server-side coupon entitlement validation | FR-RED |
| 10 | Exactly-once durable redemption | FR-RED |
| 11 | Gate-controller simulator | FR-GATE |
| 12 | Booking and redemption history | FR-HIST |
| 13 | Required staff/admin roles and permissions | FR-RBAC |
| 14 | Menu and pricing administration | FR-ADMIN |
| 15 | Audit logs | FR-AUDIT |
| 16 | Essential operational dashboards (planned / auto-booked / opted out / redeemed; generator health) | FR-OPS |
| 17 | Feedback | FR-FB |

**Live pilot verification (D-039, D-040):** the live pilot uses the entrance QR flow and authorised staff-assisted recovery. The biometric simulator is built and tested, but is disabled in the pilot environment and never presented as real verification.

Supporting items needed for the pilot to work: staff-assisted recovery (APPROVED, D-040; details P-012/Q-10), roster and membership import (PROPOSED, P-011, P-029), and demo-mode simulated checkout for demo students only (PROPOSED, P-013).

### 4.2 V1.1 / V1.2 — DEFERRED (D-026)
Kitchen and inventory (F-001); advanced ration and waste reporting (F-002); event organiser workflows (F-003); complimentary guest coupon approval and distribution (F-004); external guest coupon claiming (F-005); advanced import/export (F-006); extended notifications (F-007); additional reporting/admin (F-008). Their requirements are kept in `FUNCTIONAL_REQUIREMENTS.md` §14.

### 4.3 V2 — DEFERRED, conditional on college authorisation
Real payment gateway (F-009), real biometric integration (F-010), physical gate integration (F-011). The V2 guardrails (D-029) apply to the design now.

### 4.4 Recruiter demo and landing page
Isolated demo environment per D-048 (synthetic data, 7-day reset/expiry), built with M9; details in Q-16. Paid diner flow, refunds and staff attendance are V1.1 (D-044, F-015).

### 4.5 Explicit non-goals for V1
- Real money movement of any kind.
- Storing biometric images, templates or samples.
- Controlling physical gate hardware.
- Paid user categories such as day scholars (D-010).
- Microservices, Redis, message queues or other infrastructure without a measured need (D-003).

## 5. Core journeys (details in `USER_FLOWS.md`)
1. **Sign in:** Google → server verifies token, domain and active roster → session.
2. **Automatic booking:** every day, the system books Breakfast, Lunch and Dinner for each eligible hosteller (₹0, `HOSTEL_PREPAID`) and issues `ACTIVE` coupons. No student action is needed.
2a. **Opt-out (if enabled):** skip one meal before its cutoff → that coupon is voided.
3. **Enter (biometric, non-pilot environments):** terminal biometric simulator → identity event → redemption service → gate command → result shown on the terminal.
4. **Enter (QR fallback):** biometric fails or is unavailable → terminal displays a rotating QR → student scans it in the BiteCampus app → server validates the challenge, session and entitlement → atomic redemption → gate command → result shown on the student's phone and the terminal.
5. **Uncertain gate:** redemption committed, gate response times out → reconciliation; retrying entry reuses the existing redemption.
5a. **Staff-assisted recovery:** Counter Staff verify the student in person → same redemption service.
6. **Operate:** Main Admin sets the academic year and imports the roster; Meal Admin edits menus, voids closures and reopens windows (audited); Inventory Admin sets and adjusts planned quantity (audited); staff watch planned / auto-booked / opted-out / redeemed counts and generator health.
7. **Feedback:** the student rates a meal service they were entitled to.

## 6. Constraints
- Server authoritative for identity, roles, prices, cutoffs, booking, entitlement, redemption and gate state (V2).
- All business values configurable (D-017); unresolved values fail safe (OPEN_DECISIONS §4).
- Windows-first developer environment; learning contract governs every feature (D-027).
- Git ownership rules (D-028).

## 7. Release criteria for the pilot
See `ACCEPTANCE_CRITERIA.md` §0 (pilot exit criteria) and the per-feature criteria.
