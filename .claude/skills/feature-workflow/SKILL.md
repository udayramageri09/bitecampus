---
name: feature-workflow
description: Plan, get approval for, build, verify and teach one BiteCampus milestone as a small vertical slice. Use for any new feature or behaviour change.
---

# Feature workflow

## A. Plan (stop after this)
1. Run `git status` and read the relevant code, tests and docs (only what this feature needs).
2. Fill in the **Plan** template from `prompts/PROMPT_TEMPLATES.md`, keeping it to about one screen:
   problem · approach · request/data flow · DB changes · API changes · key decision and alternative · failure/security cases · tests · what the owner will learn · steps.
3. List any UNRESOLVED decision touched, with the safe default you will use.
4. **Ask: "Approve this plan?" and stop.** Do not edit files before approval.

## B. Build (after approval)
5. Implement in small steps. After each important file, say in 1–2 lines what it does.
6. Add tests with the code (unit + integration against real PostgreSQL where data or concurrency matters).
7. Ask before installing packages or running migrations.

## C. Verify
8. Run typecheck, lint and the relevant tests. Paste the real summary lines. If something fails, use `/debug-systematically`.
9. Run `git diff --stat` and review the diff (or use the code-reviewer agent).

## D. Teach and hand over
10. Run `/explain-feature` (walkthrough, interview questions, teach-back, learning-log entry).
11. Propose a commit message with no trailers. Wait for approval before `git add` or `git commit`. Never push.

A feature is complete only when verification passed and the owner can explain it.
