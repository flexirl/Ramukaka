# Ramu Kaka

> *"Make ordering everyday essentials as easy as talking to someone."*

Ramu Kaka is a **voice-first, AI-powered delivery platform** for daily essentials. Customers order by talking to the Ramu Kaka AI assistant in **Hindi, English or Bhojpuri**, or by using a simple manual interface.

The first product is **water delivery** (bottles and jars), starting in **Jamui, Bihar** and built on an existing water-supply business. Milk, curd, paneer, groceries and other essentials are planned for later.

## Project Status

| Item | Status |
|---|---|
| Current level | **Level 0 – Project Planning & Documentation** |
| Application code | Not started (by design) |
| Database | Conceptual model only (no tables or migrations) |
| Mobile apps | Not initialized |
| Documentation | Audited and corrected (see [CHANGELOG](CHANGELOG.md)) |
| Next step | Stakeholder review of `docs/` and decisions on the Pending Approval items (PA-01 to PA-59) |

## Panels

| Panel | Users |
|---|---|
| Customer App (Android) | Customers: voice/manual ordering, tracking, wallet, offers, reviews |
| Delivery Partner App (Android) | Delivery staff: attendance, order acceptance, delivery, COD |
| Store Manager Dashboard | One manager per store: orders, inventory, offers, COD reconciliation |
| Super Admin Dashboard | Owner: all stores, finance, wallet credits, AI cost tracking |

## Confirmed Technology Stack
React Native + TypeScript (Android) · Supabase Auth (OTP) · Supabase PostgreSQL · Supabase server-side functions · Gemini API · Google Stitch (UI/UX) · GitHub · Android Studio

## Documentation

| Document | Description |
|---|---|
| [PRD](docs/PRD.md) | Product requirements, restated from the original PDF with IDs |
| [SRS](docs/SRS.md) | Software requirements with acceptance criteria |
| [System Architecture](docs/SYSTEM_ARCHITECTURE.md) | Components, ADRs, security |
| [Database Schema](docs/DATABASE_SCHEMA.md) | Proposed conceptual data model |
| [API Documentation](docs/API_DOCUMENTATION.md) | Proposed API contract |
| [UI/UX Design](docs/UI_UX_DESIGN.md) | Design principles and screen inventory |
| [User Flows](docs/USER_FLOWS.md) | End-to-end flows for every panel |
| [Business Rules](docs/BUSINESS_RULES.md) | Confirmed rules and the **Pending Approval register** |
| [Testing Strategy](docs/TESTING_STRATEGY.md) | Test levels, business-rule matrix, AI evaluation |
| [Development Roadmap](docs/DEVELOPMENT_ROADMAP.md) | Levels 0–11 and approval gates |

AI-assisted contributors should read [CLAUDE.md](CLAUDE.md) first. Changes are tracked in [CHANGELOG.md](CHANGELOG.md).

## Source
Original PRD: *Ramu Kaka PRD v1.0*, prepared by Anubhav Kumar (FlexIRL). Status: Initial Draft for Stakeholder Review and Approval.
