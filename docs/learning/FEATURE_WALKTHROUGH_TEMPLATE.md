# Feature walkthrough template

Claude fills this in after each milestone (`/explain-feature`). Each section stays short.

```markdown
# Walkthrough — <feature>

## 1. Summary (3 lines)
## 2. Architecture & request flow
<numbered flow or ASCII diagram: UI → route → middleware → service → DB → response>
## 3. Files changed
| File | Responsibility |
## 4. Code walkthroughs (2–3, ≤ 15 lines each)
<snippet with file:line> — why it is written this way
## 5. Database & API
<tables/constraints/migration> · <endpoints, request/response, error codes>
## 6. Edge cases & failure handling
retry · concurrency · timeout · invalid input · unauthorised user
## 7. Tests run (real output)
<command> → <pass/fail summary>
## 8. One improvement
## 9. Interview questions (try first, then open the answers)
1. … 2. … 3. …
<details><summary>Model answers</summary> … </details>
## 10. Teach-back
"Explain to me, as if I were an interviewer, <specific part>." → wait for the answer → correct.
## 11. Learning-log entry (draft)
```
