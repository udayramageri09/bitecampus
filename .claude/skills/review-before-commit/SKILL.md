---
name: review-before-commit
description: Review BiteCampus changes for correctness, security, tests, scope and Git hygiene, then propose a commit for the owner to approve.
---

# Pre-commit review

1. `git status --short` and `git diff` (staged and unstaged).
2. Block if you find: `.env` or secrets, real student data, `node_modules`/build output, debug logs, unrelated reformatting, unapproved dependencies.
3. Check authorisation on every new route and ownership checks in services.
4. Check database constraints and transactions for any state change.
5. Run typecheck, lint and tests; show the results.
6. Report findings: Must-fix / Should-fix / Nice-to-have, with file:line.
7. Propose a conventional commit message (e.g. `feat(booking): idempotent automatic daily booking`) with **no** Co-Authored-By or tool trailers.
8. Show the exact PowerShell commands the owner can run, or wait for "commit it". Never push.
