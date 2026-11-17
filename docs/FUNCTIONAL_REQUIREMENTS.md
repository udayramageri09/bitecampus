# FUNCTIONAL_REQUIREMENTS

Each requirement has an ID, a status and a source (legend in `OPEN_DECISIONS.md`). Business values → `DOMAIN_RULES.md`. Acceptance tests → `ACCEPTANCE_CRITERIA.md`.

Status shorthand: **A** = Approved · **P** = Proposed · **D** = Deferred · **U** = Unresolved.

## 1. Authentication & eligibility — FR-AUTH
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-AUTH-01 | Sign in with Google; the server verifies the ID token (signature, issuer, audience, expiry, `email_verified`) | A | V2, V3 |
| FR-AUTH-02 | Reject emails outside `ALLOWED_EMAIL_DOMAINS` with the D-031 message | A | V2, V3 |
| FR-AUTH-03 | Reject users without an `ACTIVE` `Student` record, with a non-revealing message | A | V3 |
| FR-AUTH-04 | First eligible sign-in creates the `User`, links it to the matching `Student` and grants only the Student role | A role / P linking (P-023) | V2, ARCH |
| FR-AUTH-05 | Session per P-004; logout invalidates the session server-side | P | ARCH |
| FR-AUTH-06 | Rate-limit the sign-in endpoint | A | V2 |
| FR-AUTH-07 | `GET /me` returns profile, roles, permissions and eligibility (student status, current membership) | P | ARCH |

## 2. Student dashboard — FR-DASH
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-DASH-01 | Show today's and upcoming meal services in the horizon, with window, menu, list price, cutoff and the student's coupon status ("Booked automatically", "Skipped", "Redeemed", "Expired", "Cancelled by mess") | A | V3, P2R |
| FR-DASH-02 | Show the next active coupon prominently | A | V3 |
| FR-DASH-03 | Show "Covered by hostel mess fee — ₹0" alongside the list price | A | P2, P2R |
| FR-DASH-04 | Show the "Skip this meal" action only when opt-out is enabled for that service and its cutoff has not passed | A mechanism / U enablement | D-038, Q-06 |
| FR-DASH-05 | Loading, empty, error and success states on every view | A | V2 |

## 3. Meals & pricing — FR-MEAL
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-MEAL-01 | List meal services by campus-local date range | A | V2 |
| FR-MEAL-02 | Show booking-disabled slots (Snacks) as "Not available yet", without a price | A | P2 |
| FR-MEAL-03 | Price, cutoff and status are always computed by the server | A | V2 |

## 4. Automatic daily booking — FR-AB
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-AB-01 | Automatically create a ₹0 `HOSTEL_PREPAID` booking and an `ACTIVE` coupon for every eligible hosteller for each applicable meal (Breakfast, Lunch, Dinner) of each day in the horizon | A | D-034 |
| FR-AB-02 | Hostellers have **no** manual booking action and no payment step | A | D-034, D-008 |
| FR-AB-03 | Generation is idempotent: repeated, parallel or crashed-and-retried runs never create duplicates (DR-AB4, DR-AB5) | A | D-035 |
| FR-AB-04 | Generator triggers: schedule, server start, manual admin trigger; each run covers the whole horizon | A intent / P triggers (P-026) | D-035 |
| FR-AB-05 | Meal services for the horizon are generated automatically from slot configuration | P | P-024 |
| FR-AB-06 | Joining students get entitlements for services whose generation limit has not passed | A intent / U limit (Q-24) | D-042 |
| FR-AB-07 | Leaving students: unredeemed coupons after the membership end are voided; revived if eligibility returns in time | A intent / P mechanism (P-027) | D-042 |
| FR-AB-08 | Suspended/inactive students: excluded from generation; existing coupons blocked at entry | A intent / P semantics (P-031, Q-25) | D-042 |
| FR-AB-09 | Allocation change ends one membership and starts another; the generator follows the new mess | P | P-029 |
| FR-AB-10 | Entry-time safety net creates a missing booking for an eligible hosteller (never over an opt-out) | P | P-028 |
| FR-AB-11 | Each run records counts and writes an audit event; each booking records its run | A intent / P detail (P-030) | D-041 |
| FR-AB-12 | Generation never changes `plannedQuantity` | A | D-024 |

## 5. Opt-out — FR-OO
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-OO-01 | A hosteller can opt out of one meal service before its cutoff when `optOutEnabled = true` | A mechanism / U enablement (Q-06) | D-038 |
| FR-OO-02 | Opt-out voids only that meal's coupon (`OPTED_OUT`); other meals unaffected | A | D-038 |
| FR-OO-03 | After the cutoff, opt-out is rejected (`CUTOFF_PASSED`) | A | D-038 |
| FR-OO-04 | An opt-out is never undone by a generator re-run | A | D-038 |
| FR-OO-05 | Undo, limits and multi-day opt-outs: not available until Q-06 | U | Q-06 |
| FR-OO-06 | Opt-outs never change `plannedQuantity`; they are audited | A | D-024, D-041 |

## 6. Demo booking & simulated checkout — FR-DEMO
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-DEMO-01 | In demo mode only, a **demo (non-hosteller) student** can create a paid demo booking and complete a simulated checkout with success/failed/cancelled/pending outcomes | A outcomes / P gating (P-013) | V3, P2 |
| FR-DEMO-02 | Demo coupons are rejected by pilot checkpoints | A intent / P mechanism | D-009 |
| FR-DEMO-03 | A demo banner appears on every page when `DEMO_MODE=true` | P | P-013 |
| FR-DEMO-04 | The pilot shows a notice that it is an independent pilot, unless authorised wording exists | A | V2 |

## 7. Coupons — FR-COUP
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-COUP-01 | The digital coupon view shows meal, date, window, status and funding source. It shows **no QR code** | A | D-019, D-020 |
| FR-COUP-02 | Coupon status is always fetched from the server | A | V2 |
| FR-COUP-03 | Expire unredeemed coupons after the window (P-018) | P | ARCH |
| FR-COUP-04 | Void coupons when a meal service is voided (Q-15) | P | ARCH |

## 8. Biometric simulator — FR-BIO
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-BIO-01 | Terminal simulator produces MATCH / NO_MATCH / DEVICE_UNAVAILABLE / TIMEOUT | A | V3 |
| FR-BIO-02 | Behind a `BiometricVerifier` interface; no biometric data collected or stored | A | V3, V2 |
| FR-BIO-03 | MATCH sends a signed identity event; the server calls `RedemptionService` | A | V3 |
| FR-BIO-04 | Non-MATCH outcomes show "Use QR fallback" on the terminal and start QR display | A | P2 |
| FR-BIO-05 | Test/demo/training controls select the outcome and the synthetic subject; every screen says **SIMULATOR** | A | D-039 |
| FR-BIO-06 | Disabled in the live pilot environment | A | D-039 |

## 9. Entrance QR fallback — FR-QR
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-QR-01 | The entrance terminal requests and **displays** a rotating QR challenge | A | P2 |
| FR-QR-02 | The student scans it with the BiteCampus in-app scanner while signed in | A | P2 |
| FR-QR-03 | The app sends `{challengeToken}` + `Idempotency-Key`; identity comes from the session (linked `Student`) only | A | P2, P-023 |
| FR-QR-04 | The server validates the challenge (authentic, unexpired, unused, terminal active), then student status, eligibility and entitlement, then redeems atomically | A | P2, D-037 |
| FR-QR-05 | Clear outcomes for expired, invalid or already-used challenge, no entitlement, opted out, student not active, already redeemed, outside window, network failure | A | P2 |
| FR-QR-06 | The terminal displays a fresh challenge after expiry/rotation and immediately after a consumption | P | P-007 |
| FR-QR-07 | The scanner accepts only payloads matching `BC1.<token>`; anything else is rejected client-side without a network call | P | DR-Q6 |

## 10. Entitlement & redemption — FR-RED
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-RED-01 | One `RedemptionService` is used by biometric, QR and staff-assisted verification | A | D-021, D-040 |
| FR-RED-02 | Entitlement checks per DR-R3 | A | V3, D-037 |
| FR-RED-03 | At most one durable redemption per coupon under any concurrency | A | D-022 |
| FR-RED-04 | Idempotent replay: the same idempotency key (or biometric `eventId`) returns the original outcome | A | V3 |
| FR-RED-05 | Every attempt recorded as `EntryAttempt` with a reason code | A | V3 |
| FR-RED-06 | Staff-assisted recovery by `entry.assist` holders with a mandatory reason code; identity check per Q-10 | A mechanism / U check | D-040, Q-10 |

## 11. Gate controller — FR-GATE
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-GATE-01 | `GateController` interface with a simulator implementation; modes OPEN, REJECT, FAULT, TIMEOUT, DELAYED | A modes per V3 / P naming | V3 |
| FR-GATE-02 | Gate command sent only after the redemption commits | A | D-023 |
| FR-GATE-03 | Timeout → status query → reconciliation queue | A intent / P mechanism | D-023, P-016 |
| FR-GATE-04 | Retry entry reuses the existing redemption | A | D-023 |
| FR-GATE-05 | Reconciliation queue view and resolve action for authorised staff | P | ARCH, Q-09 |

## 12. History — FR-HIST
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-HIST-01 | Student meal history: each auto booking with its outcome (redeemed, skipped, expired, cancelled), paginated | A | V3, P2R |
| FR-HIST-02 | Student redemption history, including method and entry outcome | A | V3 |

## 13. RBAC, administration, membership — FR-RBAC, FR-ADMIN, FR-MEM
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-RBAC-01 | Baseline role model and permission matrix | A roles (D-043) / P matrix (P-002) | BL, ARCH |
| FR-RBAC-02 | Assign and revoke roles; audited | A intent / P authority | V2, Q-04 |
| FR-RBAC-03 | Register entrance terminals and rotate their credentials | P | P-009 |
| FR-ADMIN-01 | Edit slot config (windows, cutoffs, prices, `bookingEnabled`, `autoBookEnabled`, `optOutEnabled`) with DR-C1 validation | A | V3, D-017, D-038 |
| FR-ADMIN-02 | Edit menus of generated meal services (free-text items, Q-23) | A | V2 |
| FR-ADMIN-03 | Void a meal service (closure); its coupons become `VOID` | P | ARCH, Q-15 |
| FR-ADMIN-04 | Trigger an automatic-booking run manually and view run history with counts and errors | P | P-026 |
| FR-MEM-01 | Roster CSV/XLSX import (preview, validation, duplicate checks, explicit commit) creates/updates `Student` and `HostelMembership` | P | P-011, BL |
| FR-MEM-03 | Main Admin configures academic-year dates and hostel meal rules; memberships default to those dates | A | D-045 |
| FR-ADMIN-05 | Meal Admin reopens the opt-out or demo/paid booking window of one meal service with a reason; audited | A | D-046 |
| FR-MEM-02 | Main Admin can end, suspend or reinstate a membership, or record an allocation change; audited | P | P-029, D-042 |

## 14. Operations, audit, feedback — FR-OPS, FR-AUDIT, FR-FB
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-OPS-01 | Per meal service: planned, auto-booked (confirmed), opted out, redeemed and unredeemed counts, shown separately | A | P2, D-024 |
| FR-OPS-02 | Set and adjust planned quantity with a reason; audited; never automatic | A | D-024 |
| FR-OPS-03 | Live entrance panel: recent attempts, failure reasons, reconciliation queue size | P | ARCH |
| FR-OPS-04 | Automatic-booking health: last successful run, horizon covered, any service with missing coupons | P | D-035 |
| FR-AUDIT-01 | Append-only audit log for DR-A2 events | A | V3, D-041 |
| FR-AUDIT-02 | Filterable audit view (actor, action, entity, date) | P | ARCH |
| FR-FB-01 | Feedback (rating 1–5 + optional comment) for a meal service the student held a coupon for | A feature / P details | V2 |
| FR-FB-02 | Feedback review for authorised roles | A | V2 |
| FR-FB-03 | Identifiability notice before submission (Q-13) | A | V2 |

## 15. Deferred requirements (kept, not removed)
| ID | Requirement | St | Src |
|---|---|---|---|
| FR-KIT-01 | Inventory items, stock levels, receipts, issues to kitchen | D (F-001) | ES |
| FR-KIT-02 | Ration planning per meal service; waste recording and reporting | D (F-002) | V2, ES |
| FR-EVT-01 | Event organisers create events needing meals | D (F-003) | ES |
| FR-EVT-02 | Complimentary event coupons require authorised approval (approver Q-04) | D (F-004) | ES |
| FR-EVT-03 | Guest invitations; guests claim coupons (identity method undecided) | D (F-005) | V2, ES |
| FR-EVT-04 | Guest coupons redeem through the same redemption service with their own rules | D | V2 |
| FR-IO-01 | Advanced import/export (CSV/Excel) of bookings, redemptions, reports | D (F-006) | V2, ES |
| FR-NOT-01 | Notifications (opt-out cutoff reminders, meal changes) | D (F-007) | V2, ES |
| FR-RPT-01 | Additional reporting and admin dashboards; AI summaries | D (F-008) | V2, ES |
| FR-PAY-01 | Real payment gateway behind `PaymentProvider` | D (F-009) | V2 |
| FR-INT-01 | Real biometric and gate integrations behind existing interfaces | D (F-010, F-011) | V2 |
| FR-LAND-01 | Recruiter demo + landing page on isolated synthetic data; reset and expiry after 7 days | A (D-048) / U details (Q-16) | ES, BL |
| FR-PAID-01 | Paid diner coupons (including outsiders) with simulated payment | D (D-044) | BL |
| FR-PAID-02 | Refunds: Meal Admin executes after Hostel Director approval, hostel-management-caused only, audited | D (D-044) | BL |
| FR-ATT-01 | Kitchen-staff attendance with audited corrections | D (F-015) | BL |
