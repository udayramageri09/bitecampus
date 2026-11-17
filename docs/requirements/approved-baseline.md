# BiteCampus — Approved Product Baseline

**Status:** User-selected baseline for implementation. Technical design details and operational values that are called out as open must be confirmed before the related production workflow ships.

## Product scope and accounts

- V1 is a single BMSCE hostel-mess pilot, designed without speculative multi-college infrastructure.
- Ordinary diners, including outsiders, may use the normal diner flow with a verified personal Google account and buy meal coupons. A user-selected category is for reporting and never grants prepaid hostel access.
- Hostel-resident status is verified using official college email matched against the College IT resident list. It cannot be self-selected.
- Invited event Guests use a separate, event-scoped guest flow. They receive assigned coupons at their registered email; they do not book or pay for those coupons.

## Roles and responsibilities

- **Main Admin:** reports to the principal, has cross-department oversight/signing authority, configures academic-year dates and applicable hostel meal rules in the app without code changes, and receives weekly/monthly department reporting.
- **Meal Admin:** manages meal prices, booking/redemption windows, booking management, guest coupon operations and refunds. May reopen a booking window; overrides must be scoped, reasoned and audited.
- **Inventory Admin:** manages stock additions, usage, waste, adjustments, preparation and inventory reporting. Forecasts are advisory; purchases are not automated.
- **Counter Staff:** scans meal passes. Redemption is server-authoritative, single-use and auditable.
- **Staff Manager:** records kitchen-staff attendance and audited corrections. Payroll, biometric and location tracking are out of scope for the demo.
- **College Student/Diner:** sees meals, purchases/uses eligible coupons, views history and redeems passes.
- **Guest:** sees only invited event coupon(s) and related menu details; cannot book ordinary meals through the guest identity.
- **Recruiter:** accesses an interactive product demo using isolated synthetic data.
- **Hostel Director permission:** distinct from Main Admin permissions; one person may hold both. Hostel Director approval is required before Meal Admin issues an eligible refund.

Accounts with staff/admin roles are provisioned by authorized administrators. Client-side role/category values never authorize access.

## Hostel meal access

- Hostel fees include accommodation and food for the paid academic year.
- Eligible hostel residents do not book or pay per meal. Meal access appears in their account so actual redemption can be counted.
- Entitlement renews with yearly payment and annual roster updates. Main Admin configures academic-year dates and applicable meal rules in the app.
- Do not invent a numeric meal quota; the exact meal scope must match the hostel-fee arrangement.
- College IT supplies the resident list. Use minimum necessary fields and validate the exact columns with College IT.

## Paid meal booking and refunds

- Meal booking capacity is unlimited in V1.
- Meal Admin manages configurable booking/redemption windows and may reopen a specific window. Exact times are configuration values, not hardcoded business rules.
- Price changes apply to new bookings; save the price snapshot on each booking.
- V1 payment is simulated only. Collect no card, bank or UPI credentials.
- Meal Admin issues a refund only after Hostel Director permission and only for an issue caused by hostel management. Record the reason, approval and execution in an append-only audit trail.
- Use Asia/Kolkata for date/time policy and display. Persist timestamps in UTC and convert at application boundaries.

## Guest coupons

- Organizer requests quantity and guest details; an authorized approval is recorded; Meal Admin generates and emails assigned coupons to registered Guest email addresses.
- Guests do not book or pay. They present the assigned QR at the counter.
- Guest coupons remain distinct from paid tickets and prepaid hostel entitlements in authorization, persistence, reporting and audit.
- QR tokens are opaque, single-use, scoped to the applicable meal/date, and redeemed atomically by the server. Offline scans cannot claim final success.
- **Open:** whether the guest email embeds the QR or contains a secure link to display it in the guest flow.

## Recruiter demo

- Provide an interactive product experience with synthetic records, isolated from production.
- No real payments, outbound emails or destructive administrative actions.
- Reset demo data and expire access after seven days.
- **To finalize during demo setup:** whether requests need manual approval and which screens the demo exposes.

## Inventory, attendance, reports and privacy

- Inventory uses an append-only movement ledger with units, reasons for usage/waste/corrections and advisory forecasting. No automatic purchasing.
- Staff Manager records attendance and audited corrections. No biometrics/location tracking; payroll is out of scope unless separately approved.
- Reports separate booked, paid, cancelled, eligible/prepaid, redeemed, guest-coupon, inventory-used, waste and attendance metrics. Main Admin sees cross-department reporting; each department sees its scope. Exports are restricted and audited.
- Minimize personal data and restrict exports/logs. Final retention periods follow college policy.
- CSV/XLSX imports require preview, validation, duplicate checks and explicit commit. Email is the initial notification channel; no SMS in V1.

## UI and reliability

- Brand direction: orange, white and deep charcoal; original simple food mark; system font; responsive, accessible screens; reduced-motion support.
- Backups are encrypted and managed; a restore drill, staging/production separation and outage/rollback procedures are required before pilot. Suggested starting RPO/RTO are one hour/four hours, pending college validation.

## Remaining implementation decisions

1. Guest email delivery format: attached/embedded QR, secure link, or both.
2. Concrete examples of hostel-management-caused failures that qualify for refunds.
3. Validate the roster columns and exact meal schedule against College IT and the hostel-fee arrangement.
4. Populate Main Admin’s academic-year/meal settings and Meal Admin’s cutoff/price settings in the UI.
5. Confirm hosting region/vendor and college validation of recovery targets before deployment.
