# BiteCampus Starter Kit — V2 Audit & Gap Analysis (Phase One)

**Input audited:** `bitecampus-claude-starter-v2.zip` (uploaded 2026-11-17)
**Extraction:** succeeded, no errors. 20 files, 1 root folder (`bitecampus-claude-starter/`).
**Scope of this audit:** the ZIP only. The local repository (`C:\Users\udayr\OneDrive\Desktop\Projects\bitecampus`) and the GitHub repository were **not** inspected. Nothing in this report describes the state of the application code.

---

## 1. Existing kit structure (complete listing)

```text
bitecampus-claude-starter/
├── CLAUDE.md                          always-on project instructions (~75 lines)
├── README.md                          install steps, layout, token-efficiency model
├── START_HERE.md                      first-session prompt for Claude Code
├── docs/
│   ├── ARCHITECTURE.md                modular monolith, stack, repo layout, modules, data-model candidates
│   ├── DECISIONS.md                   decision register (DECIDED / WORKING DEFAULT / PENDING / DEFERRED)
│   ├── DELIVERY_PLAN.md               3–4 day feedback-release plan, scope-cut order, DoD
│   ├── DOMAIN_RULES.md                time, booking, simulated payment, ticket/QR, authz, feedback rules
│   ├── PILOT_AND_FEEDBACK.md          feedback-demo rules, survey prompts, evidence, management pitch
│   ├── PRODUCT_SPEC.md                identity, problem, audience, journeys, V1 scope, disclaimers
│   ├── QUALITY_AND_TESTING.md         quality gates, test layers, security baseline, perf targets
│   └── V2_FUTURE_ROADMAP.md           biometric integration, dynamic QR, guests, Razorpay — all conditional
└── .claude/
    ├── agents/
    │   ├── senior-architect.md        read-only (Read/Grep/Glob), model: sonnet
    │   ├── backend-engineer.md        Read/Grep/Glob/Edit/Write/Bash, model: sonnet
    │   ├── frontend-engineer-designer.md  Read/Grep/Glob/Edit/Write/Bash, model: sonnet
    │   ├── qa-engineer.md             Read/Grep/Glob/Bash, model: sonnet
    │   └── security-reviewer.md       read-only, model: sonnet
    └── skills/
        ├── feature-workflow/SKILL.md
        ├── debug-systematically/SKILL.md
        ├── review-before-commit/SKILL.md
        └── release-checklist/SKILL.md
```

Not present: `.claude/settings.json`, `.gitignore`, `.env.example`, any prompt templates, any learning documents, any setup guides, any changelog/version marker.

---

## 2. Requirements found in the old kit

### 2.1 Product
| # | Requirement | Source | Old status |
|---|---|---|---|
| P1 | Name BiteCampus; tagline "Your campus. Your food. Your way." | PRODUCT_SPEC, DECISIONS | DECIDED |
| P2 | Campus food ecosystem starting with one BMSCE hostel mess | PRODUCT_SPEC | DECIDED |
| P3 | Brand: premium, youthful, friendly food-tech; purple/orange inspiration; no copying Zomato/Swiggy assets or layouts | PRODUCT_SPEC, frontend agent | DECIDED |
| P4 | Mobile-first, responsive, accessible; loading/error/empty/success states | CLAUDE.md, ARCHITECTURE | DECIDED |
| P5 | No implied BMSCE endorsement; no college logos without permission | CLAUDE.md, PILOT | DECIDED |
| P6 | Outside-college message: "Outside-college access is planned for a future version." | DECIDED | DECIDED |
| P7 | Honest metrics only; targets are not results | PRODUCT_SPEC, QUALITY, PILOT | DECIDED |
| P8 | Feedback capture with identifiable/anonymous disclosure; data minimisation | DOMAIN_RULES, PILOT | DECIDED |
| P9 | Feedback survey prompts (7), evidence list, management proposal outline, launch wording | PILOT | — |

### 2.2 Identity and access
| # | Requirement | Old status |
|---|---|---|
| A1 | Google sign-in; ID token verified server-side (signature, issuer, audience, expiry, verified email) via official library | DECIDED |
| A2 | Allowed college domain(s) configured | DECIDED |
| A3 | First eligible sign-in creates student account | DECIDED |
| A4 | Domain match never grants staff/admin; privileged roles assigned explicitly by an authorised admin | DECIDED |
| A5 | Active-roster validation — source and sync process | PENDING |
| A6 | Role names and permission matrix | PENDING |
| A7 | Default deny; object-level authorisation (student sees only own records) | DECIDED |
| A8 | HTTP-only/Secure/SameSite cookies; CSRF if cookie auth; rate limiting on auth and redemption; CORS allow-list | DECIDED |

### 2.3 Meals, booking, payment, ticket
| # | Requirement | Old status |
|---|---|---|
| M1 | Prices/windows/cutoffs/capacity/eligibility are configuration, not scattered constants | DECIDED |
| M2 | Store UTC; apply policy in `Asia/Kolkata`; meal date is a campus-local date | DECIDED |
| M3 | Booking `PENDING_PAYMENT → CONFIRMED`; explicit failure/cancel states; no client-only confirmation | Proposed |
| M4 | Simulated payment only; label "DEMO / SIMULATED — NO MONEY CHARGED"; never request card/UPI/bank credentials | DECIDED |
| M5 | Duplicate checkout must not create duplicate bookings/tickets (idempotency key or unique constraint) | DECIDED |
| M6 | QR carries only an opaque high-entropy reference; no PII | DECIDED |
| M7 | Redemption validates token, status, date, meal, payment, staff authorisation; atomic, one-time; unique constraint; at most one concurrent success | DECIDED |
| M8 | Manual lookup uses the same validation/redemption service as QR | DECIDED |
| M9 | Audit staff user, timestamp, result; audit sensitive admin actions | DECIDED |
| M10 | No student refund or ticket transfer | WORKING DEFAULT |
| M11 | College cancellation → admin refund or replacement coupon | DEFERRED |
| M12 | Rescheduling before cutoff | Needs confirmation |
| M13 | Hosteller prepaid roster vs localite paid booking | PENDING (open decision #2) |
| M14 | Capacity limits in V1 and their owner | PENDING |

### 2.4 Engineering
Modular monolith; React+TS+Vite+Tailwind; Express+TS; PostgreSQL+Prisma; Zod; Vitest/Supertest/Playwright (all DECIDED). REST JSON (WORKING DEFAULT). Thin routes → service → Prisma; no generic repositories without benefit; migrations only; synthetic seed data only; consistent safe errors; quality gates (typecheck, tests, authz paths, UI states, diff review, migration rollback, docs updated). Deployment provider PENDING. Perf targets (50–100 concurrent, p95 < 300 ms, 80% backend coverage, 15–20 E2E) explicitly **unmeasured targets**.

### 2.5 V2 guardrails (from V2_FUTURE_ROADMAP)
| # | Guardrail |
|---|---|
| G1 | BiteCampus never stores biometric images/templates; matching stays in the college/vendor system |
| G2 | Integration via minimal signed identity events (opaque person ID, device ID, event ID, timestamp, result) |
| G3 | Campus entry ≠ meal redemption; only a designated meal checkpoint + valid entitlement redeems |
| G4 | No scraping/reverse-engineering the college access system |
| G5 | BiteCampus does not control physical gate hardware without formal safety review |
| G6 | Dynamic QR: short-lived, person-bound, purpose-bound, replay-resistant, revocable; screenshots rejected; no fail-open offline |
| G7 | Distinguish campus entry, dining entry, entitlement validation, coupon redemption, food collection |
| G8 | Gateway (Razorpay): server-side orders, signature/webhook verification, idempotent webhooks, reconciliation; merchant KYC owned by the college, never a student's PAN/bank |
| G9 | Non-biometric alternative for people who cannot/do not use biometrics |
| G10 | Do not block a safety-critical gate on a slow BiteCampus DB call |

### 2.6 Claude Code workflow
Inspect before change; no overwrite; ask before architecture changes / destructive commands / schema resets / force push / production migrations / large dependencies; never claim unverified results; scoped diffs; end-of-task report (files, behaviour, commands, limitations, suggested commit); session hand-off file; use subagents only when useful.

---

## 3. What should be preserved
Everything in §2 that is not superseded in §5. Specifically strong and worth keeping verbatim in spirit:
- The DECIDED/WORKING DEFAULT/PENDING/DEFERRED status vocabulary — it is the right mechanism for `OPEN_DECISIONS.md`.
- The non-negotiable invariants in `CLAUDE.md` (server authority, domain ≠ privilege, no credential collection, PII-free QR, Asia/Kolkata boundary, synthetic data).
- All V2 guardrails G1–G10 — they become the *design constraints* for the V1 simulators.
- The "manual lookup uses the same redemption service" rule — generalises directly to "biometric and QR converge on one service".
- Honest-metrics language and the feedback/pilot evidence list.
- The four skills (all sound; need learning-contract additions).
- Read-only posture of `senior-architect` and `security-reviewer`.

## 4. Outdated instructions
| Item | Where | Why outdated |
|---|---|---|
| "I am continuing from a planning conversation in ChatGPT" | START_HERE | Context is now a Claude Project + repository docs |
| 3–4 day launch timeline and Day 0–4 plan | DELIVERY_PLAN | V3 scope (biometric sim, dynamic QR, gate sim, reconciliation, RBAC for 9 roles) cannot fit; superseded by a milestone roadmap that waits on the repo audit |
| V1 = "feedback demo; tickets are not real entitlements" | PRODUCT_SPEC, PILOT, DOMAIN_RULES | V3 target is a *controlled real pilot* (~500 users). See conflict C1 |
| Dynamic QR, biometric, guest passes listed as V2-only | V2_FUTURE_ROADMAP, DECISIONS | V3 moves **simulators** of these into V1 |
| "ticket" terminology | throughout | V3 uses coupon / entitlement / redemption; needs one glossary |
| `model: sonnet` pinned in every agent | .claude/agents | Pins a model family unnecessarily; recommend omitting `model` (inherit) unless there is a cost reason |
| START_HERE asks for a "proposed folder structure" and scaffolding readiness | START_HERE | Assumes a near-empty repo; V3 must assume an existing app with partial implementation |

## 5. Conflicting requirements (old kit vs V3 prompt) — need your decision
| ID | Topic | Old kit | V3 prompt | Recommended resolution |
|---|---|---|---|---|
| **C1** | Pilot nature vs simulated payment | Feedback demo; simulated checkout grants **no** real entitlement | Real controlled pilot **and** simulated payment | Decide how a coupon becomes a real entitlement without real payment: (a) hostellers are already prepaid via mess fees, so "checkout" is a ₹0 confirmation and only localites use the simulated paid path; (b) payment collected offline by the mess and confirmed by staff; (c) pilot coupons remain demo-only. **Must be decided before booking/payment is designed.** |
| **C2** | Lunch window | 1:00–2:30 PM (PENDING, "confirm close") | 1:00–3:30 PM | Adopt 3:30 PM (newer, explicit) — awaiting your approval |
| **C3** | Dinner window | 7:00–9:30 PM | 7:30–9:30 PM | Adopt 7:30 PM — awaiting your approval |
| **C4** | Snacks | DEFERRED, not in first release | 4:30–5:30 PM window listed | Snacks need a **price, cutoff, and bookability** decision; none is given anywhere |
| **C5** | Biometric | V2, real integration only | V1 simulator | No real conflict if the simulator emits the same minimal signed event shape as G2 and stores nothing biometric. Keep G1–G10 as constraints |
| **C6** | Dynamic QR | V2; V1 used a static opaque ticket QR scanned by staff | V1 dynamic QR fallback | Replace the static ticket QR with dynamic QR (static is screenshot-replayable). Decide **QR direction** — see A-3 |
| **C7** | Gate control | BiteCampus must not control physical gates | V1 gate-controller simulator | Compatible: simulator only, behind an interface; physical integration stays V2 behind G5 |
| **C8** | Kitchen/inventory/ration/waste, event guest coupons, import/export, notifications | DEFERRED beyond first release; first in the scope-cut order | "Preserve" in V1 list G | Decide whether list G is **pilot-release scope** or **post-pilot V1.x**. Recommendation: core A–F + RBAC + menu/pricing + audit + feedback in the pilot; the rest in V1.1/V1.2 milestones |
| **C9** | Feature workflow | Explain → implement directly | Explain → **wait for approval** → implement → 11-part post-implementation walkthrough | V3 learning contract wins; skill must gain an explicit approval gate |
| **C10** | Capacity | Open: capacity limits in V1? | "No automatic reduction of hostel food capacity" | Interpretation needed — see A-5 |

## 6. Requirements in the V3 prompt that are **not** in the old kit
The V3 prompt says to preserve these "from the old kit", but they do not appear in the uploaded ZIP. They may exist in your repository, earlier conversations, or a different kit version. They will be recorded with source "V3 prompt" (not "old kit") until the repo audit confirms otherwise:

- Role list: student, hostel, staff, kitchen, meal admin, inventory admin, counter staff, staff manager, director (old kit only says role names are PENDING).
- Complimentary event coupons with authorised approval; guest invitations and coupon claiming (old kit mentions guest coupons only as deferred, and guest passes only in V2).
- Administrative dashboards and notifications (old kit lists notifications only as out-of-scope).
- Public recruiter-facing landing page (absent).
- "No automatic reduction of hostel food capacity" (absent).
- Snacks schedule (old kit defers snacks).
- Biometric simulator failure modes (failed, unavailable, timeout), gate-controller reconciliation, QR token hashing (absent).

## 7. Incomplete areas in the old kit
- No permission matrix; no role definitions.
- No database design beyond a candidate entity list; no state machines with all transitions; no constraint list.
- No API contracts (endpoints, payloads, error envelope, status codes).
- No idempotency design (key scope, retention, response replay).
- No session strategy chosen (cookie session vs JWT) — "select during implementation".
- No error-handling or logging standard; no observability.
- No deployment target, backup/restore procedure, or environment matrix.
- Refund, rescheduling, cancellation, and college-cancellation flows unresolved.
- No data retention / privacy notice text.
- Test strategy lacks how to run concurrency tests against real PostgreSQL and how to isolate test databases.

## 8. Missing documentation (relative to V3 requirements)
Product: FUNCTIONAL_REQUIREMENTS, NON_FUNCTIONAL_REQUIREMENTS, USER_ROLES_AND_PERMISSIONS, USER_FLOWS, ACCEPTANCE_CRITERIA, OPEN_DECISIONS (partially covered by DECISIONS.md).
Technical: SYSTEM_DESIGN, DATABASE_DESIGN, API_CONTRACTS, SECURITY (partial in QUALITY), AUTHENTICATION_AND_AUTHORIZATION, BIOMETRIC_QR_REDEMPTION_DESIGN, DEPLOYMENT, OBSERVABILITY.
Learning: all six (LEARNING_CONTRACT, LEARNING_LOG_TEMPLATE, FEATURE_WALKTHROUGH_TEMPLATE, DEBUGGING_GUIDE, INTERVIEW_PREPARATION, TDR template).
Claude Code: `.claude/settings.json` (permissions, deny rules for `.env`, push, force-push; commit-attribution off), prompt templates, database-reviewer / code-reviewer / learning-mentor agents.
Developer guides: Windows/PowerShell, VS Code, Claude Code, Git workflow, Node/npm, PostgreSQL/Prisma, env vars, local run, migrations/seeding, testing, Chrome DevTools, API debugging, logging, build/deploy. None exist.

## 9. Ambiguities recorded (not decided)
| ID | Ambiguity |
|---|---|
| A-1 | Hosteller vs localite: who books, who pays (simulated), who is auto-entitled? (old open decision #2, still open) |
| A-2 | Is "hostel" a role (hostel warden/office) or a user attribute (student is a hosteller)? |
| A-3 | QR direction: student shows a rotating QR to a gate scanner, or the gate shows a QR the student scans? Changes threat model and hardware assumptions |
| A-4 | Gate-timeout reconciliation policy: after a redemption committed but the gate response was lost, how long may the student retry entry against the *same* redemption, and who can override? |
| A-5 | "No automatic reduction of hostel food capacity": does it mean bookings never lower the kitchen's planned quantity for hostellers, or that capacity is never auto-capped by low bookings? |
| A-6 | Who approves complimentary event coupons (director? staff manager?) and are guests Google-authenticated or claim-by-link? |
| A-7 | Roster source: manual CSV import by an admin, or a college-provided feed? Update frequency? |
| A-8 | Session strategy: server-side session in PostgreSQL vs signed JWT cookie |
| A-9 | Refund / reschedule / cancellation rules for V1 (old kit: no refund, no transfer; rescheduling unconfirmed) |
| A-10 | Hosting provider and budget (old: PENDING) |
| A-11 | Does the recruiter landing page show live data, a demo mode, or static content? |

## 10. Recommended improvements for V3
1. **Single source of truth per topic.** Old kit splits rules across DECISIONS, DOMAIN_RULES, PRODUCT_SPEC, and CLAUDE.md with some duplication (prices appear only in DECISIONS, which is good — keep that pattern: one doc owns each fact, others link).
2. **Convert V2 guardrails into V1 simulator interfaces.** `BiometricVerifier`, `GateController`, `PaymentProvider` interfaces with `Simulated*` implementations; real adapters remain V2.
3. **One redemption service.** Biometric, QR, and manual lookup all call `RedemptionService.redeem(personRef, checkpoint, mealSlot, idempotencyKey)`.
4. **Model gate outcome separately from redemption.** `Redemption` (durable, unique per coupon) + `GateEvent` (attempts, outcome `OPENED | FAILED | UNKNOWN`) + reconciliation job/endpoint.
5. **Learning contract as a gate, not advice.** Feature-workflow skill stops after the plan and requires explicit approval; post-implementation walkthrough template is mandatory.
6. **Claude Code settings.** Add `.claude/settings.json` with deny rules (read `.env*`, `git push`, `git push --force`, `prisma migrate reset`, `rm -rf`) and commit co-author attribution disabled (exact setting name to be verified against current Claude Code docs during Phase Three).
7. **Fewer, sharper agents.** Keep the 5, add database-reviewer, code-reviewer, learning-mentor; drop model pins.
8. **Replace the day-plan with a milestone roadmap** whose content is filled in only after the read-only repo audit.
9. **Windows-first guides** with PowerShell commands only.
10. **A glossary** (coupon, entitlement, redemption, checkpoint, gate event, reconciliation, booking, payment attempt).

---

## 11. What I could and could not inspect
| Item | Status |
|---|---|
| Uploaded V2 kit ZIP | Fully inspected — all 20 files read |
| Local folder `C:\Users\udayr\...\bitecampus` | **Not accessible** from this conversation |
| GitHub `udayramageri09/bitecampus` | **Not inspected** (can be attached on request) |
| Existing `package.json`, schema, migrations, `.env`, code | Not inspected — must be audited by Claude Code locally |
