# Ramu Kaka — Development Roadmap

| Field | Value |
|---|---|
| Status | **PROPOSED – awaiting approval** |
| Confirmed input | Phase 1 / 2 / 3 scope `[PDF §8]` |
| Timeline | **PENDING APPROVAL (PA-09)**. No dates are committed in this document |

> The PDF defines three business **phases**. This roadmap divides **Phase 1 (Water Delivery MVP)** into engineering **levels**.
> **Approval gate rule:** work on a level starts only after the user explicitly approves the previous level.

---

## 1. Levels Overview

| Level | Name | Main output | PDF phase | Blocking decisions |
|---|---|---|---|---|
| **0** | Project Planning & Documentation | This `docs/` set, CLAUDE.md, README.md | — | — |
| **1** | Decisions & UX Design | Approved PA items; Google Stitch wireframes and designs | Phase 1 | PA-09, PA-10, PA-11, PA-12, PA-38 |
| **2** | Foundation & Database | Monorepo setup, Supabase project(s), CI, migrations, RLS, seed data | Phase 1 | PA-13, PA-31, PA-32, PA-39, PA-41, PA-42 |
| **3** | Authentication & Roles | OTP login, role routing, admin creating stores and managers | Phase 1 | PA-14, PA-15, PA-22 |
| **4** | Customer App – Manual Ordering | RN customer app: language, addresses, catalog, quote → confirm → order, tracking, COD | Phase 1 | PA-01, PA-02, PA-16, PA-19, PA-36 |
| **5** | Store Manager Dashboard | Store open/close, catalog and stock, orders board, manual assign, reports | Phase 1 | PA-05, PA-10, PA-33 |
| **6** | Delivery Partner App | Duty and attendance photo, offers, atomic accept, status flow, COD entry, issue reporting | Phase 1 | PA-03, PA-06, PA-17, PA-24, PA-40 |
| **7** | Super Admin Dashboard | Stores, managers, DPs, customers, settings and limits, platform reports | Phase 1 | PA-44 |
| **8** | Wallet, Offers & Notifications | Ledger, admin credits, coupons, basic cashback, push notifications, COD reconciliation | Phase 1 | PA-04, PA-08, PA-21, PA-27…30 |
| **9** | AI Voice Assistant | Gemini server agent, voice UI, 3 languages, manual fallback, AI usage and cost tracking | Phase 1 | PA-07, PA-23, PA-35 |
| **10** | Reviews, Complaints & Analytics | Ratings, complaint handling, KPI dashboards (KPI-01…09) | Phase 1 | PA-25, PA-26, PA-48 |
| **11** | Hardening, UAT & Pilot Launch | Security review, performance, device testing, UAT in Jamui, Play Store release | Phase 1 | PA-34, PA-43, PA-45 |
| 12+ | Phase 2 – Optimization | Better voice, analytics, advanced offers, online payments (if required), better attendance verification and reconciliation | Phase 2 | — |
| 13+ | Phase 3 – Expansion | Dairy, groceries; new districts, stores, suppliers; automation and BI | Phase 3 | — |

### Why the AI assistant is Level 9 (and not earlier)
The PDF requires the AI to use real prices, stock and delivery rules (FR-AI-08) and the same confirmation rules (BR-04). These depend on the order engine built in Levels 2–4. Building the AI **on top of** the quote and order API ensures voice ordering cannot skip business rules.

**Optional parallel track (RECOMMENDATION):** start an **AI voice spike** during Level 4–5. It would measure Gemini's Hindi and Bhojpuri recognition quality, latency and cost per turn, so PA-07 and PA-23 can be decided early. This needs your approval.

## 2. Level Details

### Level 0 — Project Planning & Documentation (current)
- **Deliverables:** PRD, SRS, Architecture, DB schema (conceptual), API contract, UI/UX, User flows, Business rules, Testing strategy, Roadmap, CLAUDE.md, README.md.
- **Not included:** application code, DB tables or migrations, React Native setup.
- **Exit:** user approves the documents and answers (or schedules) the PA items.

### Level 1 — Decisions & UX Design
- Decide all PA items that block Levels 2–4.
- Wireframe and design all screens in Google Stitch (UI_UX_DESIGN §3).
- Finalize branding, translation approach and native-speaker reviewers.
- **Exit:** signed-off designs and an updated BUSINESS_RULES.md.

### Level 2 — Foundation & Database
- Monorepo, linting, TypeScript config, CI (GitHub Actions).
- Supabase local, staging and production projects. Secrets management.
- Migrations for the approved schema, RLS policies, Postgres functions for atomic operations.
- RLS and business-rule test suite (TESTING_STRATEGY §3).
- **Exit:** schema plus RLS tests green in CI.

### Level 3 — Authentication & Roles
- Phone OTP with the SMS provider. Role resolution and panel routing.
- Minimal admin tooling to create stores, managers and DPs.
- **Exit:** each role can log in and is limited to its own data.

### Level 4 — Customer App (Manual Ordering)
- React Native project initialized **at this level**.
- Language picker, i18n framework, addresses, serviceability, catalog, quote → confirm → order, tracking, history.
- **Exit:** a customer can order a water jar by COD end to end against staging.

### Level 5 — Store Manager Dashboard
- Store open/close, categories and products, stock, orders board, manual assign, surge (within limits), basic reports.

### Level 6 — Delivery Partner App
- Duty toggle and attendance photo, offer notifications, atomic accept/reject, status updates, COD entry, failure reporting, history.
- **Exit:** the concurrency test for BR-03 passes. A full order lifecycle works with real devices.

### Level 7 — Super Admin Dashboard
- Store, manager, DP and customer management; store settings and limits; manager permissions; platform reports.

### Level 8 — Wallet, Offers & Notifications
- Wallet ledger, admin credits, coupons (platform and store), basic cashback, push notifications, COD reconciliation screens.

### Level 9 — AI Voice Assistant
- Server-side Gemini agent with tools, voice UI, three languages, manual fallback, AI usage and cost logging and the admin report.
- **Exit:** the AI evaluation set meets the PA-07 targets. Confirmation and price fidelity are 100%.

### Level 10 — Reviews, Complaints & Analytics
- Product reviews, complaint flow, KPI dashboards for manager and admin.

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
| Gemini cost per conversation | Operating cost | Usage logging from day 1; short prompts; caps (PA-35) |
| SMS OTP delivery / DLT registration delays | Login blocked | Start DLT registration in Level 1 (PA-22) |
| Weak connectivity | Failed orders and status updates | Idempotent APIs, retries, offline banners |
| Undecided business rules | Rework | Approval gates; PA register |
| Low digital literacy | Low adoption | Usability testing with real users in Level 1 and 11 |
| Play Store review (permissions: camera, location) | Release delay | Request only the permissions needed, with clear explanations |
