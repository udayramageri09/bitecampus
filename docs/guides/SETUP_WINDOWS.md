# SETUP_WINDOWS — one-time setup (PowerShell)

Do these once. Check each tool before installing; **skip anything you already have** (your project already has `node_modules`, so Node is probably installed).

## 1. Check what you have
Open **PowerShell** (Start → "PowerShell") and run:
```powershell
node --version        # want v22 or newer (Claude Code's npm install needs 22+)
npm --version
git --version
psql --version        # PostgreSQL client; may be "not recognized" — fine for now
code --version        # VS Code
claude --version      # Claude Code
```
Write down the versions. The repository pins **Node 24.13.1** (`.nvmrc`). If `node --version` shows a different major version, install Node 24 LTS from nodejs.org (or use `nvm-windows`).

## 2. Install only what's missing
| Tool | How | Verify |
|---|---|---|
| Git for Windows | https://git-scm.com/download/win (keep defaults; includes Git Bash, which Claude Code uses) | `git --version` |
| Node.js LTS | https://nodejs.org (LTS installer) | `node --version` |
| VS Code | https://code.visualstudio.com | `code --version` |
| PostgreSQL | https://www.postgresql.org/download/windows/ (installer; remember the `postgres` password; port 5432) | `psql --version` (add `C:\Program Files\PostgreSQL\<ver>\bin` to PATH if needed) |
| Claude Code | Follow https://code.claude.com/docs/en/setup (native Windows installer), or `npm install -g @anthropic-ai/claude-code` | `claude --version`, then `claude doctor` |

## 3. Git identity (yours, not Claude's)
```powershell
git config --global user.name     # shows your name if set
git config --global user.email
# Only if empty:
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## 4. VS Code extensions (recommended)
ESLint · Prettier · Prisma · Tailwind CSS IntelliSense · REST Client · GitLens (optional) · Claude Code (optional).

## 5. Create local databases
```powershell
psql -U postgres -c "CREATE DATABASE bitecampus_dev;"
psql -U postgres -c "CREATE DATABASE bitecampus_test;"
```
These names are used by the kit's templates.

## 6. Environment variables
- Milestone M1 creates `.env.example`. Copy it to `.env` and fill in your own values. **Never commit `.env` and never paste its contents into chat.**
- `.gitignore` already excludes `.env`.

Example `DATABASE_URL` format (local):
`postgresql://postgres:<your-password>@localhost:5432/bitecampus_dev?schema=public`

## 7. First Claude Code run
```powershell
cd "C:\dev\bitecampus"
claude
```
Log in through the browser when asked, then run `/status` to confirm `.claude/settings.json` loaded. Paste the prompt from `START_HERE.md`.

### Why `C:\dev` and not OneDrive?
OneDrive tries to sync thousands of `node_modules` files, which slows installs and can lock files ("EBUSY" errors). GitHub is your backup instead.
