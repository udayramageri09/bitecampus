# START HERE — first Claude Code session

Open the repository in VS Code, start `claude` in the terminal, and paste this prompt.

```text
We are starting BiteCampus fresh. The repository contains documentation and Claude Code configuration only — no app code yet.

This first session is READ-ONLY until I approve a plan:
- Do not create, edit or delete files. Do not install packages, run migrations, commit or push.
- Do not read or print any .env file.

Please:
1. Read CLAUDE.md, AGENTS.md, docs/OPEN_DECISIONS.md (§0, §0b), docs/ROADMAP.md, docs/ARCHITECTURE.md and docs/learning/LEARNING_CONTRACT.md.
2. Check my tools and report the versions: node --version, npm --version, git --version, psql --version (if installed), and git status / git remote -v.
   Compare node with .nvmrc (24.13.1). Tell me exactly what to install if anything is missing (PowerShell commands or download links).
3. In 5 bullet points, explain in plain language what BiteCampus is and how the pieces fit together, so I know you understood the docs.
4. Propose the plan for M1 — Foundation using the plan format in prompts/PROMPT_TEMPLATES.md (one screen):
   npm workspaces root; apps/web (React + TypeScript + Vite + Tailwind, brand tokens per D-049, a simple app shell with a
   placeholder home screen); apps/api (Express + TypeScript, env validation with Zod, /health and /ready, error envelope,
   request-ID logging); Vitest + Supertest with one passing health test; root scripts dev / build / typecheck / lint / test;
   .env.example. Postgres + Prisma come in M1b or M2 — tell me which you recommend and why.
   List every package you want to install, with one line each on why. Choose current stable versions compatible with Node 24
   and tell me the versions before installing.
5. Tell me what I will learn in M1 and what I should read first.

Then STOP and wait for me to reply "approved".
```

After you approve, Claude builds M1 in small steps, runs the checks, explains the code (`/explain-feature`) and proposes a commit. You push the branch yourself.
