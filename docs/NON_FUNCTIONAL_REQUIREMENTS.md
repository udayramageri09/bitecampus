# NON_FUNCTIONAL_REQUIREMENTS

> All numeric targets are **targets, not results** (D-032). A target becomes a claim only after a recorded, repeatable measurement (tool, environment, dataset, request mix, duration, result).

| ID | Area | Requirement | Status | Source |
|---|---|---|---|---|
| NFR-01 | Scale | Support ~500 users of one hostel mess | APPROVED | D-002 |
| NFR-02 | Peak load | Entrance bursts at meal start are the critical load. Target: 50–100 concurrent synthetic redemption/opt-out requests without errors or duplicate effects; the automatic-booking run completes within P-020's target | PROPOSED | P-020 |
| NFR-03 | Latency | p95 < 300 ms for ordinary API calls; p95 < 1 s for QR redemption excluding the gate response | PROPOSED | P-020 |
| NFR-04 | Correctness | Zero duplicate redemptions under concurrent and retried requests, proven by integration tests against real PostgreSQL | APPROVED | D-022 |
| NFR-05 | Availability | Designed for one region and one instance. No HA promise in the pilot. Documented manual fallback for entrance outage | PROPOSED | ARCH |
| NFR-06 | Fail-safe | When BiteCampus is unreachable, entry is **not** auto-granted. A manual process agreed with the mess applies (outside the system; to be agreed with the college) | APPROVED intent (V2 "do not fail open") | V2 |
| NFR-07 | Security | `SECURITY.md` baseline applies to every release | APPROVED | V2, V3 |
| NFR-08 | Privacy | No biometric data; no PII in QR, URLs or logs; synthetic data outside an authorised pilot | APPROVED | D-018, D-020, D-030 |
| NFR-09 | Auditability | All DR-A2 events are durable and attributable to a user or terminal | APPROVED | V3 |
| NFR-10 | Accessibility | WCAG 2.1 AA; keyboard access; visible focus; contrast; labels; respects reduced motion | PROPOSED | P-021 |
| NFR-11 | Mobile | Mobile-first student UI. In-app scanner works on current Android Chrome and iOS Safari with camera permission handling | APPROVED (mobile-first) / PROPOSED (browser list) | V2 |
| NFR-12 | Maintainability | Modular monolith with module boundaries per `ARCHITECTURE.md`; strict TypeScript; Zod at boundaries | APPROVED | D-003, V2 |
| NFR-13 | Testability | Time, randomness, payment, biometric and gate are injectable dependencies | PROPOSED | ARCH |
| NFR-14 | Observability | Structured JSON logs with request/correlation ID; health and readiness endpoints; error tracking (`OBSERVABILITY.md`, Phase Three) | PROPOSED | ARCH |
| NFR-15 | Data durability | Daily automated database backup, with restore tested before real pilot data is accepted | APPROVED | V2 |
| NFR-16 | Configurability | Business values editable without a deploy | APPROVED | D-017 |
| NFR-17 | Portability | Developer workflow on Windows/PowerShell | APPROVED | V3 |
| NFR-18 | Replaceability | Payment, biometric and gate implementations can be swapped without changing `RedemptionService` or booking logic | APPROVED | V3 |
| NFR-21 | Job resilience | Automatic booking stays correct with any number of runs; missed or failed runs are repaired by the next run's catch-up and by the entry-time safety net; job health is visible to staff | APPROVED | D-035 |
| NFR-19 | Time correctness | All policy decisions in `Asia/Kolkata`, tested across midnight and cutoff boundaries | APPROVED | D-016 |
| NFR-20 | Cost | Fits a student/free-tier budget; no paid infrastructure without approval | PROPOSED | ARCH, Q-17 |
