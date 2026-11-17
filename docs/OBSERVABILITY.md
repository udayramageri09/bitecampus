# OBSERVABILITY

Keep it simple: structured logs, two health endpoints, a few operational counters visible in the ops dashboard.

## 1. Logs
- JSON lines, one per event, via a small logger (e.g. pino — confirm against existing dependencies first).
- Every request gets a `requestId` (from `X-Request-Id` or generated) returned in the response header and in every error envelope.
- Standard fields: `time`, `level`, `requestId`, `principalType` (user/terminal/system), `principalId`, `route`, `status`, `durationMs`, `event`.
- **Redact:** cookies, `Authorization`, `X-Signature`, `challengeToken`, `credential`, `Idempotency-Key` values, emails in request bodies. Never log `.env` values.
- Levels: `error` (bug or dependency failure), `warn` (rejected entry, gate timeout, job retry), `info` (request summary, job summary), `debug` (local only).

## 2. Domain events worth logging (at `info`)
`auto_booking.run_completed {counts}` · `opt_out.completed` · `entry.redeemed {method, mealType}` · `entry.rejected {reasonCode}` · `gate.unknown {gateCommandId}` · `reconciliation.resolved` · `staff_assist.redeemed {actorId}`.
Logs help debugging; `AuditEvent` is the durable record. Do not use logs as the audit trail.

## 3. Health
- `GET /health` → 200 when the process is up (no DB call).
- `GET /ready` → 200 when the DB is reachable and migrations are applied; 503 otherwise.

## 4. Operational signals (ops dashboard, FR-OPS-03/04)
- Automatic booking: last successful run, horizon covered, services with integrity warnings.
- Entrance: attempts per minute, rejection reasons, `UNKNOWN` gate commands, reconciliation queue size, staff-assisted count per staff member.
- Errors: count of 5xx in the last hour.

## 5. Error tracking
Optional for the pilot. If added (e.g. Sentry free tier), scrub PII and tokens before sending. Decide in M10.

## 6. How to investigate an issue
1. Get the `requestId` from the UI error message or the browser Network tab.
2. Search the server logs for it.
3. Check the related `EntryAttempt` / `GateCommand` / `JobRun` rows.
4. Follow `docs/learning/DEBUGGING_GUIDE.md`.
