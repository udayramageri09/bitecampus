# TESTING_STRATEGY

Goal: prove the invariants that matter (automatic booking, exactly-once redemption, authorisation, time rules), not chase coverage numbers.

## 1. Layers
| Layer | Tool | What | Database |
|---|---|---|---|
| Unit | Vitest | Pure rules: cutoff/window/horizon in IST, eligibility function, state transitions, QR payload parsing, HMAC signing | none |
| Integration | Vitest + Supertest | HTTP → service → **real PostgreSQL**: auth, RBAC, generator idempotency, opt-out, redemption, concurrency, gate simulator | dedicated test DB |
| End-to-end | Playwright | Phone-sized browser: sign-in (dev stub) → coupons → scan (injected QR payload) → result; terminal screens | test DB |
| Manual | checklist | Real phone camera scanning; real terminal screen; accessibility spot checks | demo DB |

## 2. Test database
- A separate database, e.g. `bitecampus_test`, set through `DATABASE_URL` in `.env.test` (never committed).
- Before the suite: `npx prisma migrate deploy` against the test DB. Between tests: truncate tables in a helper (`TRUNCATE … RESTART IDENTITY CASCADE`), except reference data reseeded by the helper.
- Never point tests at the development or pilot database. The helper refuses to run unless the database name ends with `_test`.

## 3. Controlling time and randomness
- `Clock` is injected; tests use `fixedClock('2026-12-01T04:29:59Z')` (09:59:59 IST).
- `Random` is injected for tokens in unit tests; integration tests use the real CSPRNG.
- Always include cases on both sides of an IST boundary (cutoff, window end, midnight = 18:30 UTC).

## 4. Concurrency and idempotency tests (the important ones)
```ts
// AC-RED-03 sketch
const attempts = Array.from({ length: 50 }, () =>
  request(app).post('/api/v1/entry/qr-redemptions')
    .set('Cookie', studentCookie).set('Idempotency-Key', randomUUID())
    .send({ challengeToken: freshChallengeFor(terminal) }));
const results = await Promise.all(attempts);
expect(await db.redemption.count({ where: { couponId } })).toBe(1);      // assert the DB, not just HTTP
expect(results.filter(r => r.status >= 500)).toHaveLength(0);
```
The same pattern covers AC-AB-03 (two generator runs in parallel), AC-QR-06 (one challenge, two students), AC-RED-01 (QR + biometric) and AC-STA-04.

## 5. Simulators in tests
Gate and biometric simulators expose deterministic modes (`OPEN`, `REJECT`, `FAULT`, `TIMEOUT`, `DELAYED(ms)`); tests set the mode directly through the injected instance, not over HTTP.

## 6. Commands (PowerShell; names are confirmed during M1)
```powershell
npm run typecheck
npm run lint
npm test                    # unit + integration
npm run test:e2e            # Playwright
npm test -- --coverage      # report only; no target is claimed until measured
```

## 7. Definition of "tested"
A milestone is tested when its AC IDs have tests that ran and passed, the output was shown, and failures were fixed at the root cause. Coverage is reported, never claimed in advance (D-032).
