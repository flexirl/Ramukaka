# Ramu Kaka — Development Roadmap

| Field | Value |
|---|---|
| Status | **PROPOSED – awaiting approval** |
| Confirmed input | Phase 1 / 2 / 3 scope `[PDF §8]` |
| Timeline | **PENDING APPROVAL (PA-09)**. No dates are committed in this document |
| Last updated | 2026-10-30 (audit corrections: dependency order fixed) |

> The PDF defines three business **phases**. This roadmap divides **Phase 1 (Water Delivery MVP)** into engineering **levels**.
> **Approval gate rule:** work on a level starts only after the user explicitly approves the previous level.

---

## 1. Levels Overview

"Blocking decisions" are PA items that must be decided **before the level starts**.

| Level | Name | Main output | PDF phase | Blocking decisions |
|---|---|---|---|---|
| **0** | Project Planning & Documentation | This `docs/` set, CLAUDE.md, README.md, CHANGELOG.md | — | — |
| **1** | Decisions & UX Design | Approved PA items; Google Stitch wireframes and designs | Phase 1 | PA-09, PA-11, PA-12, PA-38, PA-59 |
| **2** | Foundation & Database | Monorepo setup, Supabase project(s), CI, migrations, RLS, seed data | Phase 1 | **PA-10**, PA-13, PA-31, PA-32, PA-39, PA-41, PA-42, PA-54 |
| **3** | Authentication & Roles | OTP login for all roles, role/status checks, **seed and setup scripts** for stores, managers and DPs (stopgap until Level 7) | Phase 1 | PA-14, PA-15, PA-22 |
| **4** | Customer App – Manual Ordering + Push | React Native setup; customer app: language, addresses, catalog, quote → confirm → order, tracking, notification inbox; **push notification infrastructure** | Phase 1 | PA-01, PA-02, PA-05, PA-16, PA-19, PA-21, PA-36, PA-49, PA-53 |
| **5** | Store Manager Dashboard | Store open/close, catalog and stock, orders board, manual assign, surge, reports | Phase 1 | PA-33, PA-50 |
| **6** | Delivery Partner App | Duty and attendance photo, delivery requests (push), atomic accept/reject, status flow, COD entry, issue reporting | Phase 1 | PA-03, PA-06, PA-17, PA-24, PA-40, PA-56 |
| **7** | Super Admin Dashboard | Stores, managers, DPs, customers, settings and limits, permissions, platform reports (replaces the Level 3 setup scripts) | Phase 1 | PA-44, PA-51 |
| **8** | Wallet, Offers & COD Reconciliation | Ledger, admin credits, coupons, basic cashback, **promotional** notifications, COD reconciliation | Phase 1 | PA-04, PA-08, PA-27…30, PA-52, PA-58 |
| **9** | AI Voice Assistant | Gemini server agent with the confirmation state machine, voice UI, 3 languages, manual fallback, AI usage and cost tracking | Phase 1 | PA-07, PA-23, PA-34 (audio), PA-35, PA-57 |
| **10** | Reviews, Complaints & Analytics | Ratings, complaint and feedback handling, KPI dashboards (KPI-01…09) | Phase 1 | PA-25, PA-26, PA-48, PA-55 |
| **11** | Hardening, UAT & Pilot Launch | Security review, performance, device testing, UAT in Jamui, Play Store release | Phase 1 | PA-34, PA-43, PA-45, PA-47 |
| 12+ | Phase 2 – Optimization | Better voice, analytics, advanced offers, online payments (if required), better attendance verification and reconciliation | Phase 2 | — |
| 13+ | Phase 3 – Expansion | Dairy, groceries; new districts, stores, suppliers; automation and BI | Phase 3 | — |

### Dependency notes (fixed in the audit)
- **PA-10 blocks Level 2**, because the dashboard platform decides the repository layout (ADR-06).
- **Push notifications are built in Level 4.** Customer status notifications need them in Level 4, and delivery requests need them in Level 6. Level 8 adds only promotional notifications.
- **Level 3 has no app yet.** Its exit is proven by API-level role tests and seed/setup scripts, not by app screens.
- **Level 4 uses seed data** for stores and products. The manager catalog UI arrives in Level 5, and admin store creation in Level 7.
- **Level 4 exit is order placement**, not full delivery. The full lifecycle is the Level 6 exit.
- **The quote engine (Level 4) needs PA-05 and PA-19** (surge and delivery charges). Until surge is configured, it is zero.
- **Store open/close exists from Level 4 in data** (seeded). The manager UI for it arrives in Level 5.

### Why the AI assistant is Level 9 (and not earlier)
The PDF requires the AI to use real prices, stock and delivery rules (FR-AI-08) and the same confirmation rules (BR-04). These depend on the order engine built in Levels 2–4. Building the AI **on top of** the quote and order API, with the server-owned confirmation step (DR-10), ensures voice ordering cannot skip business rules.

**Optional parallel track (RECOMMENDATION):** start an **AI voice spike** during Level 4–5. It would measure Gemini's Hindi and Bhojpuri recognition quality, the yes/no confirmation check accuracy, latency and cost per turn, so PA-07 and PA-23 can be decided early. This needs your approval.

## 2. Level Details

### Level 0 — Project Planning & Documentation (current)
- **Deliverables:** PRD, SRS, Architecture, DB schema (conceptual), API contract, UI/UX, User flows, Business rules, Testing strategy, Roadmap, CLAUDE.md, README.md, CHANGELOG.md.
- **Not included:** application code, DB tables or migrations, React Native setup.
- **Exit:** user approves the documents and answers (or schedules) the PA items.

### Level 1 — Decisions & UX Design
- Decide all PA items that block Levels 2–4 (at least PA-10, PA-13, PA-31, PA-32, PA-39, PA-41, PA-42, PA-54, PA-14, PA-15, PA-22, and the Level 4 list).
- Start SMS DLT registration early (PA-22). It can take weeks.
- Wireframe and design all screens in Google Stitch (UI_UX_DESIGN §3).
- Finalize branding, translation approach and native-speaker reviewers.
- **Exit:** signed-off designs and an updated BUSINESS_RULES.md.

### Level 2 — Foundation & Database
- Monorepo (layout per PA-10), linting, TypeScript config, CI (GitHub Actions), secret scanning.
- Supabase local, staging and production projects. Secrets management (NFR-SEC-11).
- Migrations for the approved schema, RLS policies (including account-status checks), Postgres functions for atomic operations.
- RLS and business-rule test suite (TESTING_STRATEGY §3).
- **Exit:** schema plus RLS tests green in CI.

### Level 3 — Authentication & Roles
- Phone OTP with the SMS provider for all roles. Role and status resolution. Protected profile fields.
- Seed and setup scripts to create stores, managers and DPs. This is a stopgap that the Level 7 admin UI replaces.
- **Exit:** API-level tests prove each role can log in by OTP, is limited to its own data, and is blocked immediately when suspended. No app UI is required at this level.

### Level 4 — Customer App (Manual Ordering) + Push
- React Native project initialized **at this level**.
- Language picker, i18n framework, addresses, serviceability, catalog (seed data), quote → confirm → order, tracking, history, notification inbox.
- Push notification infrastructure (device tokens, sending function) and customer status notifications.
- **Exit:** a customer can place a COD order for a water jar against staging. The order is stored as `placed`, the customer is notified, and status changes made with test tooling appear live in the app.

### Level 5 — Store Manager Dashboard
- Store open/close, categories and products (server functions, audit log), stock, orders board, manual assign, surge (within limits), basic reports.

### Level 6 — Delivery Partner App
- Duty toggle and attendance photo, delivery requests by push, atomic accept/reject, status updates, COD entry, failure reporting, history.
- **Exit:** the concurrency test for BR-03 passes. The **full order lifecycle** (`placed → delivered`, plus `failed`) works end to end with real devices.

### Level 7 — Super Admin Dashboard
- Store, manager, DP and customer management; store settings and limits; manager permissions; platform reports; partner attendance view. Replaces the Level 3 setup scripts.

### Level 8 — Wallet, Offers & COD Reconciliation
- Wallet ledger (serialized writes), admin credits, coupons (platform and store), basic cashback (crediting per PA-52), promotional notifications, COD reconciliation screens.

### Level 9 — AI Voice Assistant
- Server-side Gemini agent with read/quote-only tools, server-owned confirmation state machine and yes/no check, voice UI, three languages, manual fallback, rate limits, AI usage and cost logging, and the admin report.
- **Exit:** the AI evaluation set meets the PA-07 targets. Confirmation compliance and price fidelity are 100%. False confirmation rate is 0% (TESTING_STRATEGY §4.2).

### Level 10 — Reviews, Complaints & Analytics
- Product reviews, complaint and feedback flow, KPI dashboards for manager and admin (definitions PA-55).

### Level 11 — Hardening, UAT & Pilot Launch
- Security review, performance and device testing, privacy compliance, Play Store listing.
- UAT with real customers, DPs and the store manager in Jamui. Pilot go-live.

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
| Gemini cost per conversation | Operating cost | Usage logging from day 1; short prompts; rate limits; caps (PA-35) |
| SMS OTP delivery / DLT registration delays | Login blocked | Start DLT registration in Level 1 (PA-22) |
| Weak connectivity | Failed orders and status updates | Idempotent APIs, retries, offline banners |
| Undecided business rules | Rework | Approval gates; PA register |
| Low digital literacy | Low adoption | Usability testing with real users in Level 1 and 11 |
| Play Store review (permissions: camera, location) | Release delay | Request only the permissions needed, with clear explanations |
