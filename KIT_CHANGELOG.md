# KIT_CHANGELOG — V2 → V3

Full audit of V2: `docs/KIT_V2_AUDIT.md`. Decision register: `docs/OPEN_DECISIONS.md`.

## 1. Summary of changes
| Area | V2 | V3 |
|---|---|---|
| Pilot model | Feedback demo; simulated tickets were not entitlements | Real controlled pilot; ₹0 prepaid hostel entitlements; simulation only in demo mode |
| Hosteller booking | Manual booking + simulated payment | **Automatic daily booking**, idempotent, with optional opt-out before the cutoff |
| QR | Static ticket QR shown by the student, scanned by staff | **Rotating QR displayed by the entrance terminal, scanned by the student's app** |
| Biometric | V2 future integration only | Simulator in V1 (dev/test/demo), off in the live pilot; V2 guardrails kept |
| Gate | Not in V1 | Gate simulator with reconciliation |
| Redemption | One-time, atomic | One `RedemptionService` for QR, staff-assisted and biometric; exactly-once by DB constraints + idempotency |
| Roles | Pending | Baseline roles (Main Admin, Meal Admin, Inventory Admin, Counter Staff, Staff Manager, Student/Diner, Guest, Recruiter, Hostel Director permission) + permission matrix |
| Data model | Candidate list | Full proposed schema, including Student/HostelMembership, constraints and generator SQL |
| API | None | Full contract with error envelope and codes |
| Meal times | Lunch 13:00–14:30 (pending), Dinner 19:00–21:30 | Lunch 13:00–15:30, Dinner 19:30–21:30, Snacks 16:30–17:30 (booking disabled) |
| Delivery | 3–4 day plan | Milestone roadmap M0–M10 |
| Claude Code | CLAUDE.md, 5 agents (model pinned), 4 skills, no settings | Shorter CLAUDE.md, `settings.json` (deny push/.env/destructive; ask installs/migrations/commits; attribution off; Bash + PowerShell rules), 8 agents (no pins), 6 skills, prompt templates |
| Learning | Brief mentoring note | Learning contract, walkthrough template, learning log, debugging guide, interview prep, TDR template |
| Developer guides | None | Windows setup + daily workflow (PowerShell) |
| Removed | `START_HERE` reference to ChatGPT; day-by-day plan; `docs/DECISIONS.md` (superseded by `OPEN_DECISIONS.md`); `PILOT_AND_FEEDBACK.md` merged into PRD/NFR/SECURITY (survey prompts below) | — |

## 2. Preserved requirements (from V2)
Product name and tagline; brand direction (now orange/white/deep charcoal per your baseline, D-049; no copying competitors); mobile-first accessible UI; no implied BMSCE endorsement; outside-college message; honest metrics; feedback with an identifiability notice; Google sign-in with server verification; domain and roster eligibility; domain never grants privilege; default deny; object-level authorisation; secure cookies, CSRF, CORS and rate limits; prices ₹45/₹70/₹70; breakfast window and all cutoffs; configurable rules; UTC storage with IST rules; server-authoritative state; no payment credentials ever; demo labels; idempotent duplicate protection; PII-free QR; manual lookup uses the same redemption service (→ staff-assisted); audit of staff/admin actions; no student refund or transfer; synthetic data; migrations only; quality gates; testing layers; security baseline; backups with tested restore; all V2 guardrails (no biometric data, entry ≠ meal, no gate control without a safety review, no scraping, non-biometric alternative, Razorpay rules for later).

**Feedback-survey prompts kept from V2 `PILOT_AND_FEEDBACK.md`:** Was it clear how to find the meal? Could you enter without help? Was the coupon easy to understand? What confused you or failed? Which feature would make this useful daily? Would you use an approved version, and why or why not?

## 3. New requirements (V3, Phase Two and its revision)
Real controlled pilot (~500 users); ₹0 `HOSTEL_PREPAID` funding; automatic daily booking with retry safety; membership model (join, leave, suspend, allocation change); opt-out mechanism (disabled until policy); biometric simulator with four outcomes; terminal-displayed rotating QR challenge (hashed, short-lived, single-use); convergence on one redemption service; entry-time safety net; gate-controller simulator and reconciliation; staff-assisted recovery; planned quantity never automatic and audited; snacks configurable but disabled; baseline role model; essential ops dashboards; learning contract; Git ownership rules; V1.1/V1.2 modules kept (kitchen, inventory, waste, events, complimentary and guest coupons, import/export, notifications, reporting); recruiter landing page (phase undecided).

## 3b. Reconciled with your repository's `approved-baseline.md`
Roles adopted from the baseline (D-043); academic-year entitlement (D-045); window reopen (D-046); unlimited capacity (D-047); recruiter demo (D-048); brand (D-049); paid diners and refunds kept but deferred to V1.1 (D-044); staff attendance deferred (F-015). Full table: `docs/OPEN_DECISIONS.md` §0b.

## 4. Unresolved decisions (safe defaults in use)
Q-01 college email domain · Q-02 roster source and columns · Q-04 role-assignment authority · Q-06 opt-out policy · Q-07 horizon (default 2 days) · Q-08 entry grace (none) · Q-09 gate retry limits · Q-10 staff identity check · Q-12 initial planned quantity · Q-13 feedback anonymity · Q-14 price-change governance · Q-15 mess cancellations · Q-16 landing-page phase · Q-17 hosting · Q-18 data/privacy and authorisation · Q-19 snacks rules · Q-20 organiser role · Q-28 guest QR model (V1.1) · Q-22 terminal count · Q-23 menu detail · Q-24 late joiners · Q-25 suspension · Q-26 exit-date semantics · Q-27 closure calendar.
