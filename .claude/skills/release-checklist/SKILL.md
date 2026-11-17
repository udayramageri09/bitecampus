---
name: release-checklist
description: Prepare and verify a BiteCampus demo or pilot release, including environment, database, security, smoke tests and disclaimers.
---

# Release checklist

1. Which environment: local, demo or **pilot**? (`docs/SYSTEM_DESIGN.md` §8.)
2. Pilot must have `DEMO_MODE=false` and the biometric simulator **disabled**; demo coupons are rejected.
3. Check the environment variables are present (names only; never print values). OAuth origins and redirect URIs match the deployed URL.
4. Migration plan: `npx prisma migrate status`, then backup, then `migrate deploy` (owner approves), then verify. Restore has been tested.
5. HTTPS, CORS allow-list, security headers, rate limits, safe error messages, log redaction.
6. Automatic booking: run the generator, then check the ops view shows coupons for the horizon and no integrity warnings.
7. Smoke test: sign in → see auto coupons → scan the terminal QR → redeemed → scan again → already redeemed → gate timeout → retry works → staff-assisted entry works.
8. Privacy notice and the "independent pilot" disclaimer are visible; only synthetic or authorised data is present.
9. Record the version, commands, results, known issues and rollback plan in `docs/SESSION_HANDOFF.md`.
10. Do not deploy, migrate production or change DNS/secrets without explicit approval.
