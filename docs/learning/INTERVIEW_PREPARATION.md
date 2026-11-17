# INTERVIEW_PREPARATION

Practise answering out loud in under 2 minutes each. Model answers are short on purpose; expand them with your own code.

## 1. Your 60-second pitch
"BiteCampus is a hostel-mess platform I built for a controlled pilot at BMSCE (about 500 students). Hostellers are booked automatically every day; at the mess entrance they scan a rotating QR shown by a terminal, and the server redeems their meal coupon exactly once, even under retries and concurrent scans. I designed it as a modular monolith on React, Express, PostgreSQL and Prisma, with simulators for biometric, payment and gate hardware behind interfaces so real integrations can be plugged in later."

## 2. System design questions
1. **How do you guarantee a coupon is redeemed only once?** A unique constraint on `Redemption.couponId`, a conditional update `WHERE status='ACTIVE'` in the same transaction, and idempotency keys so retries replay the original result. The database decides the race, not application code.
2. **Your daily booking job runs twice. What happens?** Nothing extra: inserts use `ON CONFLICT DO NOTHING` on `UNIQUE(studentId, mealServiceId)`. Re-runs, overlapping runs and crash-then-retry converge to the same state.
3. **The gate times out after you redeemed. Now what?** The redemption is already committed; the gate command is `UNKNOWN`. We query the controller and otherwise queue it for reconciliation. A retry sends a new gate command for the same redemption. We never redeem again.
4. **Why is the QR displayed by the terminal instead of the student?** The terminal's QR proves the student is physically at that entrance right now; it is short-lived and single-use. A student-shown QR can be screenshotted and shared. The coupon never appears in a QR.
5. **Why a modular monolith, not microservices?** One team and about 500 users. The critical operations need cross-table transactions that are trivial in one database and hard across services. Module boundaries keep the option to split later.
6. **Why sessions over JWTs?** Instant revocation, nothing sensitive in the browser, and a per-request DB lookup is cheap at this scale.
7. **How do you handle time zones?** Store UTC; evaluate rules in Asia/Kolkata; meal dates are campus-local dates; tests fix the clock on both sides of boundaries.
8. **What would you change for 50,000 users?** Measure first. Likely: read replicas for dashboards, a real job queue, WebSockets/SSE for terminals, horizontal API instances. The redemption design stays the same because it relies on database constraints.

## 3. Debugging story template (STAR)
Situation → Task → Action (how you narrowed it down layer by layer) → Result (fix + regression test + what you learned).

## 4. Questions to expect about your code
Walk through one request end to end · show a migration and explain each constraint · show a concurrency test and why it asserts row counts · explain the error envelope and status codes · explain what you would test next.

## 5. Honesty rules
Say "simulated" for biometric, payment and gate. Quote only measured numbers. Say what you would do differently.
