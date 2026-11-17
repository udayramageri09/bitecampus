# OPEN_DECISIONS — BiteCampus Decision Register

This file owns the **status** of every product and engineering decision. Other documents cite decision IDs; they never restate a status differently. When a decision changes, update it here first, then update the affected documents in the same change.

**Last updated:** 2026-11-17 (Phase Two, revision 2 — automatic hosteller booking)
**Supersedes:** `docs/DECISIONS.md` from the V2 kit.

## Legend

**Status**
| Status | Meaning |
|---|---|
| APPROVED | The owner explicitly approved it. Implement it as written. |
| PROPOSED | A recommendation by Claude, or an earlier idea not yet confirmed. Can be designed around, but must be validated before the dependent feature ships. |
| DEFERRED | Intentionally out of the initial pilot. Requirements are kept; the release is V1.1/V1.2 or V2. |
| UNRESOLVED | A business rule that is missing. **Do not invent a value.** The dependent behaviour must fail safe until it is decided. |
| RESOLVED | A former question, now answered by the decision named. |

**Source**
| Tag | Origin |
|---|---|
| V2 | Uploaded `bitecampus-claude-starter-v2.zip` |
| ES | Earlier BiteCampus specifications reported by the owner in the V3 kickoff prompt. **Not present in the V2 kit.** |
| V3 | V3 kickoff prompt (this conversation, 2026-11-17) |
| P2 | Phase Two decisions by the owner (2026-11-17, first message) |
| P2R | Phase Two revision by the owner — automatic hosteller booking (2026-11-17, second message) |
| BL | The owner's repository file `docs/requirements/approved-baseline.md` (seen 2026-11-17). Where it conflicts with P2/P2R, the **later explicit decision (P2/P2R) wins**, per the repository's own `AGENTS.md` rule; reconciliation in §0b |
| ARCH | Engineering proposal by Claude |

---

## 0. Working assumptions for building (adopted 2026-11-17)
The owner asked to *"start building; wherever required make a wiser assumption; keep the process simpler and user friendly."* Therefore:

1. Every **PROPOSED** item (§2) is a **working default** for implementation. Build it as written, and revisit it if the owner or the college says otherwise.
2. Every **UNRESOLVED** item (§4) uses its **safe default** during implementation. The question stays listed because it needs an institutional answer before the real pilot, not before coding.
3. Items that need the **college** before going live: Q-01 domain, Q-02 roster source and columns (College IT, BL), Q-06 opt-out policy, Q-10 staff identity check, Q-18 data/privacy, Q-25 suspension meaning. Everything else can be changed by the owner at any time.
4. Process simplification: plans are one screen; teaching happens after each milestone, not after each file (CLAUDE.md §5).

## 0b. Reconciliation with `docs/requirements/approved-baseline.md` (BL)
| Baseline item | Outcome | Decision |
|---|---|---|
| Roles: Main Admin, Meal Admin, Inventory Admin, Counter Staff, Staff Manager, Student/Diner, Guest, Recruiter, Hostel Director permission | **Adopted** as the role model; replaces the proposed nine roles | D-043 |
| Hostel residents don't book or pay per meal; access appears in their account | **Compatible** with automatic daily booking | D-034 |
| Main Admin configures academic-year dates and hostel meal rules; entitlement renews yearly with roster updates | **Adopted**: memberships carry academic-year `validFrom`/`validTo`; Main Admin owns roster import | D-045 |
| Ordinary diners, including outsiders, buy meal coupons (simulated payment) | **Kept, deferred to V1.1** — P2 excluded paid categories from the initial hostel pilot. Built on demo-mode simulated checkout | D-044 |
| Refunds by Meal Admin after Hostel Director approval, hostel-management-caused only, audited | **Kept, deferred to V1.1** (only paid bookings can be refunded) | D-044 |
| Meal Admin may reopen a booking window (scoped, reasoned, audited) | **Adopted** for the opt-out window and demo/paid bookings; audited | D-046 |
| Unlimited booking capacity in V1 | **Adopted** (no capacity checks) | D-047 |
| Price snapshot on each booking | **Compatible** (P-019) | — |
| Counter Staff "scans meal passes"; Guests "present the assigned QR at the counter" | **Superseded for hostellers** by D-019 (terminal displays, student scans). Counter Staff run staff-assisted recovery. **Guest QR flow (V1.1) remains as BL describes** and is revisited then (Q-28) | D-019, D-040 |
| Recruiter demo: isolated synthetic data, no real email or payments, reset and access expiry after 7 days | **Adopted** for the demo environment; resolves most of Q-16 | D-048 |
| Brand: orange, white, deep charcoal; original food mark; system font | **Adopted** (newer than the V2 purple/orange) | D-049 |
| Staff Manager records kitchen-staff attendance; no biometrics/location/payroll | **Kept, deferred to V1.1** | F-015 |
| Inventory movement ledger, advisory forecasting, no auto-purchasing | **Compatible** with F-001/F-002 | — |
| Reports separate booked/paid/cancelled/prepaid/redeemed/guest/inventory/waste/attendance; restricted, audited exports | **Compatible**; pilot ops dashboard covers the prepaid part | — |
| CSV/XLSX imports with preview, validation, duplicate checks, explicit commit | **Adopted** for the roster import (dry-run = preview) | P-011 |
| Email as the first notification channel; no SMS | **Adopted** for V1.1 notifications | F-007 |
| Encrypted backups, restore drill, staging/production separation; RPO 1 h / RTO 4 h suggested | **Adopted** as targets pending college validation | NFR-15 |
| Current milestone "frontend-only demo with synthetic data" | **Superseded** by the roadmap (`docs/ROADMAP.md`); the frontend shell is finished first in M1 | — |

## 1. Approved decisions

| ID | Decision | Source |
|---|---|---|
| D-001 | Product **BiteCampus**; tagline "Your campus. Your food. Your way." | V2, V3 |
| D-002 | Initial target: one BMSCE hostel mess, approximately 500 users, as a real controlled college pilot | V3 |
| D-003 | Modular monolith; one deployable backend, one PostgreSQL database | V2, V3 |
| D-004 | Preferred stack: React + TypeScript + Vite + Tailwind; Node.js + Express + TypeScript; PostgreSQL; Prisma; Zod; Vitest, Supertest, Playwright. **Subject to the read-only repository audit. Do not replace or upgrade existing dependencies without approval.** | V2, V3 |
| D-005 | Google sign-in; ID token verified on the server (signature, issuer, audience, expiry, `email_verified`); college email-domain validation | V2, V3 |
| D-006 | Active student/hostel roster validation is required for student access (roster source is Q-02) | V3 |
| D-007 | Domain membership never grants a staff or admin role; privileged roles are assigned explicitly | V2 |
| D-008 | Hosteller meals are covered by prepaid hostel mess fees. Hosteller bookings carry a **₹0** amount and funding source `HOSTEL_PREPAID`, which is distinct from any online payment. **No simulated payment is involved in hosteller bookings** | P2, P2R |
| D-009 | Simulated payment **never** creates a real pilot entitlement. Simulated checkout stays available for demonstration and for future payment-enabled user categories | P2 |
| D-010 | No day scholars or other paid customer categories in the initial pilot without a separate approved requirement | P2 |
| D-011 | Real online payment gateway deferred until college authorisation and business documentation exist | V2, P2 |
| D-012 | List prices: Breakfast ₹45, Lunch ₹70, Dinner ₹70 (shown for information; hostellers pay ₹0) | V2, V3 |
| D-013 | Service windows: Breakfast 07:00–09:30; Lunch 13:00–15:30; Snacks 16:30–17:30; Dinner 19:30–21:30 | V3, P2 (supersedes V2 lunch 13:00–14:30 and dinner 19:00–21:30) |
| D-014 | Meal cutoffs: Breakfast — previous day 19:00; Lunch and Dinner — same day 10:00. Under automatic booking, the cutoff is the **opt-out cutoff** (and the demo paid-booking cutoff) | V2, V3, P2R |
| D-015 | Snacks exist as a configurable meal category, but booking (automatic or demo) is **disabled** until price, cutoff and operational rules are approved (Q-19) | P2 |
| D-016 | Store timestamps in UTC; evaluate business rules in `Asia/Kolkata`; the meal date is a campus-local date | V2, P2, P2R |
| D-017 | Prices, windows, cutoffs, automatic-booking and opt-out enablement and similar rules are configuration, not constants in code | V2, V3, P2R |
| D-018 | Biometric verification is a **simulator** in V1, behind a replaceable interface. Simulated outcomes: success, failure, device unavailable, timeout. No biometric image, template or sample is collected or stored | V3 |
| D-019 | QR fallback direction: a **rotating QR is displayed by the mess-entrance terminal**, and the student scans it with their own phone inside BiteCampus. Students never display a QR for staff to scan | P2, P2R |
| D-020 | The entrance QR challenge is separate from the meal coupon. It is high-entropy, short-lived, single-use, stored as a hash, server-verified, replay-protected and contains no personal or coupon data | P2 |
| D-021 | Biometric and QR (and staff-assisted recovery, D-040) are alternative identity-verification methods that converge on **one** server-side entitlement and redemption service. Neither identity success nor QR validation alone authorises a meal | V3, P2, P2R |
| D-022 | "Exactly once" means exactly one durable redemption effect per coupon, guaranteed by database transactions, constraints and idempotency. It is tested with concurrent requests against real PostgreSQL | V3 |
| D-023 | The gate controller is a **simulator** behind its own integration boundary. BiteCampus does not control real gates in V1. A controller timeout after a committed redemption is handled by reconciliation, never by redeeming again | V2, V3, P2 |
| D-024 | Bookings and opt-outs never automatically change the kitchen's planned food quantity. Planned, booked and attended quantities are kept separate. Authorised kitchen/admin staff may adjust planned quantity manually, and every adjustment is audited | P2, P2R |
| D-025 | Initial pilot scope = the 17 items in PRD §4.1, with items 4–5 reinterpreted as **automatic daily meal booking** and **₹0 prepaid hostel entitlement** | P2, P2R |
| D-026 | V1.1/V1.2 scope = the modules in PRD §4.2. They are deferred, not removed | P2 |
| D-027 | Learning contract: explain → plan → **owner approval** → implement in small milestones → walkthrough, tests, interview questions, teach-back and learning log | V3 |
| D-028 | Git ownership: no identity change, no Claude/Anthropic co-author trailers, no commit or push without approval, no force-push, history rewrite, or branch/tag deletion | V3 |
| D-029 | V2 guardrails stay in force: no biometric datasets in BiteCampus; campus entry ≠ meal redemption; no scraping of college systems; no gate control without a safety review; non-biometric alternative available | V2 |
| D-030 | Synthetic data only in demo/test environments; no claim of BMSCE endorsement without written authorisation | V2 |
| D-031 | Outside-college sign-in message: "Outside-college access is planned for a future version." | V2 |
| D-032 | Metrics are reported only after measurement; targets are labelled as targets | V2 |
| D-033 | The QR payload never carries student or coupon data. Scanning a QR alone never authorises entry: an authenticated student session and a valid server-side entitlement are both mandatory | P2 |
| D-034 | **Automatic daily booking.** Hostellers do not book manually. The system automatically creates a booking, entitlement and coupon for every active, eligible hosteller for each applicable meal (Breakfast, Lunch, Dinner) every day. Resolves Q-05 | P2R |
| D-035 | Automatic booking is **idempotent**: database constraints prevent duplicates, and it is safe when retried or executed more than once. Correctness does not depend on a scheduled job running exactly once | P2R |
| D-036 | Separate concepts: active hostel membership · automatic daily booking · meal entitlement · digital coupon · identity verification · coupon redemption · physical entry response | P2R |
| D-037 | Active hostel membership alone never authorises entry; a valid meal-specific entitlement is required | P2R |
| D-038 | **Opt-out mechanism**: a hosteller may skip one specific meal **before its cutoff**, if the policy is enabled. Opt-out invalidates only that meal's entitlement and coupon and never reduces planned quantity. The mechanism is built; whether and how it is enabled is pending college confirmation (Q-06) | P2R |
| D-039 | The biometric simulator is clearly labelled as a simulator. It is used for development, testing, training and controlled demonstrations only. It is never presented as real identity verification and is **disabled in the live pilot environment**. Resolves Q-11; supersedes P-022 | P2R |
| D-040 | The live pilot requires (a) a working entrance-QR verification flow and (b) an authorised **staff-assisted recovery** process through the same redemption service. Real biometric integration remains subject to institutional approval. Promotes P-012 | P2R |
| D-041 | Automatic booking maintains an audit trail | P2R |
| D-042 | Automatic booking handles students joining, leaving, changing hostel allocation, and inactive or suspended accounts | P2R |
| D-043 | Role model from the baseline: Student/Diner, Main Admin, Hostel Director (permission), Meal Admin, Inventory Admin, Counter Staff, Staff Manager, Guest, Recruiter (`USER_ROLES_AND_PERMISSIONS.md`). Supersedes P-001/P-003; resolves Q-03, Q-21 | BL |
| D-044 | Paid diner coupons (including outsiders), with simulated payment, and refunds (Meal Admin after Hostel Director approval, hostel-management-caused only, audited) are **kept as requirements and deferred to V1.1**. They stay separate from prepaid hostel access and guest coupons in authorisation, persistence, reporting and audit | BL, P2 |
| D-045 | Hostel entitlement follows the academic year: Main Admin configures academic-year dates and hostel meal rules; memberships use those dates; entitlement renews with the yearly roster update. No invented meal quota | BL |
| D-046 | Meal Admin may reopen a specific window (opt-out window, demo/paid booking window); every reopen is scoped to one meal service, needs a reason and is audited | BL |
| D-047 | Booking capacity is unlimited in V1 | BL |
| D-048 | Recruiter demo: isolated demo environment, synthetic data, no real emails or payments, no destructive admin actions, data reset and access expiry after 7 days. Approval flow and exposed screens are decided during demo setup (Q-16) | BL |
| D-049 | Brand: orange, white and deep charcoal; original simple food mark; system font; responsive, accessible, reduced motion | BL |

## 2. Proposed — pending validation

| ID | Proposal | Source | Blocks |
|---|---|---|---|
| P-001 | ~~Nine-role model~~ — **superseded by D-043** (baseline roles) | ES | — |
| P-002 | Permission-based RBAC: code checks permissions; roles are bundles of permissions (matrix in USER_ROLES_AND_PERMISSIONS §3) | ARCH | RBAC |
| P-003 | ~~"Hostel" role = hostel office~~ — **superseded by D-043**: Main Admin owns roster and memberships | ARCH | — |
| P-004 | Session: opaque session ID in an HTTP-only, Secure, SameSite=Lax cookie; session rows stored in PostgreSQL; CSRF protection on state-changing routes | ARCH | Auth |
| P-005 | **One booking row per (student, meal service), ever**: a full unique constraint, not a partial one. A student who opted out cannot be re-booked by a generator re-run | ARCH | D-035, D-038 |
| P-006 | A coupon can be redeemed only for its own meal service and only within that service's configured window (early/late grace is Q-08) | ARCH, derived from V2 "date/meal valid" | Redemption |
| P-007 | QR challenge time-to-live 30 s; the terminal rotates to a new challenge every 15 s and immediately after a consumption; values configurable | ARCH | QR |
| P-008 | A QR challenge is consumed only by a successful redemption; failed attempts are recorded and rate-limited but do not consume it | ARCH | QR |
| P-009 | Entrance terminals authenticate as **device principals** with a per-terminal credential and HMAC-signed requests (timestamp + nonce) | ARCH | Terminal |
| P-010 | The `MealCoupon` record **is** the entitlement (one table); the "digital coupon" is its presentation in the app | ARCH | Data model |
| P-011 | Roster CSV/XLSX import (by Main Admin, with preview, validation, duplicate checks and explicit commit — BL) creates and updates `Student` and `HostelMembership` records; dry-run first. Required in the pilot because automatic booking depends on membership | ARCH | Q-02 |
| P-012 | Staff-assisted recovery details: Counter Staff search the student, confirm identity in person, choose a reason code; audited. Mechanism approved by D-040; the identity check and reason codes are Q-10 | V2, ARCH | Q-10 |
| P-013 | Demo mode: simulated checkout is enabled only when `DEMO_MODE=true`. Demo bookings belong to **demo (non-hosteller) students**, carry `fundingSource = SIMULATED_PAYMENT` and `isDemo = true`, and pilot checkpoints reject them | ARCH | D-009 |
| P-014 | REST JSON API under `/api/v1` with a single error envelope | V2, ARCH | API |
| P-015 | `Idempotency-Key` header required on opt-out, QR redemption, demo booking/checkout, staff-assisted redemption and gate retry; records kept for 24 h | ARCH | API |
| P-016 | Gate-command timeout 3 s; automatic status query, then a reconciliation queue; values configurable | ARCH | Q-09 |
| P-017 | The first Main Admin account is bootstrapped by a one-off server CLI script, never through the UI | ARCH | Q-04 |
| P-018 | An unredeemed coupon becomes `EXPIRED` once its service window (plus any approved grace) has ended | ARCH | Coupon |
| P-019 | A booking stores a **snapshot** of the list price and the amount payable (₹0 for hostellers) | ARCH | Q-14 |
| P-020 | Performance targets (unmeasured): p95 < 300 ms for ordinary API calls; p95 < 1 s for redemption end-to-end excluding the gate; 50–100 concurrent synthetic requests; automatic generation for 500 students × 3 meals × horizon completes in under 60 s | V2, ARCH | NFR |
| P-021 | Accessibility target: WCAG 2.1 AA for student and staff UIs | ARCH | NFR |
| P-023 | **Student record separate from login.** `Student` (from the roster) is the anchor for membership, bookings, coupons and redemptions. A Google `User` links to it on first sign-in. Hostellers who never open the app still receive coupons | ARCH | D-034 |
| P-024 | Meal services for each applicable slot and date are **generated automatically** from slot configuration over the horizon; Meal Admin adds menus and voids closures | ARCH | D-034 |
| P-025 | Generation horizon: from today to today + 2 campus days (configurable). This makes Breakfast for D+1 exist before its D 19:00 cutoff. The value is pending owner validation (Q-07) | ARCH | Q-07 |
| P-026 | Generator triggers: scheduled every hour, on server start, and by a manual admin trigger. Each run covers the **whole horizon** (self-healing catch-up). A PostgreSQL advisory lock avoids wasted parallel work; correctness relies on constraints, not the lock | ARCH | D-035 |
| P-027 | Membership-end reconciliation: when a membership ends, unredeemed coupons for services after the end date are voided (`MEMBERSHIP_ENDED`). If a corrected or new membership recorded before that service's cutoff makes the student eligible again, the generator revives them | ARCH | D-042 |
| P-028 | **Entry-time safety net**: if an eligible hosteller has no booking row for the current service (for example, the job failed) and the Q-24 generation limit is met, the redemption transaction creates it with the same eligibility function as the generator. It never creates one over an opt-out or void | ARCH | D-035 |
| P-029 | `HostelMembership(studentId, messId, status, validFrom, validTo, recordedAt)` is the source of hosteller eligibility; `recordedAt` is immutable and drives the generation limit; allocation change = end one membership and start another | ARCH | D-042 |
| P-030 | Audit for generation: one audit event per run (counts per meal service, created/revived/voided); each booking stores `generationRunId`; per-row audit for opt-out, void and revive | ARCH | D-041 |
| P-031 | Suspended or inactive students are excluded from new generation, and their existing coupons are **blocked at entry** (`STUDENT_NOT_ACTIVE`) rather than voided, so reinstatement needs no repair. Semantics pending Q-25 | ARCH | Q-25 |

## 3. Deferred

| ID | Item | Target | Source |
|---|---|---|---|
| F-001 | Kitchen and inventory management | V1.1/V1.2 | ES, P2 |
| F-002 | Advanced ration and waste reporting | V1.1/V1.2 | V2, ES, P2 |
| F-003 | Event organiser workflows | V1.1/V1.2 | ES, P2 |
| F-004 | Complimentary guest coupon approval and distribution | V1.1/V1.2 | ES, P2 |
| F-005 | External guest coupon claiming | V1.1/V1.2 | V2 (guest passes), ES, P2 |
| F-006 | Advanced import/export | V1.1/V1.2 | V2, ES, P2 |
| F-007 | Extended notification workflows | V1.1/V1.2 | V2, ES, P2 |
| F-008 | Additional reporting and admin capabilities (including AI summaries) | V1.1/V1.2 | V2, ES, P2 |
| F-009 | Real payment gateway (e.g. Razorpay) | V2 | V2, P2 |
| F-010 | Real biometric integration with the college/vendor system | V2 | V2, P2R |
| F-011 | Real physical gate integration | V2 | V2 |
| F-012 | Refunds and replacement coupons after a college cancellation | Later | V2 |
| F-013 | Day scholars and other paid user categories, including the BL paid-diner flow and refunds (D-044) | V1.1 | P2, BL |
| F-014 | Multi-campus support | Later | V2 |
| F-015 | Kitchen-staff attendance (Staff Manager) with audited corrections; no biometrics, location or payroll | V1.1 | BL |

## 4. Unresolved — the owner must decide

| ID | Question | Why it matters | Safe default until decided |
|---|---|---|---|
| Q-01 | Exact allowed college email domain(s) | Sign-in eligibility | Configured `ALLOWED_EMAIL_DOMAINS`; sign-in denied if empty |
| Q-02 | Roster source, file format, refresh cadence, data owner, college authorisation | Membership, therefore automatic booking | Synthetic roster only; no real data |
| Q-04 | Who may assign each role (e.g. can a Main Admin create another Main Admin?); who approves complimentary event coupons (V1.1) | Privilege escalation | Main Admin assigns all roles (USER_ROLES_AND_PERMISSIONS §3); first Main Admin via P-017 |
| Q-06 | **Opt-out policy**: is opt-out allowed at all? For which meals? Can an opt-out be undone before the cutoff? Limits per week? Multi-day absence (leave) opt-outs? | D-038 enablement | Mechanism built; `optOutEnabled = false` for every slot; no undo; no multi-day opt-out |
| Q-07 | **Generation horizon**: how many days ahead entitlements exist (which also sets how early a student can opt out) | Visibility, opt-out window | P-025: today .. today + 2 |
| Q-08 | Early/late entry grace around service windows | Redemption | No grace (exact window) |
| Q-09 | After a gate timeout, how long may the student retry entry on the same redemption, how many times, and who may override? | Reconciliation | Retry allowed only during the service window; Counter Staff retry; Meal Admin / Main Admin resolve the queue |
| Q-10 | Staff-assisted recovery: what identity check staff must perform (e.g. college ID card), which reason codes, and whether a second staff member must confirm | D-040 integrity | Requires `entry.assist`; mandatory reason code from a short configurable list; audited; single staff member |
| Q-12 | Who sets the initial planned food quantity for each meal service, and from what baseline? | Kitchen planning | Set manually by Inventory Admin / Meal Admin; no computed default |
| Q-13 | Is feedback identifiable or anonymous? Moderation/visibility rules? | Privacy | Identifiable to Meal Admin, Inventory Admin and Main Admin only, with notice shown before submission |
| Q-14 | Who approves price changes, and when do they take effect? | Pricing governance | Meal Admin edits future meal services only; audited (P-019 snapshot) |
| Q-15 | Meal cancelled by the mess (closure, holiday): coupon handling and student communication | Coupon lifecycle | Meal Admin voids the meal service; its coupons become `VOID` (`SERVICE_VOIDED`); in-app status only |
| Q-16 | Recruiter demo details (BL open items): do requests need manual approval, and which screens does it expose? Release phase of the public landing page | Scope | Demo environment per D-048; landing page built with M9 |
| Q-17 | Hosting provider, budget, domain name, scheduled-job mechanism available on that host | Deployment, generator triggers | None chosen; generator also runs on server start and by manual trigger |
| Q-18 | Data retention, privacy notice text, data controller, written college authorisation for real data, whether a first name may be shown on the terminal | Compliance | Synthetic data only; no name on terminal |
| Q-19 | Snacks price, cutoff and operational rules; whether Snacks are auto-booked once enabled | Snacks | Booking disabled (D-015) |
| Q-20 | Are "Event Organiser" and "Guest" roles in addition to the nine? (V1.1) | V1.1 RBAC | Not implemented in pilot |
| Q-22 | Number of entrance terminals/checkpoints in the pilot | Terminal provisioning | Design supports N; seed one |
| Q-23 | Menu detail per meal service (items, dietary labels) | Menu admin | Free-text item list only; no invented attributes |
| Q-24 | **Late joiners**: if a student's membership is recorded after a meal's cutoff, do they get that meal? (Other options: until window start, or until window end) | Students joining mid-day | No. An entitlement is created only if the membership was recorded before the service's cutoff (DR-AB3); the late joiner is denied at entry for that meal |
| Q-25 | **Suspension semantics**: what suspends a hosteller (Main Admin action, fee default, disciplinary)? Does suspension stop meals immediately? | D-042 | P-031: excluded from generation; existing coupons blocked at entry while suspended |
| Q-26 | Exit date semantics: is a membership's `validTo` date inclusive (the student eats all meals that day)? | Leaving students | Inclusive |
| Q-28 | Guest coupon QR (V1.1): BL says guests present a QR at the counter, and the email may embed the QR or link to it. Keep that model for guests, or move guests to the entrance-QR model too? | V1.1 guest design | Decide when V1.1 starts |
| Q-27 | Mess closure periods (vacations, exam breaks): bulk voiding by date range, or a calendar of non-operating days? | Generation | Meal Admin voids individual services; no calendar |

## 5. Resolved questions

| ID | Question | Resolved by |
|---|---|---|
| Q-05 | Can a hosteller without a manual booking eat? | D-034 — hostellers are automatically booked; there is no manual booking |
| Q-11 | How is the biometric simulator used in the live pilot? | D-039 — it isn't; disabled in the pilot environment |
| Q-03 | Meaning of the "Hostel" role | D-043 — Main Admin owns roster and memberships |
| Q-21 | Is "Staff" a distinct role? | D-043 — no; baseline roles only |

## 6. Superseded values (kept for traceability)

| Value | Old | New | Decided by |
|---|---|---|---|
| Lunch window | 13:00–14:30 (V2, pending) | 13:00–15:30 | P2 / D-013 |
| Dinner window | 19:00–21:30 (V2) | 19:30–21:30 | P2 / D-013 |
| Snacks | Deferred (V2) | Configurable, booking disabled | P2 / D-015 |
| QR model | Static opaque ticket QR shown by the student and scanned by staff (V2 V1); dynamic QR in V2 | Rotating QR displayed by the entrance terminal and scanned by the student | P2 / D-019 |
| Pilot nature | Feedback demo; simulated tickets not entitlements (V2) | Real controlled pilot; ₹0 prepaid hostel entitlements; simulation is demo-only | P2 / D-008, D-009 |
| Hosteller booking | Manual booking with ₹0 confirmation (Phase Two, revision 1) | **Automatic daily booking**; optional opt-out before cutoff | P2R / D-034, D-038 |
| Cutoff meaning | Booking cutoff | Opt-out cutoff (and demo paid-booking cutoff) | P2R / D-014 |
| Q-06 (old) | Student cancellation or modification of a manual booking | Replaced by opt-out policy | P2R |
| Q-07 (old) | Manual booking horizon | Replaced by generation horizon | P2R |
| P-022 | Biometric simulator disabled in pilot (proposal) | Approved as D-039 | P2R |
| Delivery plan | 3–4 day release (V2) | Milestone roadmap after the repository audit | V3 |
