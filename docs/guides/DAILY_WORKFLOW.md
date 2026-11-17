# DAILY_WORKFLOW — everyday commands (PowerShell)

> Script names below are the **expected** ones. The M1 milestone confirms or adds them in your `package.json`; until then, check `npm run` to see what exists.

## 1. Start your day
```powershell
cd "C:\dev\bitecampus"
git status                 # anything uncommitted from last time?
git pull                   # only once the repo is on GitHub and you work on more than one machine
claude                     # start Claude Code; paste a prompt from prompts/PROMPT_TEMPLATES.md
```

## 2. Run the app locally
```powershell
npm run dev                # starts API + web (or run each in its own terminal: npm run dev:api / npm run dev:web)
```
- Web: http://localhost:5173 · API: http://localhost:3000/api/v1/health
- Stop with `Ctrl + C`.

## 3. Database
```powershell
npx prisma migrate status          # are all migrations applied?
npx prisma migrate dev --name <change>   # create + apply a migration (local only; Claude asks first)
npx prisma db seed                 # load synthetic data
npx prisma studio                  # browse tables in the browser
```
Never run `npx prisma migrate reset` on a database whose data you care about.

## 4. Tests and checks
```powershell
npm run typecheck
npm run lint
npm test
npm run test:e2e
```

## 5. Git and GitHub workflow (you stay in control)
1. Start each milestone on a branch:
   ```powershell
   git switch -c feat/m4-auto-booking
   ```
2. Claude builds and runs `/review-before-commit`, then shows you the diff summary and a message.
3. **You** commit (or tell Claude "commit it"):
   ```powershell
   git add -A
   git status                       # check: no .env, no node_modules
   git commit -m "feat(booking): idempotent automatic daily booking"
   ```
4. **You** push (Claude Code is configured never to push):
   ```powershell
   git push -u origin feat/m4-auto-booking
   ```
5. On GitHub, open a pull request into `main`, read it once, and merge. Then:
   ```powershell
   git switch main
   git pull
   ```
Commit message style: `type(scope): what changed`, where type is `feat`, `fix`, `test`, `docs`, `refactor` or `chore`. No Co-Authored-By lines.

## 6. When something breaks
Use the Debug prompt (`prompts/PROMPT_TEMPLATES.md` §5) and `docs/learning/DEBUGGING_GUIDE.md`.

## 7. End your day
```text
Update docs/SESSION_HANDOFF.md: state, commands verified, blockers, next step.
```
Then commit your work on the branch (step 5), even if it's unfinished, and push if you want a backup on GitHub.

## 8. Build and deploy
See `docs/DEPLOYMENT.md` and `/release-checklist`. Nothing is deployed without your explicit approval.
