# Prompt templates

Copy a prompt into Claude Code and fill in the `<…>` parts. Keep requests small: one milestone or one bug at a time.

---

## 1. Plan a milestone
```text
/feature-workflow
Milestone: <M-number and name from docs/ROADMAP.md>
Goal in one sentence: <…>
Only plan for now. Use the plan format below and stop for my approval.
```

**Plan format Claude should return (about one screen):**
```text
PLAN — <milestone>
1. Problem: …
2. Approach (and one alternative we're not choosing, with why): …
3. Request / data flow: UI → API → service → DB → response
4. Database changes: …            5. API changes: …
6. Failure & security cases: …    7. Tests I will write: …
8. What you'll learn: …           9. Steps (small): 1) … 2) … 3) …
Open decisions touched → safe default used: …
Approve this plan?
```

## 2. Implement an approved plan
```text
Approved. Implement step <n> (or "all steps") of the plan.
Explain each important file in 1–2 lines as you go.
Ask before installing packages or running migrations. Don't commit.
```

## 3. Review before commit
```text
/review-before-commit
Then show me the commit message (no Co-Authored-By trailer) and the PowerShell commands.
```

## 4. Test
```text
Use the qa-engineer agent. Write and run tests for <AC-IDs from docs/ACCEPTANCE_CRITERIA.md>.
Run against the test database. Show real pass/fail counts.
```

## 5. Debug
```text
/debug-systematically
Error: <paste exact error>
Steps: <what I clicked or ran>
Expected: <…>   Actual: <…>
Teach me how you narrowed it down.
```

## 6. Learn
```text
/explain-feature
Feature: <name>. I want to be able to explain it in an interview.
Ask me the teach-back question and wait for my answer.
```

## 7. Explain one thing
```text
Use the learning-mentor agent. Explain <file or concept> in this repo.
Compare it to C++ if that helps. Keep it short, then ask me one question.
```

## 8. Release
```text
/release-checklist
Target environment: <local | demo | pilot>. Don't deploy or migrate without asking.
```

## 9. End of session
```text
Update docs/SESSION_HANDOFF.md: current state, commands verified today, blockers, next step.
```
