# DEPLOYMENT

> Hosting is **not chosen yet** (Q-17). This document fixes the requirements and the procedure so any reasonable host can be picked in M10.

## 1. Requirements for a host
- Node.js runtime for the API (one instance is enough, NFR-01), static hosting for the React build, managed PostgreSQL with automated daily backups and point-in-time or snapshot restore.
- HTTPS on a stable domain (required for Google sign-in origins and for the phone camera API).
- A way to run the automatic-booking job hourly (platform cron, or the in-process scheduler on an always-on instance). The generator also runs on startup and by manual trigger, so a sleeping free-tier instance is tolerable for demos but **not** for the pilot.
- Secret management for environment variables.
- Budget: student/free-tier friendly (NFR-20). Check current pricing and student-pack offers when choosing; do not rely on remembered prices.

## 2. Environments
| Env | Purpose | Data | `DEMO_MODE` | Biometric sim |
|---|---|---|---|---|
| local | development | synthetic | true | on |
| test/CI | automated tests | synthetic, reset | true | on |
| demo | recruiters, demonstrations | synthetic | true | on |
| pilot | real controlled pilot | authorised roster only (Q-18) | **false** | **off** |

## 3. Environment variables (names only — values never in git)
`NODE_ENV`, `PORT`, `DATABASE_URL`, `WEB_ORIGIN`, `GOOGLE_CLIENT_ID`, `ALLOWED_EMAIL_DOMAINS`, `SESSION_SECRET`, `TERMINAL_SECRET_KEY`, `DEMO_MODE`, `BIOMETRIC_SIMULATOR_ENABLED`, `AUTH_DEV_STUB`, `QR_CHALLENGE_TTL_SECONDS`, `QR_ROTATE_SECONDS`, `GATE_TIMEOUT_MS`, `AUTO_BOOKING_HORIZON_DAYS`, `AUTO_BOOKING_CRON`, `LOG_LEVEL`. Template: `templates/env.example`.

## 4. Release procedure (owner approves each step)
1. `/release-checklist` → all checks green.
2. Build: `npm run build` (web and API).
3. Database: `npx prisma migrate status` → **backup** → `npx prisma migrate deploy` → verify.
4. Deploy the API, then the web build. Smoke-test (release checklist step 7).
5. Trigger an automatic-booking run; confirm the ops view shows the horizon with no integrity warnings.
6. Record version, time and results in `docs/SESSION_HANDOFF.md`.

## 5. Rollback
- Code: redeploy the previous build.
- Database: migrations are additive, so the previous code normally runs on the new schema. If not, restore the pre-deploy backup (practised beforehand, NFR-15).

## 6. Google OAuth configuration
In Google Cloud Console: an OAuth client of type "Web application"; authorised JavaScript origins = the exact web URLs (local `http://localhost:5173` and the deployed `https://…`). Never commit the client secret (the ID-token flow does not need one in the browser).
