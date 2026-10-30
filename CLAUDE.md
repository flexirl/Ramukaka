# CLAUDE.md — Guidance for AI-assisted development on Ramu Kaka

This file tells Claude (and any AI coding assistant) how to work in this repository.

## Project in one paragraph
Ramu Kaka is a voice-first, AI-powered delivery platform for daily essentials. It starts with **water delivery** (bottles and jars) in **Bihar (Jamui first)**, building on Prashant's existing water-supply business. It has four panels: **Customer App**, **Delivery Partner App**, **Store Manager Dashboard**, and **Super Admin Dashboard**. All panels share one Supabase backend.

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
2. **Keep labels separate:** `CONFIRMED` (from PDF), `DERIVED` (logical consequence), `RECOMMENDATION` (proposal), `PENDING APPROVAL`.
3. **Work level by level.** Stop at the end of each level and wait for explicit user approval.
4. When a PA item is decided, update `BUSINESS_RULES.md` first (record date and approver). Then update every dependent document in the same change.
5. Reference requirement IDs (`FR-CU-xx`, `FR-DP-xx`, `FR-SM-xx`, `FR-SA-xx`, `FR-AI-xx`, `FR-WP-xx`, `BR-xx`, `PA-xx`) in commits, PRs and code comments where useful.

## Non-negotiable technical rules (from PRD §7 and §9)
- Confirmed stack: React Native + TypeScript (Android via Android Studio), Supabase Auth (OTP), Supabase PostgreSQL, Supabase server-side functions, Gemini API, GitHub, Google Stitch for design.
- **One shared backend and database** with role-based access control (RLS on every table).
- **Server-side only:** wallet credits, order assignment, pricing, financial calculations, AI calls.
- **No API keys or secrets in mobile apps or in git.** Only the Supabase anon key may ship in clients.
- A manager can access **only their assigned store**.
- An order can **never** be assigned to more than one delivery partner (atomic, first accept wins).
- Order placement **always** requires explicit customer confirmation (manual or voice).
- The AI must use **real backend prices, stock and delivery rules**. It must never make them up.
- The wallet is an **append-only, auditable ledger**.
- Closed stores and out-of-stock products must block order placement.
- AI usage and cost must be logged server-side for every call.

## Conventions (proposed; finalized in Level 2)
- Money is stored as integer **paise**.
- Language codes: `hi`, `en`, `bho`. No hard-coded user-facing strings.
- TypeScript strict mode everywhere.
- Proposed monorepo layout: see [docs/SYSTEM_ARCHITECTURE.md §5](docs/SYSTEM_ARCHITECTURE.md#5-proposed-repository-layout-recommendation-not-created-yet).

## Documentation map
| File | Purpose |
|---|---|
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

## Git
- Commit messages should be clear and descriptive, and reference requirement or PA IDs where relevant.
- Never commit secrets, `.env` files, or real customer data.
