# DEBUGGING_GUIDE (Windows + Chrome)

## 1. The method (same every time)
1. **Reproduce** reliably. Write down the steps.
2. **Locate the layer:** browser → network → API → service → database.
3. **Hypothesise** (1–3 ideas). **Test the cheapest one first.**
4. **Prove** with a failing test. **Fix** the root cause. **Re-run** the tests.

## 2. Browser — Chrome DevTools (F12)
- **Console:** red errors, stack traces. Click the file:line link to jump to the source.
- **Network:** filter by `Fetch/XHR`. Click a request and check: status code, **Response** (our error envelope has `code` and `requestId`), **Headers** (cookies sent? `X-CSRF-Token` present?).
- **Application → Cookies:** is `bc_session` present, `HttpOnly`, for the right domain?
- **Device toolbar (Ctrl+Shift+M):** test phone layouts.
- **Sources:** set breakpoints in the React code (Vite serves source maps in dev).
- Phone camera issues: the camera works only on `https://` or `http://localhost`. For a real phone, use the deployed demo or a tunnel.

## 3. API
- Use VS Code's **REST Client** extension or `Invoke-RestMethod` in PowerShell:
```powershell
Invoke-RestMethod -Uri "http://localhost:3000/api/v1/health"
Invoke-RestMethod -Uri "http://localhost:3000/api/v1/meal-services?from=2026-12-01&to=2026-12-02" -WebSession $session
```
- Read the server terminal: find the log line with the same `requestId`.
- Debug in VS Code: **Run and Debug → JavaScript Debug Terminal**, then `npm run dev`; breakpoints in `.ts` files work.

## 4. Database
- `npx prisma studio` opens a browser view of the tables (read carefully; avoid editing pilot data).
- Check the rows the request should have created: `Booking`, `MealCoupon`, `Redemption`, `EntryAttempt`, `GateCommand`, `JobRun`, `AuditEvent`.
- Unique-violation errors (Prisma `P2002`) usually mean the *protection worked*; check whether the service translated it into a domain error.

## 5. Common BiteCampus bugs and where to look
| Symptom | First suspect |
|---|---|
| Coupon missing for a hosteller | Generator run (`JobRun`), membership dates/`recordedAt`, student status, IST date calculation |
| "Cutoff passed" too early or late | UTC vs IST conversion; clock injection in tests |
| 401 after sign-in | Cookie not set (CORS `credentials`, `SameSite`, wrong origin) |
| 403 on a staff route | Missing permission in the role matrix |
| QR always "expired" | Server/terminal TTL config; terminal not rotating |
| Two redemptions | Should be impossible: check the unique constraint exists in the DB (`migrate status`) |
| Gate shows pending forever | `GateCommand` status `UNKNOWN`; reconciliation job running? |

## 6. Ask Claude well
Use the Debug prompt in `prompts/PROMPT_TEMPLATES.md` with the exact error, steps, expected and actual result, and the `requestId`.
