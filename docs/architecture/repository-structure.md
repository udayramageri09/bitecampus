# Repository Structure and Code Ownership

The repository is an npm-workspaces monorepo. Milestone M1 (`docs/ROADMAP.md`) creates `apps/web` and `apps/api`. Add shared contracts and operational modules when their work begins rather than committing empty placeholder applications. Module names follow `docs/ARCHITECTURE.md` §3.

```text
bitecampus/
├─ apps/
│  ├─ web/                         # React + TypeScript + Vite
│  │  └─ src/
│  │     ├─ app/                   # app shell and route-level composition
│  │     ├─ components/            # reusable UI primitives and brand
│  │     ├─ features/               # meals, passes, scan, admin screens
│  │     ├─ lib/                    # API client, date and formatting helpers
│  │     └─ styles/                 # tokens, global styles, layout styles
│  └─ api/                         # planned Express modular monolith
│     └─ src/modules/              # auth, users, menu, bookings, payments,
│                                  # tickets, events, inventory, staff, reports
├─ packages/
│  └─ contracts/                   # planned Zod DTOs shared with the API
├─ docs/
│  ├─ requirements/               # canonical, approved business rules
│  ├─ architecture/               # ADRs, diagrams and archived drafts
│  └─ runbooks/                    # deploy, backup/restore and incident steps
├─ .github/workflows/              # CI and deployment automation
├─ .gitignore
├─ package.json                    # npm workspaces and root scripts
├─ package-lock.json               # reproducible dependency graph
├─ .nvmrc                          # Node runtime pin
├─ AGENTS.md / CLAUDE.md           # contributor instructions
└─ README.md
```

## Frontend ownership

- `app/` composes navigation, providers and screens; it should not own domain rules.
- `features/<domain>/` owns the UI and client-side state for one product area. Server state stays behind the API client when the backend is introduced.
- `components/` contains reusable visual primitives with no hidden product policy.
- `lib/` contains cross-feature helpers, API transport and formatters.
- `styles/` defines brand tokens and responsive foundations.

## Backend ownership when added

Use one deployable Express service. Each domain module owns its routes, validation, service/use-case and Prisma queries. Controllers handle HTTP; services own business rules and transaction boundaries; PostgreSQL constraints enforce critical invariants. Cross-module operations call another module’s service interface rather than reaching into its tables.

## Repository hygiene

- Keep interview-preparation material outside this repository. The root `.gitignore` also excludes designated private prep directories.
- Keep `.env` files and generated output out of Git; commit only `.env.example` with placeholders.
- Use feature branches and PRs. Do not push directly to `main` or publish private notes.
- `docs/architecture/archive/architecture-rev-2.1-draft.html` is preserved for traceability and is not a canonical requirements source.
