# Changelog

This file records changes to the Ramu Kaka documentation and business rules.
Format: newest first. IDs refer to [docs/BUSINESS_RULES.md](docs/BUSINESS_RULES.md) and [docs/SRS.md](docs/SRS.md).

## [0.3.0] — 2026-10-30 — Level 0: Single mobile app architecture

Decision by the project owner (Priya): Ramu Kaka is **one React Native mobile app** containing all four role-based panels. There are no separate apps and no web dashboard.

### Decisions recorded
- **PA-10 DECIDED**: the Manager and Admin panels live inside the single app; no web dashboard.
- **PA-11 DECIDED**: one app for all roles, replacing the earlier "two apps + web dashboard" recommendation.
- New approved rules: **BR-27** (one app, four panels, shared codebase, login and backend) and **BR-28** (backend-assigned roles, automatic routing, no self role change, backend enforcement, separate panel navigation, role-based device permissions).
- New derived rules: **DR-12** (self-registration = customer only; staff roles only from the Super Admin) and **DR-13** (no background location).
- New label **APPROVED DECISION** for owner decisions, with a decision log in BUSINESS_RULES §4.0.

### New pending decisions
- **PA-60** more than one role per account · **PA-61** app lock / auto-logout for staff panels · **PA-62** report export (no web dashboard).
- PA-12 (iOS) marked more important: managers and the owner can only use the app. PA-47 and PA-57 reworded for panels.

### Documents updated
- **SYSTEM_ARCHITECTURE**: one app with an app shell and four panels (diagram, components). ADR-06/07 decided. New ADR-08 (login, role routing, panel isolation) and ADR-09 (device permissions per role). New §5 "Technical limitations and security implications" (L1–L11). Repository layout is now one app folder with per-panel folders.
- **SRS**: new §3.0 App Shell requirements FR-APP-01…08. Panels renamed (Customer/Delivery Partner/Store Manager/Super Admin panel). New NFR-SEC-14 (app never trusted), NFR-SEC-15 (staff device protection), NFR-SEC-16 (least permissions), NFR-PERF-06 (lazy-loaded panels).
- **PRD**: note that the PDF's "apps" and "dashboards" are panels of one app. No PDF requirement changed.
- **DATABASE_SCHEMA**: `user_roles` rules (read-only for users, customer on self-registration, staff roles only via Super Admin).
- **API_DOCUMENTATION**: one client; endpoints grouped by panel with allowed roles; new `get-my-access` and `set-user-role` (S15); compatibility note for older app versions.
- **UI_UX_DESIGN**: new shared app-shell screens (SH-S01…S09); panels renamed; UXR-13…15 (panel navigation, phone-first staff screens, permission explanations).
- **USER_FLOWS**: new UF-00 login and role-based routing.
- **TESTING_STRATEGY**: web dashboard tests removed; all-panel E2E and release regression; tests for routing, isolation, role assignment, "app never trusted" sweep, permissions.
- **DEVELOPMENT_ROADMAP**: React Native app now initialized in **Level 3** (app shell, login, role routing). Levels 4–7 add the Customer, Store Manager, Delivery Partner and Super Admin panels to the same app. Blocking lists updated (PA-60 → Level 2, PA-61 → Level 5, PA-62 → Level 7).
- **CLAUDE.md, README.md**: one-app rules and description.
- **New: docs/PROJECT_OVERVIEW.md**: plain-language explanation of the whole project.

## [0.2.0] — 2026-10-30 — Level 0: Documentation audit corrections

The user approved all 11 corrections from the documentation quality audit. No PDF requirement was removed or changed. No pending business decision was answered.

### Security — AI order confirmation (audit B1)
- The AI model **can no longer place orders**. `place_order` was removed from the model's tools. The tools are now read/quote-only: `search_products`, `get_customer_addresses`, `get_quote`, `get_order_status`.
- Added a **server-owned confirmation state machine** (`collecting → awaiting_confirmation → order_placed / fallback`), with `pending_quote_id` in `ai_sessions`.
- An order is placed only after a **Confirm tap** or a **separate server-side yes/no check** on the customer's voice reply. Unclear replies never place an order. Recorded as **DR-10** and **NFR-SEC-13**.
- Added `confirmation_method` (`tap` | `voice`) to orders, the `QUOTE_NOT_CONFIRMABLE` error, and confirmation and prompt-injection eval scenarios (0% false-confirmation gate).

### Business rules (audit C1–C3)
- **BR-18** reworded: the Super Admin *can* credit wallets (PDF wording). Other crediting actors moved to **PA-52**.
- **BR-22** reworded to the PDF wording ("API keys must not be exposed"). Secret keys never ship. Restricted public keys moved to **PA-53**.
- **BR-26** added (CONFIRMED, PDF §7): OTP authentication for all roles. **PA-15** narrowed to "extra DP login checks".
- Added **DR-08** (immediate suspension), **DR-09** (AI cannot set amounts), **DR-10** (AI confirmation), **DR-11** (catalog, price and stock changes through server functions, audit-logged).
- PA-14 now also covers DP-editable profile fields. PA-24 now also covers the contact method. PA-26 now also covers "feedback". PA-44 now also covers two-person approval.

### Pending decisions added
- **PA-49** stock reservation timing · **PA-50** open/close precedence · **PA-51** platform settings list · **PA-52** wallet crediting actors · **PA-53** public client keys · **PA-54** store vs warehouse · **PA-55** KPI and performance definitions · **PA-56** delivery request contents · **PA-57** AI cost currency · **PA-58** manager wallet visibility · **PA-59** supplier scope.
- All previously unnumbered "PENDING" items now cite a PA number.

### Database (audit B3, B9, B10)
- Wallets are now **customers only** (the ER diagram previously gave every profile a wallet).
- `delivery_partners` has an explicit `id` primary key.
- Added to the ER diagram: `order_quotes`, `stock_movements`, `platform_settings`, `ai_pricing_config`, and the links Stores→Coupons/Cashback/Complaints/Reviews/Products, Orders→Reviews/Complaints, AI sessions→quotes/orders.
- `reviews` gained `store_id`. `complaints` gained `type`. `stock_movements` has a full field list.
- `order_offers` renamed to **`delivery_requests`**.
- `ai_usage_logs` is one row per Gemini call, with `turn_id`, `purpose` and `cost_currency`.
- Access matrix updated: catalog writes via functions, protected profile fields, manager wallet visibility = PA-58.

### API (audit B2, B6–B8, A6, A8)
- Category, product and stock changes moved from direct table writes to Edge Functions (`manage-category`, `manage-product`, `adjust-stock`).
- Removed the assumption that `place-order` "reserves stock". Stock handling is now PA-49.
- `reply_audio_url` (stored audio) replaced with inline `reply_audio`. Storing audio requires PA-34.
- Added: `update-my-profile`, the notifications inbox (C14), DP management (S3), partner attendance (S14), the per-function security checklist, and `ACCOUNT_SUSPENDED` / `RATE_LIMITED` error codes.
- Dispatch endpoints renamed: `my-delivery-requests`, `accept-delivery-request`, `reject-delivery-request`.

### Consistency (audit B4, B5, B7)
- Status values are **lowercase snake_case** in every document.
- Terminology: "**delivery request**" for dispatch, "**offer**" for promotions only.
- AI usage logging is per Gemini call everywhere (SRS, schema, API, tests).
- The architecture diagram now shows dashboards using Storage.

### Security requirements added (audit E1–E5)
- NFR-SEC-08 account status checked on every request.
- NFR-SEC-09 protected profile fields.
- NFR-SEC-10 AI rate limits and prompt-injection protection.
- NFR-SEC-11 service-role key handling.
- NFR-SEC-12 serialized wallet writes.
- NFR-SEC-13 AI order confirmation.

### Requirements coverage (audit A1–A8)
- Added a KPI definitions table, marked PENDING (SRS §3.8, PA-55).
- Added SRS §3.7 tracing FR-WP-01 to FR-WP-07 (wallet, payments, offers), which were previously not traced in the SRS or API (found by the post-correction cross-check).
- Added acceptance criteria for manager performance (PA-55), platform products (PA-31/PA-51), feedback (PA-26), DP pre-accept view (PA-56), notification inbox, DP management and attendance views.

### Test coverage (found by the post-correction cross-check)
- The business-rule test matrix now names every BR-01…BR-26, DR-01…DR-11 and NFR-SEC-01…13 item.

### Roadmap (audit F1–F6)
- Push notification infrastructure moved to **Level 4**. Level 8 keeps only promotional notifications.
- Level 3 exit is now API-level role tests plus seed/setup scripts (no app UI needed).
- Level 4 exit is "order placed and visible as `placed`". The full lifecycle is the Level 6 exit.
- PA-10 now blocks Level 2. PA-05, PA-49 and PA-53 now block Level 4. Blocking lists were updated for every level.

## [0.1.0] — 2026-10-30 — Level 0: Project planning and documentation
- Initial documentation set created from the Ramu Kaka PRD v1.0 PDF: PRD, SRS, System Architecture, Database Schema (conceptual), API Documentation, UI/UX Design, User Flows, Business Rules (PA-01 to PA-48), Testing Strategy, Development Roadmap, CLAUDE.md, README.md.
