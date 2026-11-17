# BiteCampus

*Your campus. Your food. Your way.*

A hostel-mess platform for a controlled BMSCE pilot (~500 users). Hostellers are booked automatically every day at ₹0 (covered by the mess fee). At the mess entrance they scan a rotating QR shown by a terminal, and the server redeems each meal coupon exactly once.

**Status:** fresh start, documentation only. The app is built milestone by milestone (`docs/ROADMAP.md`). Nothing is implemented, deployed or approved by the college yet.

## Stack (planned)
React + TypeScript + Vite + Tailwind · Node.js 24 + Express + TypeScript · PostgreSQL + Prisma · Zod · Vitest, Supertest, Playwright. One modular monolith in an npm-workspaces repo (`apps/web`, `apps/api`).

---

## Fresh setup on Windows (PowerShell, in VS Code)

**1. Create the folder and connect it to GitHub**
```powershell
New-Item -ItemType Directory -Force C:\dev | Out-Null
cd C:\dev
git clone https://github.com/udayramageri09/bitecampus.git
cd bitecampus
git status          # "No commits yet" is expected — the repo is empty
```

**2. Add these starter files.** Extract `bitecampus-fresh-start.zip` and copy the **contents** of its `bitecampus` folder into `C:\dev\bitecampus`, including the hidden `.claude` folder, `.gitignore` and `.nvmrc`. Then check:
```powershell
Get-ChildItem -Force          # should list .claude, .gitignore, .nvmrc, AGENTS.md, CLAUDE.md, docs, prompts, templates ...
```

**3. First commit and push (your identity)**
```powershell
git config user.name; git config user.email      # must be YOU (see docs/guides/SETUP_WINDOWS.md §3)
git add -A
git status
git commit -m "docs: project specification and Claude Code setup"
git push -u origin main
```
If `git push` says the branch is `master`, run `git branch -M main` first, then push again.

**4. Open in VS Code and start Claude Code**
```powershell
code .
```
In VS Code open a terminal (Ctrl + `) and run `claude`, or use the Claude Code extension. Run `/status` to check the project settings loaded, then paste the prompt from `START_HERE.md`.

From then on, work on a branch per milestone and push it yourself (`docs/guides/DAILY_WORKFLOW.md`). Claude Code is configured never to push.

## Where things are
- `CLAUDE.md` / `AGENTS.md` — rules for Claude Code and contributors
- `docs/OPEN_DECISIONS.md` — every decision, assumption and open question
- `docs/ROADMAP.md` — build order (M1 → M10)
- `docs/guides/` — Windows setup and daily commands
- `docs/learning/` — how you'll learn each feature, plus interview prep
- `prompts/PROMPT_TEMPLATES.md` — copy-paste prompts
- `docs/requirements/approved-baseline.md` — earlier baseline, reconciled in `docs/OPEN_DECISIONS.md` §0b
