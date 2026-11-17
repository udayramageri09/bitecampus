# BiteCampus — Claude Code Starter Kit V3

*Your campus. Your food. Your way.*

This kit gives Claude Code everything it needs to help you build BiteCampus safely, and to teach you while it does.

## Use it in 3 steps
1. **Set up:** follow the root `README.md` (new folder, clone, first push).
2. **Orient:** start `claude` and paste the prompt from `START_HERE.md`.
3. **Build:** approve the M1 plan, then follow `docs/ROADMAP.md` with the prompts in `prompts/PROMPT_TEMPLATES.md`.

New to the tools? `docs/guides/SETUP_WINDOWS.md` first, then `docs/guides/DAILY_WORKFLOW.md`.

## What's inside
```text
CLAUDE.md                      Always-on rules for Claude Code (short)
START_HERE.md                  First session: read-only repository audit prompt
README.md (repo root)          Fresh setup: new folder, clone, first push
KIT_CHANGELOG.md               What changed from V2; preserved/new/unresolved requirements
.claude/
  settings.json                Permissions (no push, no .env reads, ask before installs/migrations/commits);
                               commit/PR attribution off
  agents/                      software-architect · backend-engineer · frontend-engineer · database-reviewer ·
                               security-reviewer · qa-engineer · code-reviewer · learning-mentor
  skills/                      feature-workflow · debug-systematically · review-before-commit · explain-feature ·
                               db-change · release-checklist
prompts/PROMPT_TEMPLATES.md    Copy-paste prompts: plan, implement, review, test, debug, learn, release
templates/                     env.example · gitignore.additions (references only)
docs/
  Product:   PRODUCT_BRIEF · PRD · FUNCTIONAL_REQUIREMENTS · NON_FUNCTIONAL_REQUIREMENTS · DOMAIN_RULES ·
             USER_ROLES_AND_PERMISSIONS · USER_FLOWS · ACCEPTANCE_CRITERIA · OPEN_DECISIONS · ROADMAP
  Technical: ARCHITECTURE · SYSTEM_DESIGN · DATABASE_DESIGN · API_CONTRACTS · SECURITY ·
             AUTHENTICATION_AND_AUTHORIZATION · BIOMETRIC_QR_REDEMPTION_DESIGN · TESTING_STRATEGY ·
             DEPLOYMENT · OBSERVABILITY · SESSION_HANDOFF · KIT_V2_AUDIT
  learning/  LEARNING_CONTRACT · LEARNING_LOG_TEMPLATE · FEATURE_WALKTHROUGH_TEMPLATE · DEBUGGING_GUIDE ·
             INTERVIEW_PREPARATION · TECHNICAL_DECISION_RECORD_TEMPLATE
  guides/    SETUP_WINDOWS · DAILY_WORKFLOW
```

## Three ideas to remember
1. **Hostellers are booked automatically**, at ₹0, every day. The job is safe to run any number of times.
2. **The terminal shows the QR; the student's app scans it.** Every entry method calls one redemption service, and each coupon can be redeemed only once.
3. **You own the repo.** Claude plans, waits for approval, never pushes, never adds co-author lines, and never touches `.env`.

## Honest status
This kit is documentation and configuration only. It does not mean any feature is implemented, tested, deployed or approved by the college. Targets are not results.
