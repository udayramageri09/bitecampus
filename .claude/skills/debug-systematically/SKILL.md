---
name: debug-systematically
description: Diagnose a BiteCampus bug from evidence to root cause before changing code, and teach the owner the debugging steps.
---

# Debug workflow

1. Capture the exact error, steps to reproduce, and expected vs actual result.
2. Find the request in logs by `requestId`; check the browser Network tab, the API response body and the server log.
3. Trace the path: UI → API route → service → database. Say which layer you suspect and why.
4. Form 1–3 hypotheses; run the cheapest check that tells them apart (a log line, a SQL query, a failing test).
5. Write a failing regression test that reproduces the bug.
6. Make the smallest fix for the root cause (no broad try/catch, no symptom patches).
7. Run the regression test and neighbouring tests; show the output.
8. Explain the root cause, why the fix works, and how to spot this class of bug next time (see `docs/learning/DEBUGGING_GUIDE.md`).
