# Ramu Kaka — Development Roadmap

| Field | Value |
|---|---|
| Status | **PROPOSED – awaiting approval** |
| Confirmed input | Phase 1 / 2 / 3 scope `[PDF §8]`; single mobile app decision (BR-27, BR-28) |
| Timeline | **PENDING APPROVAL (PA-09)**. No dates are committed in this document |
| Last updated | 2026-10-30 (single mobile app decision) |

> The PDF defines three business **phases**. This roadmap divides **Phase 1 (Water Delivery MVP)** into engineering **levels**.
> Everything is built inside **one React Native app** with four panels (BR-27). Each level adds or completes a panel in that same app.
> **Approval gate rule:** work on a level starts only after the user explicitly approves the previous level.

---

## 1. Levels Overview

"Blocking decisions" are PA items that must be decided **before the level starts**.

| Level | Name | Main output | PDF phase | Blocking decisions |
|---|---|---|---|---|
| **0** | Project Planning & Documentation | This `docs/` set, PROJECT_OVERVIEW, CLAUDE.md, README.md, CHANGELOG.md | — | — |
| **1** | Decisions & UX Design | Approved PA items; Google Stitch designs for the app shell and all four panels (phone screens) | Phase 1 | PA-09, PA-12, PA-38, PA-59 |
| **2** | Foundation & Database | Repository setup, Supabase project(s), CI, migrations, RLS, seed data | Phase 1 | PA-13, PA-31, PA-32, PA-39, PA-41, PA-42, PA-54, PA-60 |
| **3** | App Shell, Login & Role Routing | **React Native app initialized**; language picker, OTP login, role check, automatic routing to four (empty) panels, status screens, setup scripts for staff roles and the first Super Admin | Phase 1 | PA-14, PA-15, PA-22 |
| **4** | Customer Panel – Manual Ordering + Push | Addresses, catalog, quote → confirm → order, tracking, notification inbox; **push notification infrastructure** | Phase 1 | PA-01, PA-02, PA-05, PA-16, PA-19, PA-21, PA-36, PA-49, PA-53 |
| **5** | Store Manager Panel | Store open/close, catalog and stock, orders board, manual assign, surge, reports | Phase 1 | PA-33, PA-50, PA-61 |
| **6** | Delivery Partner Panel | Duty and attendance photo, delivery requests (push), atomic accept/reject, status flow, COD entry, issue reporting | Phase 1 | PA-03, PA-06, PA-17, PA-24, PA-40, PA-56 |
| **7** | Super Admin Panel | Stores, managers, DPs, customers, role assignment, settings and limits, permissions, platform reports (replaces the Level 3 setup scripts) | Phase 1 | PA-44, PA-51, PA-62 |
| **8** | Wallet, Offers & COD Reconciliation | Ledger, admin credits, coupons, basic cashback, **promotional** notifications, COD reconciliation | Phase 1 | PA-04, PA-08, PA-27…30, PA-52, PA-58 |
| **9** | AI Voice Assistant (Customer panel) | Gemini server agent with the confirmation state machine, voice UI, 3 languages, manual fallback, AI usage and cost tracking | Phase 1 | PA-07, PA-23, PA-34 (audio), PA-35, PA-57 |
| **10** | Reviews, Complaints & Analytics | Ratings, complaint and feedback handling, KPI screens in the Manager and Admin panels (KPI-01…09) | Phase 1 | PA-25, PA-26, PA-48, PA-55 |
| **11** | Hardening, UAT & Pilot Launch | Security review, performance, device testing, UAT in Jamui, Play Store release of the single app | Phase 1 | PA-34, PA-43, PA-45, PA-47 |
| 12+ | Phase 2 – Optimization | Better voice, analytics, advanced offers, online payments (if required), better attendance verification and reconciliation | Phase 2 | — |
| 13+ | Phase 3 – Expansion | Dairy, groceries; new districts, stores, suppliers; automation and BI | Phase 3 | — |

### Dependency notes
- **PA-10 and PA-11 are decided** (BR-27): one React Native app, no web dashboard. The repository has one app folder (SYSTEM_ARCHITECTURE §6).
- **The app is initialized in Level 3**, not Level 4. Login and role routing are shared by every panel, so they come first. Level 3 can then be tested in the real app, with placeholder panels.
- **PA-60 (one or more roles per account) blocks Level 2**, because it decides how `user_roles` and the router are built.
- **Push notifications are built in Level 4.** Customer status notifications need them in Level 4, and delivery requests need them in Level 6. Level 8 adds only promotional notifications.
- **Level 4 uses seed data** for stores and products. The Manager catalog screens arrive in Level 5, and admin store creation in Level 7.
- **Level 4 exit is order placement**, not full delivery. The full lifecycle is the Level 6 exit.
- **The quote engine (Level 4) needs PA-05 and PA-19.** Until surge is configured, it is zero.
- **PA-61 (staff app lock) blocks Level 5**, the first staff panel.
- **Every release ships all four panels.** From Level 4 onward, each level's exit includes a release regression test of every panel built so far (TESTING_STRATEGY §2).

### Why the AI assistant is Level 9 (and not earlier)
The PDF requires the AI to use real prices, stock and delivery rules (FR-AI-08) and the same confirmation rules (BR-04). These depend on the order engine built in Levels 2–4. Building the AI **on top of** the quote and order API, with the server-owned confirmation step (DR-10), ensures voice ordering cannot skip business rules.

**Optional parallel track (RECOMMENDATION):** start an **AI voice spike** during Level 4–5. It would measure Gemini's Hindi and Bhojpuri recognition quality, the yes/no confirmation check accuracy, latency and cost per turn, so PA-07 and PA-23 can be decided early. This needs your approval.

## 2. Level Details

### Level 0 — Project Planning & Documentation (current)
- **Deliverables:** PRD, SRS, Architecture, DB schema (conceptual), API contract, UI/UX, User flows, Business rules, Testing strategy, Roadmap, Project Overview, CLAUDE.md, README.md, CHANGELOG.md.
- **Not included:** application code, DB tables or migrations, React Native setup.
- **Exit:** user approves the documents and answers (or schedules) the PA items.

### Level 1 — Decisions & UX Design
- Decide all PA items that block Levels 2–4 (at least PA-13, PA-31, PA-32, PA-39, PA-41, PA-42, PA-54, PA-60, PA-14, PA-15, PA-22, and the Level 4 list).
- Decide **PA-12 (iOS)** early. With no web dashboard, it decides which phones managers and the owner can use.
- Start SMS DLT registration early (PA-22). It can take weeks.
- Design the shared app shell and all four panels in Google Stitch, with Manager and Admin screens designed for phones (UI_UX_DESIGN §3).
- Finalize branding, translation approach and native-speaker reviewers.
- **Exit:** signed-off designs and an updated BUSINESS_RULES.md.

### Level 2 — Foundation & Database
- Repository with one app folder and the `supabase/` folder; linting, TypeScript config, CI (GitHub Actions), secret scanning.
- Confirm the React Native framework choice (*recommendation:* Expo, which still produces an Android Studio project).
- Supabase local, staging and production projects. Secrets management (NFR-SEC-11).
- Migrations for the approved schema, RLS policies (role, account status, store scope), Postgres functions for atomic operations.
- RLS and business-rule test suite (TESTING_STRATEGY §3), including the role-isolation sweep.
- **Exit:** schema plus RLS tests green in CI.

### Level 3 — App Shell, Login & Role Routing
- React Native project initialized **at this level**.
- Language picker, i18n framework, phone OTP login for all roles, `get-my-access`, automatic routing to the four panels (placeholder home screens), suspended / waiting-for-approval / no-access screens, role re-checks, logout clearing cached data.
- Permission-request framework (explanation sheet, per-role rules, no background location).
- Server-side setup scripts to create the first Super Admin and to pre-register staff roles. This is a stopgap that the Level 7 Super Admin panel replaces.
- **Exit:** with test accounts, each role logs in through the app and lands in its own panel. API tests prove no role can use another role's endpoints and that suspension takes effect immediately.

### Level 4 — Customer Panel (Manual Ordering) + Push
- Addresses, serviceability, catalog (seed data), quote → confirm → order, tracking, history, notification inbox.
- Push notification infrastructure (device tokens, sending function) and customer status notifications.
- **Exit:** a customer can place a COD order for a water jar against staging. The order is stored as `placed`, the customer is notified, and status changes made with test tooling appear live in the app.

### Level 5 — Store Manager Panel
- Store open/close, categories and products (server functions, audit log), stock, orders board, manual assign, surge (within limits), basic reports, all on phone screens. Staff device protection per PA-61.

### Level 6 — Delivery Partner Panel
- Duty toggle and attendance photo, delivery requests by push, atomic accept/reject, status updates, COD entry, failure reporting, history.
- **Exit:** the concurrency test for BR-03 passes. The **full order lifecycle** (`placed → delivered`, plus `failed`) works end to end on real devices, with customer, DP and manager logged in on different phones running the same app.

### Level 7 — Super Admin Panel
- Store, manager, DP and customer management; role assignment (pre-register staff phone numbers); store settings and limits; manager permissions; platform reports; partner attendance view. Replaces the Level 3 setup scripts (except the first Super Admin bootstrap).

### Level 8 — Wallet, Offers & COD Reconciliation
- Wallet ledger (serialized writes), admin credits, coupons (platform and store), basic cashback (crediting per PA-52), promotional notifications, COD reconciliation screens.

### Level 9 — AI Voice Assistant
- Server-side Gemini agent with read/quote-only tools, server-owned confirmation state machine and yes/no check, voice UI in the Customer panel, three languages, manual fallback, rate limits, AI usage and cost logging, and the Super Admin report.
- **Exit:** the AI evaluation set meets the PA-07 targets. Confirmation compliance and price fidelity are 100%. False confirmation rate is 0% (TESTING_STRATEGY §4.2).

### Level 10 — Reviews, Complaints & Analytics
- Product reviews, complaint and feedback flow, KPI screens for the Manager and Admin panels (definitions PA-55; export PA-62).

### Level 11 — Hardening, UAT & Pilot Launch
- Security review (including the "app is never trusted" sweep), performance and device testing, privacy compliance, Play Store listing for the single app, permission declarations.
- UAT with real customers, DPs, the store manager and the owner in Jamui. Pilot go-live.

## 3. Phase 1 MVP Definition (pending PA-09)

PDF Phase 1 includes all Levels 1–11. If the timeline needs to be shorter, these are candidates to **defer** (your decision):
- Bhojpuri voice (keep the Bhojpuri UI) → if pilot recognition quality is poor (PA-07).
- Store-level coupons (keep platform coupons).
- Advanced performance comparisons (keep basic reports).

## 4. Dependencies & Risks

| Risk | Impact | Mitigation (RECOMMENDATION) |
|---|---|---|
| Bhojpuri speech recognition quality | Core feature weak for the target users | Early AI spike; manual fallback; text and voice together |
| AI misreads a confirmation | Unwanted orders | Server-owned confirmation (DR-10), yes/no check, tap alternative, 0% false-confirmation gate |
| One app for all roles: a bug in one panel affects everyone | Broken releases | Release regression of every panel; staged rollouts; over-the-air JS updates if Expo is chosen |
| Staff screens visible inside the app's code on every phone | Attackers discover admin endpoints | Backend authorization on every call (NFR-SEC-14); no secrets in the app |
| Manager/Admin work only on phones | Harder reporting and data entry | Phone-first design (UXR-14); export decision PA-62 |
| Staff without Android phones | Cannot use the system | Decide PA-12 (iOS) in Level 1 |
| Gemini cost per conversation | Operating cost | Usage logging from day 1; short prompts; rate limits; caps (PA-35) |
| SMS OTP delivery / DLT registration delays | Login blocked | Start DLT registration in Level 1 (PA-22) |
| Weak connectivity | Failed orders and status updates | Idempotent APIs, retries, offline banners |
| Undecided business rules | Rework | Approval gates; PA register |
| Low digital literacy | Low adoption | Usability testing with real users in Level 1 and 11 |
| Play Store review (camera, location, microphone in one app) | Release delay | Foreground permissions only, no background location (DR-13), clear explanations |
