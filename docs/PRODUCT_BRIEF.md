# PRODUCT_BRIEF — BiteCampus

> One-page summary. Details: `PRD.md`. Statuses and decision IDs: `OPEN_DECISIONS.md`.

**BiteCampus** — *Your campus. Your food. Your way.* (D-001)

## Problem
In a hostel mess, meal information, booking, entitlement checks, entry, attendance and feedback are handled manually or in separate places. Students lack a clear view of what they are entitled to. Staff have no auditable record of who ate, and planners lack reliable operational data.

## Who it is for (initial pilot)
- **Hostel residents** at one BMSCE hostel mess, approximately 500 users (D-002). Their meals are prepaid through hostel mess fees (D-008).
- **Mess and administrative staff**: Main Admin, Meal Admin, Inventory Admin and Counter Staff in the pilot (D-043).

## What the pilot delivers
1. Google sign-in restricted to the college domain and the active hostel roster.
2. A student dashboard listing meals, menus and list prices.
3. **Automatic daily booking**: every eligible hosteller is booked for Breakfast, Lunch and Dinner each day at ₹0 (covered by the prepaid mess fee), with a digital meal coupon. There is no manual booking. A hosteller can skip a meal before its cutoff if the college enables opt-out.
4. Entry at the mess entrance: the student scans a **rotating QR displayed by the entrance terminal** using the BiteCampus app, or uses **staff-assisted recovery**. A **biometric verification simulator** demonstrates the future biometric path in development, testing, training and demonstrations; it is not used in the live pilot.
5. Server-side entitlement checks with **exactly-once durable redemption**.
6. A **gate-controller simulator** with reconciliation when the gate response is uncertain.
7. Booking and redemption history, staff roles, menu/pricing admin, audit logs, essential operational dashboards and feedback.

## What it is not (V1)
- Not a payment product. Simulated checkout exists only for demonstration (D-009). There is no real gateway (D-011).
- Not a biometric system. No biometric data is collected or stored (D-018, D-029).
- Not a gate controller. Physical gates are simulated (D-023).
- Not an official BMSCE service until written authorisation exists (D-030).

## Why it is technically interesting
- Idempotent automatic booking that stays correct under retries, overlapping runs and membership changes (D-035).
- Several identity-verification methods converge on one authoritative redemption service (D-021).
- Exactly-once durable effects under concurrency, proven against real PostgreSQL (D-022).
- Separation of identity, entitlement, redemption and physical entry, with reconciliation for uncertain hardware outcomes (D-023).
- Integration boundaries (payment, biometric, gate) designed so simulators can be replaced by authorised integrations later.

## Release phases
| Phase | Content |
|---|---|
| **Pilot (V1.0)** | The 17 pilot items (D-025) |
| **V1.1 / V1.2** | Kitchen and inventory, ration/waste reporting, events and organisers, complimentary and guest coupons, advanced import/export, extended notifications, extra reporting (D-026) |
| **V2 (conditional)** | Real payment gateway, real biometric integration, physical gate integration, subject to college authorisation (F-009–F-011) |
| **Undecided** | Recruiter-facing landing page (Q-16) |

## Success signals (measured, not assumed — D-032)
Active eligible sign-ins; auto-booked vs opted-out vs redeemed per meal service; successful redemptions; rejected duplicate attempts; QR-fallback usage rate; gate reconciliation count; feedback responses; defects found and fixed.
