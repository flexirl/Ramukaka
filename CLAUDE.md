# CLAUDE.md — Guidance for AI-assisted development on Ramu Kaka

This file tells Claude (and any AI coding assistant) how to work in this repository.

## Project in one paragraph
Ramu Kaka is a voice-first, AI-powered delivery platform for daily essentials. It starts with **water delivery** (bottles and jars) in **Bihar (Jamui first)**, building on Prashant's existing water-supply business. It is **one React Native mobile app** with four role-based panels: **Customer**, **Delivery Partner**, **Store Manager** and **Super Admin**. There are no separate apps and no web dashboard (BR-27). All panels share one codebase, one login system and one Supabase backend. A plain-language summary is in [docs/PROJECT_OVERVIEW.md](docs/PROJECT_OVERVIEW.md).

## Current level
**Level 0 – Project Planning & Documentation.** See [docs/DEVELOPMENT_ROADMAP.md](docs/DEVELOPMENT_ROADMAP.md).

Until the user approves the next level:
- Do **NOT** write application code.
- Do **NOT** create database tables, SQL or migrations.
- Do **NOT** initialize React Native or any other app scaffold.

## Source of truth
1. The original **Ramu Kaka PRD v1.0 (PDF)** by FlexIRL, restated in [docs/PRD.md](docs/PRD.md).
2. [docs/BUSINESS_RULES.md](docs/BUSINESS_RULES.md): confirmed rules and the **Pending Approval register (PA-xx)**.
3. The other `docs/` files are derived documents. If they conflict with 1 or 2, then 1 and 2 win.

## Rules for working on this project
1. **Never invent business requirements.** If something is not in the PRD and not approved, mark it `PENDING APPROVAL` and add or reference a `PA-xx` item.
2. **Keep labels separate:** `CONFIRMED` (from PDF), `APPROVED DECISION` (decided by the project owner, with date and approver), `DERIVED` (logical consequence), `RECOMMENDATION` (proposal), `PENDING APPROVAL`.
3. **Work level by level.** Stop at the end of each level and wait for explicit user approval.
4. When a PA item is decided, update `BUSINESS_RULES.md` first (record date and approver). Then update every dependent document in the same change.
5. Reference requirement IDs (`FR-CU-xx`, `FR-DP-xx`, `FR-SM-xx`, `FR-SA-xx`, `FR-AI-xx`, `FR-WP-xx`, `BR-xx`, `DR-xx`, `NFR-xx`, `PA-xx`) in commits, PRs and code comments where useful.
6. Record every documentation or rule change in [CHANGELOG.md](CHANGELOG.md).
7. `APPROVED DESIGN RULE` (e.g. DR-10, DR-11) is only for technical rules the user approved. It never answers a business decision that is still `PENDING APPROVAL`.

## Non-negotiable technical rules (from PRD §7 and §9)
- Confirmed stack: React Native + TypeScript (Android via Android Studio), Supabase Auth (OTP for **all** roles), Supabase PostgreSQL, Supabase server-side functions, Gemini API, GitHub, Google Stitch for design.
- **One React Native app** with four panels (BR-27). Never create a separate app or a web dashboard.
- After login, the app routes the user to a panel based on the **backend-assigned role** (BR-28). Users can never change their own role; self-registration creates a customer only (DR-12). Each panel has its own navigation and screens, and panels never import each other's screens.
- **The app is never trusted for access control** (NFR-SEC-14). All panels ship in one app, so every backend call must check role, status and store scope.
- Request device permissions only when the user's role and the feature need them. Never request background location (DR-13).
- **One shared backend and database** with role-based access control (RLS on every table).
- **Server-side only:** wallet credits, order assignment, pricing, financial calculations, AI calls, and every catalog, price and stock change (DR-11).
- **API keys must not be exposed in the mobile app (PDF §7).** Secret keys (Gemini, service-role, SMS) never go in any client, dashboard or git. Only the Supabase anon key ships in clients until **PA-53** decides on restricted public keys (Maps, FCM).
- The Supabase service-role key exists only in Edge Function secrets. Every function authorizes the caller (JWT, role, **active status**, store scope) before any privileged query.
- Account status is checked on every request, so suspension is immediate.
- A manager can access **only their assigned store**.
- An order can **never** be assigned to more than one delivery partner (atomic, first accept wins).
- Order placement **always** requires explicit customer confirmation (manual or voice).
- **The AI model can never place an order or set any amount** (DR-09, DR-10). It has read/quote-only tools. The server places the order only after a Confirm tap or a server-side yes/no check of the customer's voice reply, for the session's pending quote.
- The AI must use **real backend prices, stock and delivery rules**. It must never make them up.
- The wallet is an **append-only, auditable ledger**, with writes serialized per wallet.
- Closed stores and out-of-stock products must block order placement.
- AI usage and cost must be logged server-side for every Gemini API call.

## Conventions (proposed; finalized in Level 2)
- Money is stored as integer **paise**.
- Language codes: `hi`, `en`, `bho`. No hard-coded user-facing strings.
- Status values are lowercase snake_case (`placed`, `out_for_delivery`, ...).
- Terminology: **delivery request** = dispatch of an order to delivery partners; **offer** = promotional offer only.
- TypeScript strict mode everywhere.
- Proposed repository layout (one app, panel folders): see [docs/SYSTEM_ARCHITECTURE.md §6](docs/SYSTEM_ARCHITECTURE.md#6-proposed-repository-layout-recommendation-not-created-yet).

## Documentation map
| File | Purpose |
|---|---|
| docs/PROJECT_OVERVIEW.md | Plain-language explanation of the whole project |
| docs/PRD.md | Product requirements (restated from PDF) |
| docs/SRS.md | Testable functional and non-functional requirements |
| docs/SYSTEM_ARCHITECTURE.md | Architecture and ADRs |
| docs/DATABASE_SCHEMA.md | Conceptual data model (no DDL yet) |
| docs/API_DOCUMENTATION.md | Proposed API contract |
| docs/UI_UX_DESIGN.md | Design principles and screen inventory |
| docs/USER_FLOWS.md | End-to-end flows |
| docs/BUSINESS_RULES.md | Rules + Pending Approval register |
| docs/TESTING_STRATEGY.md | Test approach and business-rule test matrix |
| docs/DEVELOPMENT_ROADMAP.md | Levels, gates and risks |
| CHANGELOG.md | History of documentation and rule changes |

## Git
- Commit messages should be clear and descriptive, and reference requirement or PA IDs where relevant.
- Never commit secrets, `.env` files, or real customer data.
