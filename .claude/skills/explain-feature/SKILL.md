---
name: explain-feature
description: After a BiteCampus milestone, teach it back to the owner — walkthrough of the real code, request flow, edge cases, interview questions, a teach-back exercise and a learning-log entry.
---

# Explain a feature

Use `docs/learning/FEATURE_WALKTHROUGH_TEMPLATE.md`. Keep each section short and point to real `file:line` locations.

1. Summary (3 lines): what was built and why.
2. Request flow as a numbered list or ASCII diagram: UI → API → service → DB → response.
3. Files: one line each on its responsibility.
4. Two or three key code walkthroughs (≤ 15 lines each) with "why it's written this way".
5. Database and API changes.
6. Edge cases and failure handling: what happens on retry, concurrency, timeout and invalid input.
7. Tests run and their real results.
8. One possible improvement.
9. 3–5 interview questions with model answers hidden under a "Try first" heading.
10. Teach-back: ask the owner to explain one part in their own words, then wait and correct.
11. Append an entry to `docs/learning/LEARNING_LOG.md` using the template (ask before creating the file if it doesn't exist).
