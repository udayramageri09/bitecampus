---
name: db-change
description: Safely plan and apply a BiteCampus Prisma schema change or migration, with review, approval and verification.
---

# Database change

1. Run `npx prisma migrate status`. Read `prisma/schema.prisma` and the existing migrations. Never edit an applied migration.
2. Describe the change: tables/columns, constraints (including raw SQL Prisma cannot express), indexes and the data impact on existing rows.
3. Ask the database-reviewer agent to review it, then show its findings.
4. **Ask for approval**, then run `npx prisma migrate dev --create-only --name <name>` so the SQL can be reviewed and hand-edited.
5. Show the generated SQL; add documented raw SQL if needed; get approval to apply.
6. Apply with `npx prisma migrate dev` (local only). Never `migrate reset` or `db push` on a shared database.
7. Update `prisma/seed.ts` if needed (synthetic data only), then run the relevant integration tests.
8. Update `docs/DATABASE_DESIGN.md` if the design changed.
