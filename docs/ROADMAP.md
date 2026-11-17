# ROADMAP — build order

Replaces the V2 "3–4 day" delivery plan. Each milestone is a small vertical slice: plan → approval → build → verify → teach back (CLAUDE.md §5). The repository starts with documentation only (fresh start, 2026-11-17).

| # | Milestone | Delivers | Proves (AC IDs) | Learn |
|---|---|---|---|---|
| M0 | Orientation (read-only) | Claude reads the docs, checks your tools, and proposes the M1 plan | — | How the docs fit together |
| M1 | Foundation | npm workspaces monorepo: `apps/web` (React + TS + Vite + Tailwind, brand tokens D-049, app shell) and `apps/api` (Express + TS); `.env.example`; env validation (Zod); health/ready; error envelope; request-ID logging; Prisma connected; test DB + Vitest/Supertest harness; CI-ready scripts | health tests | Express request lifecycle, env config, test setup |
| M2 | Sign-in & sessions | Google token verification; `Student` link; DB sessions + CSRF; `/me`; logout; role/permission tables; `requirePermission`; first-Main-Admin CLI | AC-AUTH-*, AC-RBAC-01 (partial) | OAuth ID tokens, cookies, middleware |
| M3 | Residency | Academic year (D-045); `Student`, `Mess`, `HostelMembership` (+ exclusion constraint); roster CSV/XLSX import with preview; membership actions; audit | AC-MEM-01 | Constraints, transactions, CSV validation |
| M4 | Catalog & automatic booking | Slot config; meal-service generation; **idempotent generator** (schedule + startup + manual); reconciliation; JobRun + audit | AC-AB-01…11, AC-AB-13, AC-ADMIN-01/02 | `ON CONFLICT`, idempotency, time zones |
| M5 | Student app | Dashboard, coupons (no QR), history, opt-out mechanism (disabled by default) | AC-COUP-01, AC-HIST-01, AC-OO-* | React data fetching, UI states |
| M6 | Entrance QR + redemption + gate | Terminal device auth (HMAC); rotating challenge; in-app scanner; **RedemptionService**; safety net; gate simulator; reconciliation; retry | AC-QR-*, AC-RED-*, AC-GATE-*, AC-AB-12 | Concurrency, unique constraints, hashing, retries |
| M7 | Staff-assisted recovery & ops | Assisted entry; reconciliation queue; ops dashboard (planned/booked/opted-out/redeemed, generator health); planned-quantity edits | AC-STA-*, AC-OPS-* | RBAC in practice, aggregation queries |
| M8 | Admin, audit, feedback | Menus, voiding services, role management UI, audit viewer, feedback | AC-ADMIN-03, AC-AUDIT-01, AC-FB-01 | Admin UX, append-only logs |
| M9 | Simulators & demo mode | Biometric simulator (SIMULATOR label, off in pilot); demo students + simulated checkout; recruiter demo environment with 7-day reset (D-048) | AC-BIO-*, AC-DEMO-* | Interfaces/adapters, feature flags |
| M10 | Release readiness | Deployment (Q-17), backups + restore test, security review, load test of entry burst, privacy notice | Pilot exit criteria (ACCEPTANCE_CRITERIA §0) | Deployment, observability |
| Later | V1.1/V1.2 | Paid diner flow + refunds (D-044), kitchen & inventory, waste, staff attendance (F-015), events & guest coupons, import/export, email notifications, reports | — | — |

**Critical path:** M1 → M2 → M3 → M4 → M6. M5, M7 and M8 can follow in any order once M4 exists.
