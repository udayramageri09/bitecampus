# USER_ROLES_AND_PERMISSIONS

> **Status:** role names and responsibilities follow your repository's `docs/requirements/approved-baseline.md` (source **BL**, D-043). They replace the earlier proposed nine-role model (P-001, now superseded). The permission matrix below is a working default (P-002) and may be adjusted.

## 1. Principles (APPROVED unless marked)
1. Default deny (V2).
2. Code checks **permissions**, not role names (P-002). Roles are named bundles of permissions stored in the database.
3. A user may hold several roles; effective permissions are the union. For example, one person may be both Main Admin and Hostel Director (BL).
4. Students can only see their own records (V2).
5. Domain, roster or membership grants at most the Student role (D-007). Staff/admin accounts are provisioned by authorised administrators (BL). Client-side role or category values never authorise anything (BL).
6. Every role grant or revoke is audited (DR-A2).
7. Entrance terminals are **device principals**, not users or roles (P-009). Fixed permissions: `terminal.challenge.issue`, `terminal.biometric.submit`, `terminal.result.read`.

## 2. Roles
| Role key | Name | Responsibilities (BL, adapted to Phase Two) | Pilot |
|---|---|---|---|
| `STUDENT` | College Student / Diner | Sees meals and their automatic coupons, scans the entrance QR, opts out if enabled, views history, gives feedback | ✔ |
| `MAIN_ADMIN` | Main Admin | Cross-department oversight; configures academic-year dates and hostel meal rules; owns roster and membership imports; provisions staff/admin roles; cross-department reports | ✔ |
| `HOSTEL_DIRECTOR` | Hostel Director (permission) | Approves eligible refunds (V1.1, paid flow); may be combined with Main Admin | V1.1 |
| `MEAL_ADMIN` | Meal Admin | Meal prices, windows and cutoffs, reopening a window (scoped, reasoned, audited), menus, voiding services, booking management, reconciliation queue, guest-coupon operations (V1.1), refund execution after Hostel Director approval (V1.1) | ✔ |
| `INVENTORY_ADMIN` | Inventory Admin | Preparation planning (planned quantity, D-024); stock, usage, waste, adjustments and inventory reports (V1.1) | ✔ (planning) |
| `COUNTER_STAFF` | Counter Staff | Entrance operations: watches entry outcomes, staff-assisted recovery (D-040), gate retries | ✔ |
| `STAFF_MANAGER` | Staff Manager | Records kitchen-staff attendance and audited corrections (V1.1). No payroll, biometrics or location tracking (BL) | V1.1 |
| `GUEST` | Guest | Event-scoped invited coupons only; cannot book ordinary meals (BL) | V1.1 |
| `RECRUITER` | Recruiter | Interactive demo on isolated synthetic data; demo environment only; access expires after 7 days (BL) | demo env |

Roles from the earlier proposal map as follows: "Director" → Main Admin (+ Hostel Director for refunds); "Hostel" (office) → Main Admin; "Kitchen" → Inventory Admin (planning); "Staff" (generic) → not a separate role (Counter Staff or Inventory Admin as appropriate). This resolves Q-03 and Q-21.

## 3. Permission matrix (pilot)
✔ = granted. "own" = only the caller's records.

| Permission | STU | MAIN | MEAL | INV | CTR |
|---|---|---|---|---|---|
| `meal.read` | ✔ | ✔ | ✔ | ✔ | ✔ |
| `booking.read.own` / `coupon.read.own` / `redemption.read.own` | ✔ | | | | |
| `booking.optout.own` (only when opt-out is enabled, Q-06) | ✔ | | | | |
| `entry.qr.redeem.self` | ✔ | | | | |
| `feedback.create.own` | ✔ | | | | |
| `academic_year.manage` (academic-year dates, hostel meal rules) | | ✔ | | | |
| `roster.import` / `roster.manage` (students, memberships) | | ✔ | | | |
| `autobooking.run` (manual trigger; idempotent) | | ✔ | ✔ | | |
| `mealslot.config.manage` (windows, cutoffs, prices, booking/auto-book/opt-out flags) | | | ✔ | | |
| `mealservice.manage` (menu, void, reopen window) | | | ✔ | | |
| `kitchen.plan.adjust` | | | ✔ | ✔ | |
| `ops.dashboard.read` | | ✔ | ✔ | ✔ | ✔ |
| `entry.assist` (staff-assisted recovery, D-040) | | | ✔ | | ✔ |
| `gate.retry` (same redemption) | | | ✔ | | ✔ |
| `reconciliation.resolve` | | ✔ | ✔ | | |
| `booking.read.any` (support lookup) | | ✔ | ✔ | | ✔ |
| `feedback.read` | | ✔ | ✔ | ✔ | |
| `audit.read` | | ✔ | | | |
| `role.assign.staff` (MEAL, INV, CTR, STAFF_MANAGER) | | ✔ | | | |
| `role.assign.privileged` (MAIN_ADMIN, HOSTEL_DIRECTOR) | | ✔ | | | |
| `terminal.manage` (register, rotate credential) | | ✔ | | | |
| `simulator.control` (demo/test only) | | ✔ | ✔ | | |

Main Admin assigning other Main Admins is a working default (Q-04). The first Main Admin is bootstrapped by a CLI script (P-017).

## 4. Deferred permissions (V1.1/V1.2)
`inventory.*`, `waste.record` (Inventory Admin) · `attendance.record`, `attendance.correct` (Staff Manager) · `event.create`, `comp_coupon.request` (Organiser — whether this is a role is Q-20) · `comp_coupon.approve` (authorised approver, BL; who exactly is Q-04) · `guest_coupon.issue` (Meal Admin) · `guest_coupon.read.own` (Guest) · `refund.approve` (Hostel Director) · `refund.execute` (Meal Admin) · `paid_booking.create.own` (paid diner, D-044) · `export.*` (restricted and audited, BL) · `notification.manage` · `report.cross_department` (Main Admin).

## 5. Enforcement
- Middleware `requirePermission('x.y')` on every protected route. Object-level checks (`own`) happen in the service, not the route.
- Permission changes take effect on the next request; nothing about roles is cached across requests.
- Tests: every protected route has at least one 401 test (no session), one 403 test (wrong permission) and one object-level test where `own` applies.
