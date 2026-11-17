# BiteCampus contributor instructions

Read this file and `docs/requirements/approved-baseline.md` before changing product behavior. The user’s latest explicit decisions take precedence over older drafts; report conflicts and update the canonical requirements when requested.

## Product and architecture boundaries

- V1 is a BMSCE hostel-mess pilot. Demo and test environments use synthetic data; never imply that demo interactions move real money or food.
- Use a responsive React + TypeScript + Vite app in `apps/web`. The backend, when started, is one Node.js + TypeScript + Express modular monolith backed by PostgreSQL and Prisma.
- Keep paid diner coupons, prepaid hostel meal access and invited event-guest coupons as separate product flows. Reuse presentation where useful, but do not merge their authorization or accounting.
- Main Admin configures academic-year dates and applicable hostel meal rules in the app. Meal Admin manages meal prices and booking windows and may reopen a window. Refund execution requires Hostel Director permission and is limited to hostel-management-caused issues.
- Never treat user-selected profile categories as authority for hostel entitlements or staff/admin roles. Enforce access on the server when the API exists.
- The Rev. 2.1 architecture draft under `docs/architecture/archive/` is historical and not an approval source where it conflicts with the current baseline.

## Repository organization

- `apps/web`: app bootstrap/routes, shared UI, feature-owned screens, API client and styles.
- `apps/api`: planned modular-monolith API; each domain owns its routes, service and data access.
- `packages/contracts`: planned shared request/response schemas; never put secrets here.
- `docs/requirements`: confirmed decisions and unresolved policy details.
- `docs/architecture`: diagrams and short decision records.
- `docs/runbooks`: deployment, restore and incident procedures.
- Keep personal interview-preparation material outside this repository. Never stage or push it.

## Engineering practices

- Keep business rules out of UI components. Validate inputs and enforce authorization on the server.
- Use accessible semantic HTML, responsive layouts, keyboard support, clear loading/empty/error states, and honor reduced-motion preferences.
- Use environment variables for secrets; commit placeholders only in `.env.example`.
- Keep feature changes small and reviewable. Use feature branches and pull requests; do not push directly to `main`.
- Explain significant implementation choices, file ownership, request flow, failure cases and interview-relevant tradeoffs to the user.

## V3 starter kit (added 2026-11-17)

- `CLAUDE.md` and `docs/` now hold the V3 product and engineering specification. `docs/OPEN_DECISIONS.md` is the decision register and reconciles `docs/requirements/approved-baseline.md` (see its §0b). Where they differ, the owner's latest explicit decision wins, as stated above.
- Key updates since the baseline: hostellers are **booked automatically** each day at ₹0; at the mess entrance the **terminal displays a rotating QR and the student scans it in the app** (Counter Staff provide staff-assisted recovery); the paid diner flow, refunds and staff attendance move to V1.1; the frontend-only milestone is replaced by `docs/ROADMAP.md`. The repository was restarted fresh on 2026-11-17; the earlier scaffold is kept outside the repo.
- Claude Code permissions live in `.claude/settings.json`: no pushes, no `.env` reads, approval before installs, migrations and commits, and no co-author trailers.
